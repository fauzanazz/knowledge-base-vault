# Tensor Parallelism for LLM Serving

Tensor Parallelism (TP) is the primary strategy for sharding large language model weights across multiple GPUs within a single node. Pioneered by the **Megatron-LM** project (Shoeybi et al., 2019) and adapted for inference in vLLM, TP enables each GPU to hold only a *fraction* of any given weight matrix while contributing a *fraction* of the total computation. Because modern matrix multiplications decompose naturally into independent partial products that are later aggregated, TP delivers near-linear latency scaling with GPU count under fast interconnects such as NVLink.

---

## Table of Contents

1. [Conceptual Foundation](#conceptual-foundation)
2. [Column and Row Parallelism](#column-and-row-parallelism)
3. [MLP Layer Splitting (Llama-Style)](#mlp-layer-splitting-llama-style)
4. [Attention Head Splitting](#attention-head-splitting)
5. [All-Reduce Communication](#all-reduce-communication)
6. [NCCL and the Communication Stack](#nccl-and-the-communication-stack)
7. [GPU Interconnects: NVLink vs PCIe vs InfiniBand](#gpu-interconnects-nvlink-vs-pcie-vs-infiniband)
8. [Scaling Characteristics and Super-Linear Effects](#scaling-characteristics-and-super-linear-effects)
9. [TP Constraints and Divisibility Requirements](#tp-constraints-and-divisibility-requirements)
10. [V1 Architecture Improvements for TP](#v1-architecture-improvements-for-tp)
11. [Configuration Reference](#configuration-reference)
12. [Failure Modes and Troubleshooting](#failure-modes-and-troubleshooting)
13. [Decision Guide](#decision-guide)

---

## Conceptual Foundation

At its core, tensor parallelism exploits the distributive property of matrix multiplication:

```
Y = X @ W   can be decomposed as:

Column split:  W = [W₀ | W₁ | ... | W_{N-1}]  (split along output dim)
               Each GPU i computes: Y_i = X @ W_i   (independent, no comm)

Row split:     W = [W₀ ; W₁ ; ... ; W_{N-1}]  (split along input dim)
               Each GPU i computes: P_i = X_i @ W_i  (partial sum)
               Final: Y = P₀ + P₁ + ... + P_{N-1}   (all-reduce needed)
```

The key insight is that a **Column Parallel** layer produces sharded outputs that can be fed directly as inputs to a **Row Parallel** layer — with only a single all-reduce at the boundary. This column→row pairing is the fundamental building block of Megatron-LM-style TP.

---

## Column and Row Parallelism

### Column Parallelism

```
Weight W [in_features × out_features]
Split along output (column) dimension into N shards:

GPU 0: W₀ [in × out/N]    GPU 1: W₁ [in × out/N]    ...    GPU N-1: W_{N-1} [in × out/N]

Input X [batch × in_features]  ←  REPLICATED on every GPU (broadcast once)

Each GPU i computes:   Y_i = X @ W_i   →   Shape: [batch × out/N]

Result: Y is SHARDED across GPUs (no communication needed)
```

**Properties:**
- No inter-GPU communication during the forward pass
- Input must be fully replicated on each GPU (requires broadcast at layer start, or the previous layer was row-parallel which already produced replicated output via all-reduce)
- Output is partitioned — not usable by downstream ops that expect full tensors without further coordination

### Row Parallelism

```
Weight W [in_features × out_features]
Split along input (row) dimension into N shards:

GPU 0: W₀ [in/N × out]    GPU 1: W₁ [in/N × out]    ...    GPU N-1: W_{N-1} [in/N × out]

Input X [batch × in_features]  ←  SHARDED  (X_i = X[:, i*(in/N) : (i+1)*(in/N)])

Each GPU i computes:   P_i = X_i @ W_i   →   Shape: [batch × out]  (partial sum)

All-reduce:  Y = P₀ + P₁ + ... + P_{N-1}   →   REPLICATED on every GPU
```

**Properties:**
- Requires an all-reduce collective to sum partial results
- Input must be sharded (aligns perfectly with column-parallel output)
- Output is replicated — ready for the next operation without further communication

### Column → Row: The Paired Pattern

The canonical Megatron-LM pattern pairs a column-parallel layer with a row-parallel layer to achieve a full compute pass with only **one all-reduce**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Forward Pass (TP=4)                           │
│                                                                  │
│  Full Input X                                                    │
│       │                                                          │
│       ├─── replicate to all GPUs ──────────────────────────┐   │
│       │                                                      │   │
│  GPU0 │  GPU1 │  GPU2 │  GPU3                                │   │
│  W₀col│  W₁col│  W₂col│  W₃col   ← Column Parallel         │   │
│       ↓       ↓       ↓       ↓                              │   │
│  Y₀   │  Y₁   │  Y₂   │  Y₃      (sharded activations)     │   │
│       ↓       ↓       ↓       ↓                              │   │
│  [activation fn — element-wise, no comm]                     │   │
│       ↓       ↓       ↓       ↓                              │   │
│  W₀row│  W₁row│  W₂row│  W₃row   ← Row Parallel            │   │
│       ↓       ↓       ↓       ↓                              │   │
│  P₀   │  P₁   │  P₂   │  P₃      (partial sums)            │   │
│       └───────┴───────┴───────┘                              │   │
│                    ALL-REDUCE  ← single comm per MLP/Attn    │   │
│                       ↓                                      │   │
│                  Full Output Y (replicated on all GPUs)      │   │
└─────────────────────────────────────────────────────────────────┘
```

---

## MLP Layer Splitting (Llama-Style)

A Llama-style MLP consists of three projections: `gate_proj`, `up_proj`, and `down_proj`, with a SiLU-gated activation:

```
output = down_proj( silu(gate_proj(x)) * up_proj(x) )
```

Under TP, these are split as follows:

```
                     ┌──────────────────────────────────────────────┐
                     │             Llama MLP with TP=4              │
                     └──────────────────────────────────────────────┘

Input x [batch × hidden]   ← REPLICATED on all GPUs

                    GPU 0          GPU 1          GPU 2          GPU 3
                 ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
gate_proj (col)  │ W_g0    │    │ W_g1    │    │ W_g2    │    │ W_g3    │
up_proj   (col)  │ W_u0    │    │ W_u1    │    │ W_u2    │    │ W_u3    │
                 └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
                      │              │              │              │
                   silu(g0)*u0    silu(g1)*u1    silu(g2)*u2    silu(g3)*u3
                      │              │              │              │
down_proj (row)  ┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
                 │ W_d0    │    │ W_d1    │    │ W_d2    │    │ W_d3    │
                 └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
                      │              │              │              │
                      └──────────────┴──────────────┴──────────────┘
                                         ALL-REDUCE
                                             │
                                       Output [batch × hidden]
                                       (replicated, ready for next layer)
```

**Steps:**
1. `gate_proj` and `up_proj` are **column-parallel**: each GPU computes `[i*(intermediate/N) : (i+1)*(intermediate/N)]` output neurons
2. SiLU activation is applied **locally** (element-wise on sharded output, no communication)
3. Element-wise multiply of `silu(gate) * up` happens locally on each GPU's shard
4. `down_proj` is **row-parallel**: each GPU receives its aligned shard of the intermediate activations
5. **Single all-reduce** aggregates partial sums → full output available on all GPUs

---

## Attention Head Splitting

For multi-head attention (MHA), TP splits along the **attention head** dimension:

```
Total attention heads: H = 32 (e.g., Llama-3-8B)
TP size: 4
Heads per GPU: H/TP = 8

                     ┌──────────────────────────────────────────────┐
                     │       Attention with TP=4 (32 heads)         │
                     └──────────────────────────────────────────────┘

Input x [batch × seq × hidden]   ← REPLICATED

             GPU 0        GPU 1        GPU 2        GPU 3
Q_proj (col) [H 0-7]     [H 8-15]    [H 16-23]   [H 24-31]
K_proj (col) [H 0-7]     [H 8-15]    [H 16-23]   [H 24-31]
V_proj (col) [H 0-7]     [H 8-15]    [H 16-23]   [H 24-31]

Attention:   Local 8-head    Local 8-head    Local 8-head    Local 8-head
             self-attn       self-attn       self-attn       self-attn
             (no comm)       (no comm)       (no comm)       (no comm)

O_proj (row) partial sum   partial sum    partial sum    partial sum
                 └──────────────┴──────────────┴──────────────┘
                                     ALL-REDUCE
                                         │
                              Full context output (replicated)
```

**For GQA (Grouped Query Attention)** (e.g., Llama 3.1 8B uses 32 Q heads, 8 KV heads):
- Q heads: split as `32 / TP` per GPU
- KV heads: split as `8 / TP` per GPU (must satisfy `num_kv_heads % TP == 0`)
- With TP=4: 8 Q heads + 2 KV heads per GPU

**For MLA (Multi-head Latent Attention, DeepSeek):**
- TP is not efficient for MLA due to latent projection duplication — Expert Parallelism (EP) with DP is preferred for MoE models using MLA

---

## All-Reduce Communication

The all-reduce is the single collective communication operation that makes TP functional. It sums partial tensors across all N GPUs and distributes the result back to all participants.

### Ring All-Reduce Algorithm

```
Ring topology (4 GPUs):

    GPU 0 ──► GPU 1
      ▲          │
      │          ▼
    GPU 3 ◄── GPU 2

Reduce-scatter phase (N-1 steps):
  Each GPU sends 1/N of its partial sum to the next GPU
  After N-1 steps: each GPU holds the fully reduced value for 1/N of the tensor

All-gather phase (N-1 steps):
  Each GPU sends its reduced shard to the next GPU
  After N-1 steps: all GPUs hold the complete reduced tensor

Total steps: 2(N-1)
Bandwidth cost: 2 × (N-1)/N × message_size  [approaches 2× for large N]
```

### Message Size in vLLM

For an LLM with hidden dimension `H`, serving `T` tokens with data type `D` bytes:

```
All-reduce message size = T × H × D bytes

Example (Llama-3-70B, BF16, during decode):
  T = 256 tokens in batch
  H = 8192 hidden dim
  D = 2 bytes (BF16)
  Size = 256 × 8192 × 2 = 4 MB per all-reduce

During prefill (long prompt of 4096 tokens):
  Size = 4096 × 8192 × 2 = 64 MB per all-reduce
```

Decode all-reduces are small and latency-bound; prefill all-reduces are large and bandwidth-bound.

---

## NCCL and the Communication Stack

**NCCL** (NVIDIA Collective Communications Library) is vLLM's primary inter-GPU communication library. It implements all-reduce, all-gather, reduce-scatter, point-to-point send/recv, and all-to-all collectives.

### NCCL in vLLM's Stack

```
vLLM Python Code
    └── PyTorch Distributed  (torch.distributed.all_reduce)
           └── NCCL  (libncll.so)
                  ├── NVLink Transport   (intra-node, GPU↔GPU)
                  ├── PCIe Transport     (intra-node, no NVLink)
                  ├── InfiniBand (IB)    (inter-node, RDMA)
                  └── RoCE (RDMA/Ethernet) (inter-node)
```

### NCCL Initialization Process

```
1. vLLM calls torch.distributed.init_process_group(backend="nccl")
   └── Each worker process registers its rank and world size

2. NCCL performs ring topology discovery
   └── Detects NVLink connections, PCIe topology, IB HCAs

3. NCCL establishes channels
   └── Number of channels configurable (default: auto-detected)
   └── Each channel = independent ring for pipelining transfers

4. Warm-up all-reduce validates connectivity
   └── Log: "NCCL INFO Connected all rings, use ring PXN 0 GDR 1"

5. Workers ready for model execution
```

### Key NCCL Environment Variables

```bash
# Per-node IP for NCCL rendezvous
VLLM_HOST_IP=<private_ip_of_this_node>

# Specify InfiniBand HCA device
NCCL_IB_HCA=mlx5

# Debug logging (useful during setup)
NCCL_DEBUG=INFO        # Basic connectivity info
NCCL_DEBUG=TRACE       # Full protocol trace (verbose)

# Prevent Gloo backend from selecting wrong interface on IB clusters
GLOO_SOCKET_IFNAME=eth0

# GPUDirect RDMA (GPU-to-NIC direct transfer, bypass CPU)
# Enable by ensuring gdrcopy is installed and:
NCCL_NET_GDR_LEVEL=5   # Force GDR

# Tune for InfiniBand
NCCL_IB_TIMEOUT=23     # Increase for slow networks
NCCL_IB_RETRY_CNT=7
```

---

## GPU Interconnects: NVLink vs PCIe vs InfiniBand

The performance of tensor parallelism is fundamentally constrained by the interconnect bandwidth between GPUs. Different hardware configurations have dramatically different characteristics:

### Interconnect Comparison Table

| Interconnect | Topology | Bandwidth | Latency | TP Suitability |
|---|---|---|---|---|
| **NVLink 4.0** (H100) | Intra-node, all-to-all | 900 GB/s total (18 links × 50 GB/s) | ~1 µs | ✅ Excellent — TP=8 standard |
| **NVLink 3.0** (A100) | Intra-node | 600 GB/s total | ~1 µs | ✅ Very good — TP=8 viable |
| **NVLink-C2C** (GB200) | CPU-GPU on die | 900+ GB/s | ~0.5 µs | ✅ Exceptional |
| **NVSwitch** (DGX) | Full-bisection intra-node | Scales with topology | ~1 µs | ✅ Ideal for large TP |
| **PCIe 4.0** | Intra-node (L40S, A10G) | 32–64 GB/s | 5–10 µs | ⚠️ Marginal — TP=2-4 only |
| **InfiniBand HDR** | Inter-node | 200 Gb/s (~25 GB/s) | ~1–2 µs | ⚠️ Use for PP, not TP |
| **InfiniBand NDR** | Inter-node | 400 Gb/s (~50 GB/s) | ~1–2 µs | ⚠️ Borderline for TP |
| **RoCE 100GbE** | Inter-node | 100 Gb/s (~12.5 GB/s) | ~2–5 µs | ❌ Too slow for TP |

### Intra-Node: NVLink Architecture

```
DGX H100 (8× H100, NVSwitch topology):

  GPU 0 ──NVLink──┐
  GPU 1 ──NVLink──┤
  GPU 2 ──NVLink──┤   NVSwitch    ← full bisection bandwidth
  GPU 3 ──NVLink──┤   Fabric      ← every GPU can send to every GPU
  GPU 4 ──NVLink──┤   ~900 GB/s   simultaneously at full bandwidth
  GPU 5 ──NVLink──┤
  GPU 6 ──NVLink──┤
  GPU 7 ──NVLink──┘

All-reduce on 4 MB tensor (TP=8): ~4 MB / (900 GB/s / 8) ≈ ~35 µs
                                                                ↑ negligible vs. 5ms compute
```

### Intra-Node: PCIe Topology (No NVLink)

```
Server with L40S GPUs (PCIe 4.0):

  GPU 0 ──PCIe──┐
  GPU 1 ──PCIe──┼── CPU Socket 0 ──PCIe──── GPU 4
  GPU 2 ──PCIe──┤   (NUMA node)              GPU 5
  GPU 3 ──PCIe──┘                            GPU 6
                                             GPU 7

Cross-socket PCIe: goes through QPI/UPI (much slower)
All-reduce on 4 MB tensor (TP=4): ~4 MB / (16 GB/s) ≈ ~250 µs
                                                          ↑ significant overhead
```

### Inter-Node: InfiniBand for Pipeline Parallelism

```
Node 0                        Node 1
 8× H100 (NVLink)              8× H100 (NVLink)
   TP=8 group                    TP=8 group
       │                              │
  ConnectX-7 HCA ←── IB HDR ──► ConnectX-7 HCA
  (200 Gb/s)                    (200 Gb/s)

Pipeline stage boundary communication:
  Activation tensor [256 tokens × 8192 dim × 2 bytes] = 4 MB
  Transfer time: 4 MB / 25 GB/s ≈ 160 µs
  (Acceptable for PP stage crossings; too slow for per-layer TP)
```

### GPUDirect RDMA

With GPUDirect RDMA enabled, NCCL can transfer data directly from GPU HBM to the NIC without involving the CPU:

```
Without GPUDirect:     GPU HBM → CPU cache → System RAM → NIC → network
With GPUDirect RDMA:   GPU HBM ──────────────────────── NIC → network

Latency reduction: eliminates 2 memory copies (~microseconds per transfer)
Bandwidth: achieves full NIC wire speed (vs. PCIe-limited without GDR)
```

Enable in Docker: `--ipc=host --shm-size=16G`  
Enable in Kubernetes:
```yaml
securityContext:
  capabilities:
    add: ["IPC_LOCK"]
volumes:
- name: dshm
  emptyDir:
    medium: Memory
```

---

## Scaling Characteristics and Super-Linear Effects

### Linear vs Super-Linear Scaling

Naive expectation: doubling GPUs (TP=1→TP=2) should double throughput.

Reality from vLLM benchmarks: **super-linear throughput gains** are common because TP affects not just compute but also KV cache capacity.

```
TP=1 → TP=2 (Llama-3.1-70B example):

  Compute: 2× (linear — each GPU does half the work)
  
  GPU memory freed by weight reduction:
    Weights: 140 GB total → 70 GB per GPU (freed 70 GB)
    KV cache blocks increase: 13.9× more blocks available
    
  Effect on throughput:
    Larger KV cache → larger batch sizes
    Larger batches → better GPU utilization  
    Throughput increase: 3.9× (vs 2× naive expectation)
```

### Why KV Cache Size Matters

```
GPU VRAM = Model Weights + KV Cache + Activation Buffers + CUDA Overhead

  TP=1, 80 GB A100:
    Weights: ~140 GB  → doesn't fit (need model parallelism!)
    
  TP=2, 2× 80 GB A100:
    Weights: 70 GB per GPU
    Available for KV: 80 - 70 - overhead ≈ 5 GB per GPU → 10 GB total
    
  TP=8, 8× 80 GB A100:
    Weights: 17.5 GB per GPU
    Available for KV: 80 - 17.5 - overhead ≈ 55 GB per GPU → 440 GB total
    KV cache is 44× larger than TP=2 case!
```

The super-linear effect is most pronounced when models are weight-memory-bound (large models on GPUs with limited VRAM).

---

## TP Constraints and Divisibility Requirements

Tensor parallelism imposes strict divisibility requirements on model dimensions:

```
For standard attention:
  num_attention_heads % tensor_parallel_size == 0

For GQA/MQA:
  num_key_value_heads % tensor_parallel_size == 0

For MLP:
  intermediate_size % tensor_parallel_size == 0

For embeddings:
  vocab_size % tensor_parallel_size == 0  (preferred, with padding if needed)

For MoE layers (if using TP over MoE):
  num_experts % tensor_parallel_size == 0
```

### Common Model Constraints

| Model | Heads | Max TP (heads) | Notes |
|---|---|---|---|
| Llama-3.1-8B | 32 Q, 8 KV | TP=8 (8 KV heads) | 1 KV head per GPU |
| Llama-3.1-70B | 64 Q, 8 KV | TP=8 | 1 KV head per GPU |
| Llama-3.1-405B | 128 Q, 8 KV | TP=8 max for GQA | Use PP for multi-node |
| Mistral-7B | 32 Q, 8 KV | TP=8 | Standard |
| DeepSeek-V3 | MLA (latent) | TP=1 preferred | Use EP instead |

---

## V1 Architecture Improvements for TP

vLLM V1 (GA in v0.11.0, December 2025) introduced key improvements to the TP worker architecture:

### V0 Architecture (Asymmetric)

```
V0 TP Worker Topology:

Process 0 (rank 0):
  ├── Scheduler
  ├── BlockManager  
  ├── Worker 0 (GPU 0)   ← colocated with scheduler
  └── ZeroMQ IPC to ranks 1-N

Process 1 (rank 1):
  └── Worker 1 (GPU 1)

...asymmetric: rank 0 is special
```

### V1 Architecture (Symmetric)

```
V1 TP Worker Topology:

API Server Process:
  └── FastAPI + tokenization + detokenization

EngineCore Process:
  ├── Scheduler (SEPARATE from workers)
  └── ZeroMQ IPC to all workers

Worker Process 0 (rank 0, GPU 0):   ←  symmetric
  └── Caches request state locally
  
Worker Process 1 (rank 1, GPU 1):   ←  symmetric
  └── Caches request state locally

...
Worker Process N-1 (rank N-1, GPU N-1):
  └── Caches request state locally

Communication: scheduler sends only INCREMENTAL DIFFS
               workers apply diffs to cached state
```

**Benefits:**
- All TP workers are architecturally identical — simplifies fault recovery
- Incremental state diffs dramatically reduce IPC overhead
- Enables clean separation of scheduling from execution

---

## Configuration Reference

### Basic TP Setup

```bash
# Single-node, 4-GPU TP
vllm serve meta-llama/Llama-3.1-70B-Instruct \
    --tensor-parallel-size 4

# Single-node, 8-GPU TP (full node)
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8

# Python API
from vllm import LLM
llm = LLM(
    "meta-llama/Llama-3.1-70B-Instruct",
    tensor_parallel_size=4
)
```

### Combined TP + Pipeline Parallelism

```bash
# 2 nodes × 8 GPUs each (TP within node, PP across nodes)
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray
```

### Environment Variables for NCCL Tuning

```bash
# Basic multi-node setup
export VLLM_HOST_IP=10.0.0.1    # unique per node
export NCCL_IB_HCA=mlx5         # IB device name
export GLOO_SOCKET_IFNAME=eth0  # prevent Gloo hang on IB clusters

# Performance tuning
export NCCL_SOCKET_NTHREADS=4   # parallel socket threads
export NCCL_NSOCKS_PERTHREAD=4  # sockets per thread

# Debug
export NCCL_DEBUG=INFO          # log connectivity info
```

---

## Failure Modes and Troubleshooting

| Symptom | Root Cause | Resolution |
|---|---|---|
| NCCL timeout during `all_reduce` | Network congestion, IB HCA misconfigured | Set `NCCL_IB_HCA=mlx5`, check `ibstat`, increase `NCCL_IB_TIMEOUT` |
| Hang on `init_process_group` | Missing `VLLM_HOST_IP` or firewall blocking port | Set unique `VLLM_HOST_IP` per node; open ports 6379 (Ray) + 29500 (distributed) |
| Low TP throughput on PCIe system | PCIe bandwidth saturation | Reduce TP size, use PP instead, ensure NUMA affinity |
| `AssertionError: num_heads % tp_size` | Model head count not divisible by TP | Reduce TP size to a valid divisor |
| NCCL out-of-memory | All-reduce buffers too large | Reduce batch size or use `--max-num-seqs` |
| Gloo backend hangs on IB cluster | Wrong socket interface selected | Set `GLOO_SOCKET_IFNAME=eth0` |

### TP vs PP Decision Rule

```
NVLink available between all GPUs?
  YES, model fits on node: → Use TP = GPU count on node
  YES, model too large for node: → Use TP = GPU count, PP = node count
  
No NVLink (PCIe only)?
  → Prefer PP over large TP
  → If TP needed: limit to TP=2 on same PCIe root complex
```

---

## Decision Guide

```
┌─────────────────────────────────────────────────────────┐
│              Tensor Parallelism Decision Guide           │
└─────────────────────────────────────────────────────────┘

Does the model fit on a single GPU?
    YES → No TP needed (single GPU serving)
    NO  ↓

Does the node have NVLink (H100, A100, GB200)?
    YES → Use TP = (GPUs per node), e.g., TP=8 for DGX
    NO  → Use TP=2 max (same PCIe root complex) or use PP

Does TP alone cover the model?
    YES → Done: vllm serve --tensor-parallel-size N
    NO  → Combine with PP: --tensor-parallel-size N --pipeline-parallel-size M

Is the model an MoE (DeepSeek, Mixtral)?
    YES → Consider Expert Parallelism (EP) + Data Parallelism instead
          See: distributed-llm-serving.md for EP guidance
    NO  → Standard TP applies
```

---

## See Also

- [Pipeline Parallelism](pipeline-parallelism.md) — Layer-based parallelism for cross-node deployments
- [Distributed LLM Serving](distributed-llm-serving.md) — Multi-node Ray setup, MoE expert parallelism, PD disaggregation
- [vLLM V1 Architecture](vllm-v1-architecture.md) — V1 scheduler and worker architecture improvements

---

*Based on vLLM V1 architecture (v0.11.x+, 2025–2026). Sources: Megatron-LM paper (Shoeybi et al. 2019), vLLM official documentation, vLLM blog posts, GitHub RFCs.*
