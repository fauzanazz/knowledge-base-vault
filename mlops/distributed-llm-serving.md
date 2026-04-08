# Distributed LLM Serving

Multi-node LLM serving is the operational frontier of large-scale inference — running models that are too large for a single node, or running at throughput levels that demand hundreds of GPUs working in concert. This article covers the full stack: Ray integration for cluster orchestration, NCCL configuration for GPU communication, disaggregated prefill/decode (PD disaggregation) for workload separation, expert parallelism (EP) for MoE models, and data parallelism patterns and scaling strategies.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Ray Integration for Multi-Node Deployment](#ray-integration-for-multi-node-deployment)
3. [NCCL Setup and Configuration](#nccl-setup-and-configuration)
4. [Disaggregated Prefill/Decode (PD Disaggregation)](#disaggregated-prefill-decode-pd-disaggregation)
5. [Expert Parallelism for MoE Models](#expert-parallelism-for-moe-models)
6. [Data Parallelism Patterns](#data-parallelism-patterns)
7. [Wide Expert Parallelism (WideEP)](#wide-expert-parallelism-wideep)
8. [Scaling Strategies and Decision Guide](#scaling-strategies-and-decision-guide)
9. [Fault Tolerance and Recovery](#fault-tolerance-and-recovery)
10. [Hardware-Specific Guidance](#hardware-specific-guidance)
11. [Configuration Reference](#configuration-reference)

---

## Architecture Overview

A distributed vLLM deployment consists of several layers working together:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Distributed vLLM Stack                           │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                   Client / Load Balancer                      │  │
│  └───────────────────────────┬──────────────────────────────────┘  │
│                              │  HTTP/WebSocket                       │
│  ┌───────────────────────────▼──────────────────────────────────┐  │
│  │              API Server Process (FastAPI)                     │  │
│  │   OpenAI-compatible endpoint, tokenization, streaming        │  │
│  └───────────────────────────┬──────────────────────────────────┘  │
│                              │  ZeroMQ IPC                          │
│  ┌───────────────────────────▼──────────────────────────────────┐  │
│  │                  EngineCore Process                           │  │
│  │   Scheduler · KV cache manager · Persistent batch            │  │
│  └──────┬──────────────────────────────────────────────┬────────┘  │
│         │  Worker IPC                                   │           │
│  ┌──────▼──────┐ ┌──────────────┐ ┌──────────────┐    │           │
│  │ Worker      │ │ Worker       │ │ Worker       │   ...           │
│  │ (GPU 0)     │ │ (GPU 1)      │ │ (GPU N-1)    │                 │
│  │ TP rank 0   │ │ TP rank 1    │ │ TP rank N-1  │                 │
│  └─────────────┘ └──────────────┘ └──────────────┘                 │
│         │ NCCL all-reduce (TP) / P2P (PP) / all-to-all (EP)        │
│  ┌──────┴──────────────────────────────────────────────────────┐   │
│  │           Ray Cluster (multi-node orchestration)            │   │
│  │   Placement groups · Resource scheduling · Health checks    │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Ray Integration for Multi-Node Deployment

### Why Ray?

Ray serves as vLLM's **distributed compute fabric** for multi-node deployments. It provides:

- **Worker placement** across nodes via placement groups (SPREAD strategy)
- **Resource discovery** — automatic enumeration of GPUs and memory per node
- **Task routing** — dispatching forward pass shards to correct GPUs across nodes
- **Cluster lifecycle** — head/worker node coordination, health monitoring
- **Fault tolerance** (Ray 2.55+) — gang scheduling for DP groups

### Ray Cluster Architecture

```
Ray Head Node (typically co-located with API server)
├── Raylet (resource manager)
├── Object Store (plasma)
├── vLLM API Server (FastAPI + tokenization)
├── AsyncLLMEngine / EngineCore
├── Scheduler
└── RayDistributedExecutor
     ├── Placement Group (SPREAD strategy across nodes)
     │    ├── Bundle 0: Node 0, 8 GPUs (TP group)
     │    ├── Bundle 1: Node 1, 8 GPUs (TP group)
     │    └── ...
     ├── RayWorkerWrapper (Node 0, GPU 0-7) [PP stage 0, TP=8]
     ├── RayWorkerWrapper (Node 1, GPU 0-7) [PP stage 1, TP=8]
     └── ... (additional PP stages)

Ray Worker Nodes
├── Raylet (resource manager)
├── Object Store
└── RayWorkerWrapper processes (GPU workers)
```

### Classic Ray Setup

**Step 1: Start Ray cluster**

```bash
# Head node
ray start --head \
    --port=6379 \
    --node-ip-address=<head_node_ip> \
    --num-gpus=8

# Worker nodes (run on each worker)
ray start \
    --address=<head_node_ip>:6379 \
    --node-ip-address=<this_node_ip> \
    --num-gpus=8

# Verify cluster
ray status
```

**Step 2: Launch vLLM**

```bash
# On head node
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray \
    --port 8000
```

### Simplified Deployment: `ray symmetric-run` (2025)

The `ray symmetric-run` command launches the same command on every node simultaneously — analogous to `mpirun` or `torchrun` in HPC environments:

```bash
# In SLURM sbatch script or via mpssh/pdsh
ray symmetric-run \
    --address <head_node_ip>:6379 \
    --min-nodes 2 \
    --num-gpus 8 \
    -- vllm serve Qwen/Qwen3-32B \
       --tensor-parallel-size 8 \
       --pipeline-parallel-size 2

# What happens under the hood:
# 1. First node starts Ray in head mode, waits for min-nodes
# 2. Other nodes join as workers
# 3. Head node executes the vLLM serve command
# 4. Environment variables automatically propagated to all nodes
```

This eliminates the need to manually script Ray start/stop across nodes — especially useful in SLURM or Kubernetes environments.

### Ray Executor vs Multiprocessing Executor

| Backend | Use Case | Notes |
|---|---|---|
| `mp` (multiprocessing) | Single-node TP/PP | Workers fork from parent; simpler, no placement group issues |
| `ray` | Multi-node TP/PP | Required for cross-node; handles resource discovery |

**Important caveat with Ray Serve:**  
When using Ray Serve for autoscaling, use `distributed_executor_backend="mp"` for single-node TP. The Ray executor's placement group model conflicts with Ray Serve's actor resource allocation in V1's subprocess-based EngineCore. Multi-node TP with Ray Serve requires the Ray executor and careful architectural planning.

```python
# Single-node TP within Ray Serve deployment: use mp backend
from ray import serve
from vllm import AsyncLLMEngine, AsyncEngineArgs

@serve.deployment
class VLLMDeployment:
    def __init__(self):
        args = AsyncEngineArgs(
            model="meta-llama/Llama-3.1-70B-Instruct",
            tensor_parallel_size=8,
            distributed_executor_backend="mp",  # ← critical for single-node
        )
        self.engine = AsyncLLMEngine.from_engine_args(args)
```

### Ray Placement Group Configuration

For multi-node deployments, Ray MUST use SPREAD strategy to distribute workers across nodes:

```bash
# Force SPREAD placement strategy (prevents all workers on one node)
export VLLM_DISTRIBUTED_EXECUTOR_CONFIG='{"placement_group_options":{"strategy":"SPREAD"}}'

# Default PACK strategy causes all workers to land on one node (wrong for multi-node)
# Symptom: placement group never schedules or all GPUs on same node
```

---

## NCCL Setup and Configuration

### NCCL's Role in Distributed vLLM

NCCL (NVIDIA Collective Communications Library) handles all GPU-to-GPU communication:

```
Communication type → NCCL collective used:

  Tensor Parallelism (TP):          All-reduce (sum partial weights)
  Pipeline Parallelism (PP):        P2P send/recv (pass activations)
  Expert Parallelism (EP):          All-to-all (route tokens to experts)
  KV Cache Transfer (PD disagg):    P2P send/recv (transfer KV blocks)
  EP All-gather variant:            All-gather + reduce-scatter
```

### Transport Layer

```
                 ┌─────────────────────────────────┐
                 │        NCCL Transport            │
                 ├────────────┬────────────────────┤
  Intra-node:   │  NVLink    │  PCIe               │
                │  600+ GB/s │  32-64 GB/s          │
                │  ~1 µs     │  ~5-10 µs            │
                ├────────────┴────────────────────┤ │
  Inter-node:   │  InfiniBand (IB)  │  RoCE        │ │
                │  200-400 Gb/s     │  100 Gb/s    │ │
                │  ~1-2 µs          │  ~2-5 µs     │ │
                └───────────────────┴──────────────┘
                
  GPUDirect RDMA: IB/RoCE with direct GPU HBM → NIC transfer
  (bypasses CPU, achieves full wire speed)
```

### Multi-Node NCCL Environment Variables

```bash
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# REQUIRED for multi-node
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
export VLLM_HOST_IP=<private_ip_of_this_node>   # Unique per node
export NCCL_IB_HCA=mlx5                          # IB HCA device name

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# COMMON FIXES for IB clusters
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
export GLOO_SOCKET_IFNAME=eth0    # Prevent Gloo initialization hang on IB
export NCCL_IB_TIMEOUT=23         # Increase timeout for slow/congested networks
export NCCL_IB_RETRY_CNT=7

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# GPUDirect RDMA
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Docker: run with --ipc=host --shm-size=16G
# Kubernetes: add IPC_LOCK capability + /dev/shm emptyDir volume

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DEBUGGING
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
export NCCL_DEBUG=INFO    # Show connectivity choices
export NCCL_DEBUG=TRACE   # Full protocol trace (very verbose)
```

### Kubernetes NCCL Config

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-worker
spec:
  template:
    spec:
      containers:
      - name: vllm
        image: vllm/vllm-openai:latest
        env:
        - name: VLLM_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NCCL_IB_HCA
          value: "mlx5"
        - name: GLOO_SOCKET_IFNAME
          value: "eth0"
        securityContext:
          capabilities:
            add: ["IPC_LOCK"]
        volumeMounts:
        - mountPath: /dev/shm
          name: dshm
      volumes:
      - name: dshm
        emptyDir:
          medium: Memory
          sizeLimit: 16Gi
      resources:
        limits:
          nvidia.com/gpu: 8
```

### NCCL Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Hang on `init_process_group` | Missing `VLLM_HOST_IP` or firewall | Set unique IP per node; open ports 6379, 29500+ |
| NCCL timeout during all-reduce | IB congestion or wrong HCA | Set `NCCL_IB_HCA=mlx5`, check `ibstat` |
| `GLOO` backend hang on IB | Wrong socket interface | Set `GLOO_SOCKET_IFNAME=eth0` |
| Low IB bandwidth | GPUDirect not enabled | Install `gdrcopy`, use `--ipc=host` |
| Placement group never schedules | Ray PACK strategy | Set SPREAD strategy via env var |
| Heartbeat failures in k8s | Raylet resource contention | Increase CPU/memory for raylet pods |

---

## Disaggregated Prefill/Decode (PD Disaggregation)

### The Bimodal Workload Problem

LLM inference has two fundamentally different phases that conflict when colocated:

```
┌────────────────────────────────────────────────────────────────┐
│                   Workload Characteristics                      │
├──────────────┬─────────────────────────┬───────────────────────┤
│ Phase        │ Prefill                 │ Decode                │
├──────────────┼─────────────────────────┼───────────────────────┤
│ Operation    │ Matrix-matrix multiply  │ Matrix-vector multiply│
│ Bottleneck   │ FLOPS (compute-bound)   │ HBM bandwidth         │
│ Duration     │ Seconds (long prompts)  │ Milliseconds/token    │
│ Batch type   │ Large, compute-heavy    │ Small, memory-light   │
│ Kernel type  │ High-throughput         │ Low-latency           │
└──────────────┴─────────────────────────┴───────────────────────┘

Problem: A single 4096-token prefill request monopolizes all GPUs in
the TP/EP group for 1-2 seconds, stalling all ongoing decode work.

Result: TTFT degrades (queue buildup) AND ITL degrades (stalled decodes)
        simultaneously — both SLOs violated at once.
```

### PD Disaggregation Architecture

Based on **DistServe** (Hao AI Lab, 2024), PD disaggregation runs prefill and decode on independent GPU pools:

```
                         ┌─────────────────────┐
                         │    Client Request    │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   Proxy / Router     │
                         │   (ZMQ ROUTER)       │
                         │   Heartbeat: 5s      │
                         └────┬────────────┬───┘
                              │            │
                 ┌────────────▼───┐   ┌────▼────────────┐
                 │  Prefill Pool  │   │   Decode Pool   │
                 │  N instances   │   │   M instances   │
                 │                │   │                 │
                 │  kv_role:      │   │  kv_role:       │
                 │  kv_producer   │   │  kv_consumer    │
                 │                │   │                 │
                 │  max_tokens=1  │   │  Receives KV    │
                 │  (prefill only)│   │  from prefill   │
                 └───────┬────────┘   └──────┬──────────┘
                         │                   │
                         │    KV Transfer     │
                         └───────────────────►
                           P2P NCCL / RDMA /
                           Mooncake / NIXL

Request flow:
  1. Client sends request to proxy
  2. Proxy routes to prefill server: compute KV cache for all prompt tokens
  3. Prefill server streams KV cache to decode server
  4. Decode server generates tokens, streams back to client
```

### KV Cache Transfer Mechanisms

#### 1. P2P NCCL Connector (Built-in)

```
Implementation: ncclSend / ncclRecv between prefill and decode GPUs
Channels: 16 NCCL channels by default (nccl_num_channels)
Mode: PUT_ASYNC — non-blocking transfer while prefill continues

Routing:
  Request ID: ___prefill_addr_{zmq_addr}___decode_addr_{zmq_addr}_{uuid}
  
Best for: Clusters where NCCL is already configured (same IB fabric)
Limitation: Requires network visibility between prefill and decode GPUs
```

#### 2. Mooncake Connector (RDMA-based)

```
Technology: GPU RDMA (GDR) — direct GPU memory to GPU memory via NIC
Transport:  InfiniBand with GPUDirect RDMA
Capability: Multi-hop KV transfer across cluster nodes
Requirement: gdrcopy installed, RDMA-capable NICs

Config:
  kv_connector: "MooncakeConnector"
  kv_connector_extra_config:
    mooncake_config_path: "/path/to/mooncake.json"
    
Advantage: Lower CPU involvement, higher bandwidth utilization
```

#### 3. LMCache + NIXL (Production-Grade, 2025)

```
Architecture:
  LMCache: extracts KV blocks from vLLM paged memory
  NIXL: NVIDIA's cross-platform transport library
    └── Supports NCCL, UCX, UCCL backends
    └── GPUDirect RDMA for maximum throughput
    
Performance: Matches or exceeds NVIDIA Dynamo benchmarks
Topologies:  1P1D (1 prefill : 1 decode)
             NP1D (N prefill : 1 decode) — for throughput scaling

Key design:  vLLM default block size of 16 tokens preserved
             → Compatible with prefix caching
```

#### 4. KV Offloading to CPU (Vertical Disaggregation)

```
Use case: KV cache overflow from GPU HBM to CPU DRAM
         (not prefill/decode disaggregation, but related concept)

Method: cudaMemcpyAsync (DMA) — 83.4 GB/s bidirectional
vs custom CUDA kernel — 68.5 GB/s (DMA is faster!)

V0.12.0 improvement: New contiguous physical block layout
  Before: ~32 KB effective block size → slow DMA (small transfers)
  After:  ~2 MB effective block size (Llama-3.1-8B) → fast DMA
  Impact: Up to 9× throughput improvement, 4× TTFT reduction

Config:
  --kv-transfer-config '{"kv_connector": "SharedStorageConnector",
                          "kv_role": "kv_both"}'
```

### PD Configuration Example (1 Prefill : 3 Decode)

```bash
# Prefill server (GPU 0) — compute-optimized
CUDA_VISIBLE_DEVICES=0 vllm serve meta-llama/Llama-3.1-8B-Instruct \
    --kv-transfer-config '{
      "kv_connector": "P2pNcclConnector",
      "kv_role": "kv_producer",
      "kv_buffer_size": "1e10",
      "kv_port": "21001",
      "kv_connector_extra_config": {
        "proxy_ip": "0.0.0.0",
        "proxy_port": "30001",
        "send_type": "PUT_ASYNC",
        "nccl_num_channels": "16"
      }
    }' \
    --gpu-memory-utilization 0.9 \
    --port 8100

# Decode servers (GPUs 1,2,3) — memory-bandwidth-optimized
# Lower GPU utilization to accommodate incoming KV cache
for GPU in 1 2 3; do
  CUDA_VISIBLE_DEVICES=$GPU vllm serve meta-llama/Llama-3.1-8B-Instruct \
      --kv-transfer-config '{
        "kv_connector": "P2pNcclConnector",
        "kv_role": "kv_consumer",
        "kv_buffer_size": "8e9"
      }' \
      --gpu-memory-utilization 0.7 \
      --port $((8200 + GPU)) &
done
```

### PD Benefits for MoE / EP Workloads

For large MoE deployments (e.g., DeepSeek-R1 with EP=64+):

```
Without PD disaggregation:
  All-to-all dispatch uses deepep_high_throughput for prefill
  AND deepep_low_latency for decode
  → But same GPU must serve both → kernel conflict → suboptimal

With PD disaggregation:
  Prefill instances: --all2all-backend deepep_high_throughput
    → Maximum throughput kernels, no latency constraints
  Decode instances:  --all2all-backend deepep_low_latency  
    → Minimum latency kernels, never blocked by prefill
  
  Straggler isolation:
    Slow prefill no longer blocks ongoing decode passes
    Each pool scales independently based on demand
```

---

## Expert Parallelism for MoE Models

### Why EP for MoE?

Mixture-of-Experts models (DeepSeek-V3/R1: 256 experts, 8 activated/token; Mixtral; Qwen3-MoE) have sparse activation at inference time. Standard TP over MoE layers wastes resources:

```
MoE MLP with TP (inefficient):
  All GPUs hold shards of ALL 256 experts
  Forward pass: all GPUs must synchronize even if their expert shard
  isn't in the top-8 for any token in the batch
  → Unnecessary synchronization, wasted compute

MoE MLP with EP (efficient):
  Each GPU holds COMPLETE copies of a SUBSET of experts
  Forward pass: tokens route to GPUs that own the needed experts
  → Only GPUs with active experts do work
  → No wasted synchronization
```

### EP Architecture

```
EP Size = TP_SIZE × DP_SIZE

Configuration: TP=1, DP=8 (8 GPUs total, EP=8)
  GPU 0: Experts 0-31    (32 of 256 experts)
  GPU 1: Experts 32-63
  GPU 2: Experts 64-95
  GPU 3: Experts 96-127
  GPU 4: Experts 128-159
  GPU 5: Experts 160-191
  GPU 6: Experts 192-223
  GPU 7: Experts 224-255

Layer-by-layer execution:
  Attention layers: Each GPU processes its own subset of requests (no TP sync)
  MoE expert layers: All-to-all dispatch → local compute → all-to-all combine
```

### All-to-All Communication in EP

```
EP All-to-All Dispatch/Combine:

  Input: Each of 8 GPUs has tokens needing experts from any GPU
  
  Dispatch (all-to-all):
    GPU 0 sends tokens needing experts on GPU 1 → GPU 1
    GPU 0 sends tokens needing experts on GPU 2 → GPU 2
    ... (each GPU sends to each other GPU simultaneously)
  
  Expert compute:
    Each GPU computes its local expert for all received tokens
  
  Combine (all-to-all):
    Each GPU sends computed outputs back to originating GPUs
    GPU 1 sends results back to GPU 0 for its tokens
    ...
  
  Communication volume: O(batch × hidden × EP)
  At EP=64: dominates total step time without optimization
```

### EP All-to-All Backends

| Backend | Optimized For | Notes |
|---|---|---|
| `allgather_reducescatter` | General (default) | NCCL-based, works for any EP+DP config |
| `deepep_high_throughput` | Prefill (large batches) | DeepEP kernel, optimized for bulk token routing |
| `deepep_low_latency` | Decode (latency-sensitive) | DeepEP kernel, minimizes round-trip time |
| `flashinfer_nvlink_one_sided` | Multi-node NVLink (GB200) | FlashInfer, one-sided NVLink transfer |
| `flashinfer_nvlink_two_sided` | Multi-node NVLink | FlashInfer, bidirectional NVLink |

### Expert Parallel Load Balancer (EPLB)

Real-world inference traffic causes unequal expert utilization — some experts receive many more tokens than others ("hot experts"), creating compute bottlenecks on the GPUs that hold them.

```
Without EPLB:
  GPU 2 (experts 64-95):  receives 40% of all tokens → bottleneck
  GPU 7 (experts 224-255): receives 5% of all tokens → idle
  
  Pipeline throughput = 1 / GPU_2_time (bottleneck drives latency)

With EPLB enabled (--enable-eplb):
  
  1. Track per-expert token counts over sliding window
     window_size: 1000 steps (configurable)
  
  2. Every step_interval (default: 3000 steps), compute new mapping:
     minimize max(expert_tokens_per_gpu) across all EP ranks
     
  3. Live weight shuffle: move expert weights in GPU memory
     (no model restart required)
  
  4. Optional: redundant experts (num_redundant_experts: 2)
     Duplicate hot experts onto multiple EP ranks
     Memory cost: ~2.4 GB per redundant expert per EP rank (DeepSeek-V3)
```

```bash
vllm serve deepseek-ai/DeepSeek-V3-0324 \
    --enable-expert-parallel \
    --enable-eplb \
    --eplb-config '{
      "window_size": 1000,
      "step_interval": 3000,
      "num_redundant_experts": 2,
      "log_balancedness": true
    }'
```

### Dual Batch Overlap (DBO) for High EP

At large EP degrees (64+), all-to-all communication time exceeds compute time, creating idle GPU cycles. DBO (`--enable-dbo`) hides communication behind compute:

```
Without DBO (EP=64, decode step):
  ┌──────────────────────────────────────────────────┐
  │ Attention │  A2A dispatch  │  Expert compute  │  A2A combine  │
  └──────────────────────────────────────────────────┘
               └──── wait ────┘                  └──── wait ────┘
               GPU idle here                     GPU idle here

With DBO (two interleaved micro-batches A and B):
  ┌──────────────────────────────────────────────────────────────┐
  │ Attn_A │ A2A_dispatch_A │ Expert_A + Attn_B │ A2A_combine_A  │
  │         │ Attn_B overlap │ A2A_dispatch_B   │ Expert_B+comb_B│
  └──────────────────────────────────────────────────────────────┘
  
  Communication of microbatch A overlaps with attention compute of B
  Communication of B overlaps with expert compute of A
  
  GPU utilization: 65% → 89% on 64-GPU WideEP clusters
```

---

## Data Parallelism Patterns

### Dense Model DP: Multi-Instance Serving

For dense models (non-MoE), data parallelism is implemented at the infrastructure layer:

```
Dense DP Setup:

  Instance 0 (8× GPU, TP=8): Handles requests 0, 4, 8, ...
  Instance 1 (8× GPU, TP=8): Handles requests 1, 5, 9, ...
  Instance 2 (8× GPU, TP=8): Handles requests 2, 6, 10, ...
  Instance 3 (8× GPU, TP=8): Handles requests 3, 7, 11, ...
  
  Load balancer: nginx / Ray Serve / Kubernetes Service
  
  Scaling: Linear throughput with replica count
  Limitation: Does NOT reduce per-request latency
              Does NOT increase per-request KV cache capacity
```

### MoE Model DP: Integrated DP+EP

For MoE models, DP is integrated within a single vLLM instance:

```
MoE DP+EP (8 GPUs, DP=8, EP=8):

  Each DP rank has:
    - Its own attention compute (replicated weights, independent input)
    - Shared expert compute (via all-to-all routing to expert GPUs)
  
  Step execution per DP rank:
    1. Attention: process local subset of requests independently
    2. All-to-all dispatch: route tokens to expert-owning GPUs
    3. Expert compute: process locally-owned experts
    4. All-to-all combine: return results to originating DP ranks
    5. Sample: generate next token for local requests
  
  Gang requirement: ALL DP ranks must be present for step to complete
                    (all-to-all is a blocking collective)
```

### DP Scaling Table

| Pattern | DP | TP | PP | EP | Use Case |
|---|---|---|---|---|---|
| Single large model (dense) | 1 | 4–8 | 1 | off | Dense model, single fast node |
| Multi-node dense | 1 | 8 | 2+ | off | Dense model, multi-node IB cluster |
| MoE single node | 8 | 1 | 1 | on | DeepSeek-V3 on 8-GPU H100 node |
| MoE multi-node WideEP | 16–64+ | 1 | 1 | on | DeepSeek at scale |
| MoE + TP attention | 4 | 2 | 1 | on | Hybrid: TP for attention + EP for experts |
| MoE + PD disagg | N prefill | 1 | 1 | on | Maximum throughput, independent scaling |

---

## Wide Expert Parallelism (WideEP)

WideEP is the pattern of running Expert Parallelism across a very large number of GPUs (64–128+) with a small number of experts per GPU:

```
Standard EP (8 GPUs, DeepSeek-V3 with 256 experts):
  Each GPU: 256/8 = 32 experts = ~170 GB of expert weights
  KV cache space per GPU: 80 GB - 170 GB = NEGATIVE (doesn't fit on H100!)
  
  → Need fewer experts per GPU to fit model

WideEP (64 GPUs, 256 experts):
  Each GPU: 256/64 = 4 experts = ~21 GB of expert weights
  KV cache space per GPU: 80 GB - 21 GB - overhead ≈ 55 GB per GPU
  Total KV cache: 55 × 64 = 3.5 TB across cluster
  
  Massive KV cache → huge batch sizes → high GPU utilization
```

### WideEP Performance Benchmarks (vLLM v0.11.0, 2025)

| Configuration | Hardware | Throughput |
|---|---|---|
| WideEP, DeepSeek-R1 | H200 cluster, ConnectX-7 IB | **2.2k tok/s/H200** |
| WideEP + PD disagg, DeepSeek-V3 | GB200, 4 prefill + 1 decode | **26.2K prefill TPGS** |
| WideEP improvement on GB200 vs H200 | NVFP4 + NVLink-C2C | **3–5× throughput improvement** |

### WideEP Multi-Node Deployment

```bash
# Node 1 (Primary — receives client traffic)
vllm serve deepseek-ai/DeepSeek-V3-0324 \
  --all2all-backend deepep_low_latency \
  --tensor-parallel-size 1 \
  --enable-expert-parallel \
  --data-parallel-size 16 \           # 16 DP ranks total
  --data-parallel-size-local 8 \      # 8 on this node
  --data-parallel-address 192.168.1.100 \
  --data-parallel-rpc-port 13345 \
  --api-server-count=8                # 8 parallel API server threads

# Node 2 (Headless worker — no API server, pure compute)
vllm serve deepseek-ai/DeepSeek-V3-0324 \
  --all2all-backend deepep_low_latency \
  --tensor-parallel-size 1 \
  --enable-expert-parallel \
  --data-parallel-size 16 \
  --data-parallel-size-local 8 \
  --data-parallel-start-rank 8 \      # ranks 8-15 on this node
  --data-parallel-address 192.168.1.100 \
  --headless                          # No API server

# InfiniBand clusters: required
export GLOO_SOCKET_IFNAME=eth0
```

### Ray Serve WideEP Fault Tolerance (Ray 2.55+)

```python
from ray.serve.llm import build_dp_deployment, LLMConfig, ModelLoadingConfig

llm_config = LLMConfig(
    model_loading_config=ModelLoadingConfig(
        model_id="deepseek-ai/DeepSeek-V3-0324"
    ),
    deployment_config=dict(
        num_replicas=2,          # 2 DP groups (16 GPUs each)
        max_ongoing_requests=100,
    ),
    engine_kwargs=dict(
        tensor_parallel_size=1,
        data_parallel_size=8,    # 8 ranks per DP group
        enable_expert_parallel=True,
        all2all_backend="deepep_low_latency",
    ),
)

# Gang scheduling: all 8 ranks in each group scheduled/recovered together
# On any rank failure: entire group marked unhealthy → torn down → rebuilt
app = build_dp_deployment(llm_config)
```

---

## Scaling Strategies and Decision Guide

### Decision Tree

```
┌────────────────────────────────────────────────────────────────────┐
│            Distributed Serving Strategy Decision Tree               │
└────────────────────────────────────────────────────────────────────┘

Model fits on 1 GPU (≤80 GB)?
  YES → Single GPU, no distributed parallelism

Model fits on 1 node (≤8× 80 GB = 640 GB)?
  YES
   └─ GPUs have NVLink (H100/A100/H200)?
       YES → Tensor Parallelism: --tensor-parallel-size 8
       NO  → Pipeline Parallelism within node: --pipeline-parallel-size N

Model requires multiple nodes?
  Dense model (Llama, Mistral, Qwen-dense)?
    → TP within node + PP across nodes
    → vllm serve --tensor-parallel-size 8 --pipeline-parallel-size N

  MoE model (DeepSeek-V3/R1, Mixtral, Qwen-MoE)?
    Low traffic, simple setup?
      → TP=8 within node (single node if fits)
    High throughput needed?
      → DP + EP (WideEP): --data-parallel-size N --enable-expert-parallel
    Maximum throughput + SLO isolation?
      → DP + EP + PD Disaggregation
      → Independent prefill/decode pools with KV transfer

Autoscaling needed?
  YES → Ray Serve with gang scheduling (Ray 2.55+ for WideEP)
  NO  → Direct Ray cluster deployment

Fault tolerance priority?
  HIGH → Smaller EP groups (smaller blast radius per failure)
         Ray Serve gang-aware recovery
  LOW  → Maximize EP width for throughput
```

### Throughput Scaling Patterns

```
Throughput scales with:

  1. TP: slightly super-linear (KV cache effect)
     TP=1→4: often 5-8× throughput (not 4×)
     
  2. PP: linear (adds stages, increases batch capacity)
     PP=1→2: ~2× throughput capacity
     
  3. DP: linear (independent replicas)
     DP=1→4: ~4× throughput
     
  4. EP (MoE): non-linear benefits from KV cache expansion
     EP=8→64: much more than 8× throughput due to batch size increase

  5. PD Disagg: multiplicative on top of DP
     Because prefill and decode scale independently:
     Total TTFT: scales with prefill pool size
     Total ITL:  scales with decode pool size
```

---

## Fault Tolerance and Recovery

### Failure Blast Radius by Strategy

| Strategy | Failure Unit | Recovery Action | Traffic Impact |
|---|---|---|---|
| TP | All TP ranks (all-or-nothing) | Restart entire TP group | All requests on that instance fail |
| PP | Entire pipeline | Restart all pipeline stages | All requests in pipeline fail |
| EP/WideEP | Single DP group | Ray Serve tears down + rebuilds group | Only that group's requests fail |
| PD Disagg | Single prefill/decode instance | Individual instance restart | Only in-flight requests to that instance |
| Dense DP | Single replica | LB routes around failed replica | Smooth degradation |

### Recovery Time Estimates

```
TP/PP recovery:
  Model load time: 30-120 seconds (weight loading from storage)
  Total: 1-3 minutes (full restart required)

EP/WideEP gang recovery (Ray Serve):
  Gang detected unhealthy → traffic rerouted immediately (< 1s)
  New gang spawned → model loaded → rejoins pool
  Total: 1-3 minutes recovery time, but other gangs continue serving

PD Disagg recovery:
  Prefill instance failure: decode instances continue with queued KV
  Decode instance failure: new decode instance starts (no prefill restart)
  KV in-flight: failed requests retried on surviving instances
```

---

## Hardware-Specific Guidance

| Hardware | Recommended Strategy | Notes |
|---|---|---|
| H100/H200 (NVLink, 8× per node) | TP=8 intra-node | Full 600 GB/s NVLink bandwidth |
| H100/H200 multi-node (IB ConnectX-7) | TP=8 + PP=N nodes | IB for PP, NVLink for TP |
| L40S (no NVLink) | PP or TP=2 | PCIe 4.0 limits TP scalability |
| A10G (AWS G5, no NVLink) | TP=1 + EP+DP for MoE | Use PP for multi-GPU dense models |
| GB200 NVL72 | WideEP + PD disagg | NVLink-C2C 900 GB/s, NVFP4 kernels |
| A100 single node | TP=8 | NVLink 3.0, 600 GB/s |
| Mixed GPU nodes | PP across nodes | PP doesn't require identical GPU types |

---

## See Also

- [Tensor Parallelism](tensor-parallelism.md) — Detailed TP mechanics, NCCL all-reduce, interconnect bandwidth
- [Pipeline Parallelism](pipeline-parallelism.md) — Stage splitting, bubble overhead, BatchScheduler
- [vLLM V1 Architecture](vllm-v1-architecture.md) — Unified scheduler, ZeroMQ IPC, V1 performance improvements

---

*Based on vLLM V1 architecture (v0.11.x+, 2025–2026). Sources: vLLM official documentation, DistServe (Hao AI Lab 2024), vLLM blog posts, Anyscale WideEP fault tolerance post, LMCache/NIXL documentation, Ray 2.55 release notes.*
