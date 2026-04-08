# Pipeline Parallelism for LLM Serving

Pipeline Parallelism (PP) distributes *contiguous groups of transformer layers* across multiple GPUs or nodes, with each device holding a complete set of layers (its "stage") rather than shards of each layer. Unlike tensor parallelism which requires high-bandwidth NVLink for all-reduce after every layer, PP only requires point-to-point communication at stage boundaries — making it the preferred strategy for multi-node deployments where inter-node bandwidth is limited.

---

## Table of Contents

1. [Motivation and When to Use PP](#motivation-and-when-to-use-pp)
2. [How Stage Splitting Works](#how-stage-splitting-works)
3. [Pipeline Communication Pattern](#pipeline-communication-pattern)
4. [Pipeline Bubble Overhead](#pipeline-bubble-overhead)
5. [Micro-Batching to Reduce Bubbles](#micro-batching-to-reduce-bubbles)
6. [1F1B Schedule](#1f1b-schedule)
7. [vLLM BatchScheduler and V1 Improvements](#vllm-batchscheduler-and-v1-improvements)
8. [PP vs TP: Detailed Comparison](#pp-vs-tp-detailed-comparison)
9. [Combining TP and PP](#combining-tp-and-pp)
10. [Layer Allocation Strategies](#layer-allocation-strategies)
11. [Memory and Communication Analysis](#memory-and-communication-analysis)
12. [PP Failure Modes](#pp-failure-modes)
13. [Configuration Reference](#configuration-reference)
14. [Hardware Guidance](#hardware-guidance)

---

## Motivation and When to Use PP

### The Memory Problem

Large language models exceed the memory capacity of individual nodes as well as single GPUs:

```
Model memory requirements (BF16 weights):

  Llama-3.1-8B:    ~16 GB   → fits on 1× A100/H100
  Llama-3.1-70B:   ~140 GB  → needs 2× 80GB GPUs minimum
  Llama-3.1-405B:  ~810 GB  → needs 8-16× 80GB GPUs
  DeepSeek-R1:     ~671 GB  → needs multi-node
  DeepSeek-V3:     ~685 GB  → needs multi-node
```

Tensor parallelism (TP) solves this within a node using NVLink, but once a model requires more GPUs than a single node provides — or when GPUs lack NVLink — **pipeline parallelism** becomes necessary.

### When PP Is Preferred Over TP

```
Use Pipeline Parallelism when:

  1. Model spans multiple nodes
     (inter-node bandwidth too slow for per-layer all-reduce)

  2. GPUs lack NVLink (PCIe-only: L40S, A10G, T4)
     (PCIe bandwidth ~32-64 GB/s vs NVLink ~600+ GB/s)

  3. Heterogeneous GPU clusters
     (different GPU types across nodes; PP ignores intra-node topology)

  4. Fault isolation: PP stages fail independently
     (though recovery still requires pipeline restart)

  5. Extremely large models requiring >8× 80GB GPUs
     (combine TP=8 within each node + PP across nodes)
```

---

## How Stage Splitting Works

Pipeline parallelism divides the model's transformer layers into `PP` sequential stages. Each stage is assigned to one GPU (or a group of GPUs in a combined TP+PP configuration).

### Layer Assignment

```
Llama-3.1-405B: 126 transformer layers
Pipeline stages: PP=4 (4 GPUs or 4 node groups)

Default even split:
  Stage 0 (GPU/Group 0): Embedding + Layers 0–31
  Stage 1 (GPU/Group 1): Layers 32–63
  Stage 2 (GPU/Group 2): Layers 64–95
  Stage 3 (GPU/Group 3): Layers 96–125 + LM Head

Memory per stage: ~202 GB (⅟₄ of total weights)
```

### Uneven Splitting

vLLM supports uneven layer allocation to handle models where the first/last stages have additional overhead (embedding tables, language model head):

```
Adjusted split for 405B (accounting for embedding overhead):
  Stage 0: Embedding + Layers 0–30   (31 layers + embedding cost)
  Stage 1: Layers 31–62              (32 layers)
  Stage 2: Layers 63–94              (32 layers)
  Stage 3: Layers 95–125 + LM Head   (31 layers + LM head)
```

Unlike TP, PP has no strict divisibility requirements — any partition of layers is valid.

---

## Pipeline Communication Pattern

### Point-to-Point Send/Recv

Between consecutive pipeline stages, activations are passed via **point-to-point NCCL send/recv** operations:

```
Forward pass through PP=4 pipeline:

  Stage 0                Stage 1                Stage 2                Stage 3
  ┌─────────┐            ┌─────────┐            ┌─────────┐            ┌─────────┐
  │ Emb     │            │         │            │         │            │         │
  │ L0-L31  │ ──SEND──► │ L32-L63 │ ──SEND──► │ L64-L95 │ ──SEND──► │ L96-L125│
  │         │            │         │            │         │            │ LM Head │
  └─────────┘            └─────────┘            └─────────┘            └─────────┘
       ▲                                                                     │
       │                                                                     ▼
  Input tokens                                                         Output logits

Communication: NCCL ncclSend / ncclRecv
Message size:  [batch_tokens × hidden_dim × dtype_bytes]
Example:        256 × 8192 × 2 = 4 MB per stage boundary
```

### Communication Volume Comparison

```
Tensor Parallelism (all-reduce, per layer):
  126 layers × 4 MB = 504 MB of communication per forward pass
  All-reduce: requires high-bandwidth NVLink (600+ GB/s)

Pipeline Parallelism (P2P send/recv, per stage boundary):
  3 boundaries × 4 MB = 12 MB of communication per forward pass
  P2P: works over InfiniBand (25-50 GB/s) or even Ethernet
```

PP reduces total communication volume by ~42× for 126-layer models with 3 stage boundaries vs 125 all-reduces.

---

## Pipeline Bubble Overhead

### The Naive Pipeline Problem

Without micro-batching, a naive PP pipeline has severe idle time — the "pipeline bubble":

```
Naive PP=4 pipeline (1 batch, 4 stages):

Time →

Stage 0: [COMPUTE]────────────────────────────────────────[IDLE]
Stage 1:          [IDLE]────[COMPUTE]──────────────────────[IDLE]
Stage 2:                            [IDLE]────[COMPUTE]─────[IDLE]
Stage 3:                                              [IDLE]─[COMPUTE]

Total time = 4 × compute_time (per stage)
Efficiency  = 1/4 = 25% GPU utilization

Bubble fraction = (PP - 1) / PP = 3/4 for this case
```

### Bubble Fraction Formula

For a pipeline of `PP` stages processing `M` micro-batches:

```
bubble_fraction = (PP - 1) / (M + PP - 1)

Where:
  PP = number of pipeline stages
  M  = number of micro-batches

Examples:
  PP=4, M=1:   bubble = 3/4  = 75%  (terrible)
  PP=4, M=4:   bubble = 3/7  = 43%  (poor)
  PP=4, M=8:   bubble = 3/11 = 27%  (moderate)
  PP=4, M=16:  bubble = 3/19 = 16%  (acceptable)
  PP=4, M=32:  bubble = 3/35 = 9%   (good)
  PP=2, M=8:   bubble = 1/9  = 11%  (good — lower PP is better)
```

Key insight: **smaller PP and larger batch sizes both reduce bubble overhead**. This is why PP=2 across 2 nodes is strongly preferred over PP=4 when possible.

---

## Micro-Batching to Reduce Bubbles

Micro-batching splits a large batch into `M` smaller sub-batches ("micro-batches") that flow through the pipeline in interleaved fashion:

```
PP=4, M=4 micro-batches (mb0, mb1, mb2, mb3):

Time →       t0    t1    t2    t3    t4    t5    t6
              
Stage 0:    [mb0] [mb1] [mb2] [mb3] [idle][idle][idle]
Stage 1:    [idle][mb0] [mb1] [mb2] [mb3] [idle][idle]
Stage 2:    [idle][idle][mb0] [mb1] [mb2] [mb3] [idle]
Stage 3:    [idle][idle][idle][mb0] [mb1] [mb2] [mb3]

Bubble slots: 6 idle slots out of 4×7=28 total
Efficiency: (28-6)/28 = 78.6%
Bubble fraction: (PP-1)/(M+PP-1) = 3/7 = 43%
```

As M grows, stages stay busy with successive micro-batches while waiting for the previous one to propagate:

```
PP=4, M=8 micro-batches (optimal for medium-size batches):

Time →    t0  t1  t2  t3  t4  t5  t6  t7  t8  t9  t10
Stage 0:  [0] [1] [2] [3] [4] [5] [6] [7]  ↑   ↑   ↑
Stage 1:   ↑  [0] [1] [2] [3] [4] [5] [6] [7]  ↑   ↑
Stage 2:   ↑   ↑  [0] [1] [2] [3] [4] [5] [6] [7]  ↑
Stage 3:   ↑   ↑   ↑  [0] [1] [2] [3] [4] [5] [6] [7]

(↑ = bubble/idle)
Efficiency: (PP=4, M=8) → bubble = 3/11 ≈ 27%
```

---

## 1F1B Schedule

The **1 Forward 1 Backward (1F1B)** schedule was developed for training to minimize in-flight activations (memory) while maintaining high pipeline utilization. For inference (decode-only, no backward pass), a variant is used:

### 1F1B for Inference

In LLM inference, there is no backward pass, so 1F1B is adapted to the **interleaved microbatch** pattern:

```
1F1B Inference Schedule (PP=4, M=4):

                Startup         Steady State         Drain
Stage 0:  [F0][F1][F2][F3]  [F4][F5]...            [.][.][.]
Stage 1:      [F0][F1][F2]  [F3][F4][F5]...         [.][.][.][.]
Stage 2:          [F0][F1]  [F2][F3][F4][F5]...      [.][.][.][.][.]
Stage 3:              [F0]  [F1][F2][F3][F4][F5]...  [.][.][.][.][.][.]

Properties:
- Pipeline depth in "startup": PP-1 = 3 bubble slots
- Steady state: all stages active simultaneously
- Drain: PP-1 bubble slots again
- Total bubble: 2×(PP-1) = 6 slots for any M ≥ PP
```

### Interleaved Schedule (Virtual Stages)

For even better efficiency, the **interleaved schedule** assigns each GPU multiple non-contiguous layer groups (virtual stages):

```
Standard PP=4 (1 virtual stage per GPU):
  GPU 0: Layers 0-31
  GPU 1: Layers 32-63
  GPU 2: Layers 64-95
  GPU 3: Layers 96-125

Interleaved PP=4 (2 virtual stages per GPU):
  GPU 0: Layers 0-15, 64-79   (virtual stages 0 and 2)
  GPU 1: Layers 16-31, 80-95  (virtual stages 0 and 2)
  GPU 2: Layers 32-47, 96-110 (virtual stages 1 and 3)
  GPU 3: Layers 48-63, 111-125 (virtual stages 1 and 3)

Bubble fraction with V virtual stages per GPU:
  bubble = (PP - 1) / (V × M + PP - 1)
  
With V=2, M=4: bubble = 3/11 ≈ 27% (same as M=8 in standard)
Reduces activation memory proportional to V (fewer in-flight micro-batches)
```

---

## vLLM BatchScheduler and V1 Improvements

### V0 Pipeline Parallelism (Virtual Engines)

In vLLM V0, PP was implemented via **virtual engines** — multiple independent scheduler + block manager + cache engine instances, one per pipeline stage:

```
V0 PP Architecture:

  Stage 0:  [Scheduler 0] + [BlockManager 0] + [CacheEngine 0]
  Stage 1:  [Scheduler 1] + [BlockManager 1] + [CacheEngine 1]
  Stage 2:  [Scheduler 2] + [BlockManager 2] + [CacheEngine 2]
  Stage 3:  [Scheduler 3] + [BlockManager 3] + [CacheEngine 3]

Problems:
  - No centralized scheduler → cannot globally optimize microbatch allocation
  - Each stage schedules independently → microbatch imbalance across stages
  - No mechanism to predict ideal micro-batch count for bubble minimization
```

### V1 BatchScheduler (PR #19873)

vLLM V1 introduces a `BatchScheduler` with two-phase scheduling:

```
V1 PP Architecture:

  Centralized Scheduler (EngineCore process)
       │
       ├── Phase 1: Collect all schedulable requests
       │   scheduled_queue = [req_1, req_2, ..., req_N]
       │
       └── Phase 2: Drain queue in balanced micro-batches
           target_tokens_per_microbatch = total_tokens / num_microbatches
           micro_batch_0 = [req_1, ..., req_k]   ← ~target tokens
           micro_batch_1 = [req_k+1, ..., req_j] ← ~target tokens
           ...

  All micro-batches dispatched to pipeline with controlled token balance
```

### BatchScheduler Performance Results

```
Benchmark: 256 requests, 4-node LAN, RTX 4070 GPUs, PP=4

Without BatchScheduler:  146 seconds total
With BatchScheduler:      70 seconds total
Improvement:            2.08× speedup

Cause: balanced micro-batches prevent "last micro-batch" stragglers
       where one stage has 1 token and other stages have 100+
```

### Comparison: V0 vs V1 PP

| Feature | V0 PP | V1 PP |
|---|---|---|
| Scheduling | Per-stage virtual engines | Centralized BatchScheduler |
| Micro-batch balance | No global optimization | Two-phase balanced allocation |
| KV cache management | Per-stage block managers | Centralized with unified KV view |
| Bubble minimization | Manual tuning required | Automated token budget balancing |
| Performance | Baseline | ~2× better at batch workloads |

---

## PP vs TP: Detailed Comparison

| Dimension | Tensor Parallelism | Pipeline Parallelism |
|---|---|---|
| **Parallelism type** | Intra-layer (weight splitting) | Inter-layer (stage splitting) |
| **Latency reduction** | ✅ Yes — all GPUs work in parallel per layer | ❌ No — sequential stages increase latency |
| **Throughput scaling** | ✅ Yes (+ super-linear KV cache effect) | ✅ Yes (higher batch capacity) |
| **Memory reduction** | ✅ Yes — weight sharding | ✅ Yes — layer partitioning |
| **Communication** | All-reduce every layer | P2P send/recv at stage boundaries |
| **Communication volume** | High (every layer) | Low (3 boundaries for PP=4) |
| **Bandwidth requirement** | Very high (NVLink: 600+ GB/s) | Moderate (IB: 25-50 GB/s) |
| **Best interconnect** | NVLink (intra-node) | InfiniBand, Ethernet (inter-node) |
| **Divisibility constraint** | Strict (must divide head count, etc.) | None — any layer split is valid |
| **Bubble overhead** | None (fully parallel) | Significant — requires micro-batching |
| **Failure blast radius** | Entire TP group (all-or-nothing) | Entire pipeline (all stages) |
| **Heterogeneous HW support** | Poor (all GPUs must be identical) | Good (different layer counts per GPU) |
| **Multi-node support** | Possible but expensive (needs fast IB) | Designed for multi-node |
| **Best use case** | Same-node GPUs with NVLink | Cross-node, slow interconnects |

### Latency Impact

```
Serving a single request (no batching):

  Tensor Parallelism (TP=8):
    Each layer: 1 all-reduce + parallel compute
    Total latency: ~1× per-layer compute (parallel)
    
  Pipeline Parallelism (PP=4):
    Each stage: sequential compute + P2P transfer
    Total latency: 4× per-stage compute (sequential)
    
  PP adds latency proportional to pipeline depth.
  For latency-sensitive applications: prefer TP over PP.
  For throughput-maximizing applications: PP is acceptable.
```

---

## Combining TP and PP

The standard production pattern for multi-node large model serving:

```
Strategy: TP within each node, PP across nodes

Example: Llama-3.1-405B on 2 nodes × 8 GPUs (H100):

  Node 0 (PP stage 0):
    GPU 0–7  → TP group (NVLink)
    Handles: Embedding + Layers 0–62

  Node 1 (PP stage 1):
    GPU 0–7  → TP group (NVLink)
    Handles: Layers 63–125 + LM Head

Communication:
  Within node: NVLink all-reduce (600 GB/s) → fast
  Between nodes: IB P2P send/recv (25 GB/s) → acceptable
```

### Configuration

```bash
# 2 nodes × 8 GPUs each
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray

# 4 nodes × 8 GPUs each (larger model)
vllm serve deepseek-ai/DeepSeek-R1 \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 4 \
    --distributed-executor-backend ray
```

### Memory Budget with TP+PP

```
Llama-3.1-405B: ~810 GB total weights

With TP=8, PP=2 (16 GPUs total, each 80 GB):
  Total VRAM: 16 × 80 GB = 1280 GB
  Weight memory per GPU: 810 / 16 ≈ 50.6 GB
  Available for KV cache: 80 - 50.6 - overhead ≈ 25 GB per GPU
  Total KV cache: 25 × 16 = 400 GB (plenty for large batches)
```

---

## Layer Allocation Strategies

### Even Splitting

The simplest strategy — divide total layers by PP:

```python
layers_per_stage = total_layers // PP
# Stage i gets: layers [i * layers_per_stage : (i+1) * layers_per_stage]
```

**Problem:** Stage 0 has embedding lookup (memory-intensive), Stage `PP-1` has LM head (compute-intensive). These make stages 0 and PP-1 slower than middle stages.

### Load-Balanced Splitting

Accounts for non-uniform layer costs:

```
Profiling-based assignment:
  Measure compute time per layer type
  Allocate layers to equalize stage compute times

Example (hypothetical):
  Embedding:      15ms
  Transformer layer: 10ms each
  LM Head:        12ms

  Target per stage: (15 + 126×10 + 12) / 4 = 288.25ms

  Stage 0: Emb (15ms) + 27 layers (270ms) = 285ms  ← 27 layers
  Stage 1: 29 layers (290ms) = 290ms               ← 29 layers
  Stage 2: 29 layers (290ms) = 290ms               ← 29 layers
  Stage 3: 41 layers + LM Head (410+12=422ms)      ← shouldn't happen

Ideal: vLLM supports custom layer allocation via model-specific config
```

### First/Last Stage Overhead

For models with large vocabulary (embedding tables are large):

```
LLaMA vocab size: 128,256 tokens
Embedding table size: 128,256 × 8192 × 2 bytes = ~2.1 GB

This is on Stage 0 only. With TP=8:
  Each GPU on Stage 0: 128,256/8 × 8192 × 2 ≈ 262 MB
  Manageable, but still causes slight asymmetry
```

---

## Memory and Communication Analysis

### Activation Memory at Stage Boundaries

Each stage must buffer activations being passed to the next stage:

```
Buffer size per stage boundary:
  = max_batch_tokens × hidden_dim × dtype_bytes
  
Example (Llama-3.1-70B, BF16, 512 token batch):
  = 512 × 8192 × 2 = 8.4 MB per boundary

With PP=4: 3 boundaries × 8.4 MB = 25.2 MB in-flight per request batch
```

vLLM pre-allocates these buffers at startup to avoid dynamic allocation overhead.

### Stage Compute Parity

For optimal pipeline efficiency, each stage should take the same compute time:

```
Compute time per stage = (total_compute_time) / PP

If stages are imbalanced:
  Fast stage:  waits for slow stage → bubble
  Slow stage:  becomes bottleneck → limits pipeline throughput

Pipeline throughput limited by:
  max(stage_compute_time) + communication_overhead
```

---

## PP Failure Modes

### Bubble Amplification at Low Batch Rates

```
Problem scenario:
  PP=4 pipeline, configured for M=16 micro-batches
  Actual traffic: only 2 requests arrive
  
  With M=2 and PP=4: bubble = (4-1)/(2+4-1) = 3/5 = 60%
  
  60% idle time vs configured 16% target
  Low-traffic periods waste GPU resources significantly
```

**Mitigation:** Use dynamic micro-batch sizing; vLLM V1 BatchScheduler adapts automatically.

### Stage Imbalance

```
Symptom: Stage N-1 (last stage with LM head) is always slower
Effect:  Stages 0 through N-2 finish and wait for Stage N-1

Diagnosis: NCCL profiling shows Stage N-1 send time consistently late
Resolution: Reduce layers in Stage N-1, increase in middle stages
```

### P2P Buffer Deadlock

```
Problem: Large activation tensors + small NCCL buffers → NCCL blocking
         Stage N tries to send, NCCL buffer full
         Stage N+1 not ready to recv (still computing)
         Deadlock
         
Resolution: Pre-allocate larger NCCL buffers
            Reduce batch size
            Check NCCL_BUFFSIZE environment variable
```

### Node Failure in Multi-Node PP

```
PP failure blast radius: entire pipeline
  Node 0 fails → Stage 0 gone → Stages 1-3 all stall

No partial recovery: all stages must restart together
This is the fundamental availability risk of PP
(vs. EP/DP where individual ranks can be recovered independently)
```

---

## Configuration Reference

### Basic PP Setup

```bash
# Single model, 2 nodes
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray

# Combined TP+PP (most common production setup)
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray

# With custom chunk size for balanced micro-batches
vllm serve meta-llama/Llama-3.1-405B-Instruct \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 4 \
    --max-num-seqs 256 \
    --max-num-batched-tokens 32768 \
    --distributed-executor-backend ray
```

### Ray Setup for Multi-Node PP

```bash
# Head node
ray start --head --port=6379 --node-ip-address=<head_ip>

# Worker nodes
ray start --address=<head_ip>:6379 --node-ip-address=<this_node_ip>

# Launch vLLM (on head node)
vllm serve /path/to/model \
    --tensor-parallel-size 8 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray

# Alternative: ray symmetric-run (2025+)
ray symmetric-run \
    --address <head_ip>:6379 \
    --min-nodes 2 \
    --num-gpus 8 \
    -- vllm serve /path/to/model \
       --tensor-parallel-size 8 \
       --pipeline-parallel-size 2
```

### Environment Variables

```bash
# Per-node IP (required for multi-node)
export VLLM_HOST_IP=<node_private_ip>

# IB device for inter-node communication
export NCCL_IB_HCA=mlx5

# Prevent Gloo hang on IB clusters
export GLOO_SOCKET_IFNAME=eth0

# Debug pipeline stalls
export NCCL_DEBUG=INFO
export VLLM_LOGGING_LEVEL=DEBUG
```

---

## Hardware Guidance

| Hardware Configuration | PP Strategy | Notes |
|---|---|---|
| H100/A100 single node (8 GPUs) | PP=1 (use TP only) | NVLink makes TP sufficient |
| H100 2 nodes (16 GPUs, IB) | TP=8, PP=2 | Standard large model setup |
| H100 4 nodes (32 GPUs, IB) | TP=8, PP=4 | For 405B+ models |
| L40S single node (no NVLink) | PP=4 (layers across GPUs) | PCIe limits TP; use PP even within node |
| A10G (AWS G5, no NVLink) | PP within node if needed | TP limited to TP=2 per PCIe complex |
| Mixed GPU types | PP across GPU types | PP doesn't require identical GPUs |
| GB200 NVL72 | TP=72 sufficient (no PP needed) | NVLink-C2C across 72 GPUs |

---

## See Also

- [Tensor Parallelism](tensor-parallelism.md) — Intra-layer weight sharding with NVLink all-reduce
- [Distributed LLM Serving](distributed-llm-serving.md) — Ray integration, multi-node setup, MoE expert parallelism
- [vLLM V1 Architecture](vllm-v1-architecture.md) — Unified scheduler and V1 improvements to PP

---

*Based on vLLM V1 architecture (v0.11.x+, 2025–2026). Sources: vLLM GitHub RFC #11945 (PP V1), PR #19873 (BatchScheduler), vLLM official documentation, PipeSwitch and Megatron-LM papers.*
