# vLLM V1 Architecture

vLLM V1 is a ground-up re-architecture of the vLLM inference engine, made generally available in **v0.11.0 (December 2025)**, at which point V0 was officially deprecated. After 18 months of V0 development, the codebase had accumulated significant technical debt: features built independently couldn't be cleanly combined, the scheduler/worker colocation created architectural asymmetry, and CPU overhead was becoming the dominant bottleneck for fast GPUs. V1 addresses these issues with 8 core architectural changes that together deliver up to **1.7× throughput improvement** over V0.

---

## Table of Contents

1. [Why V1? The V0 Problems](#why-v1-the-v0-problems)
2. [Change 1: ZeroMQ Execution Loop](#change-1-zeromq-execution-loop)
3. [Change 2: Unified Scheduler](#change-2-unified-scheduler)
4. [Change 3: Zero-Overhead Prefix Caching](#change-3-zero-overhead-prefix-caching)
5. [Change 4: Symmetric Tensor-Parallel Architecture](#change-4-symmetric-tensor-parallel-architecture)
6. [Change 5: Persistent Batch and Efficient Input Preparation](#change-5-persistent-batch-and-efficient-input-preparation)
7. [Change 6: torch.compile and Piecewise CUDA Graphs](#change-6-torchcompile-and-piecewise-cuda-graphs)
8. [Change 7: FlashAttention 3 Integration](#change-7-flashattention-3-integration)
9. [Change 8: Multiprocessing for Multimodal and Preprocessing](#change-8-multiprocessing-for-multimodal-and-preprocessing)
10. [New KV Cache Manager](#new-kv-cache-manager)
11. [Performance Results](#performance-results)
12. [V0 vs V1 Comparison Summary](#v0-vs-v1-comparison-summary)
13. [Migration from V0 to V1](#migration-from-v0-to-v1)
14. [Known Limitations and Roadmap](#known-limitations-and-roadmap)

---

## Why V1? The V0 Problems

### V0 Architecture Constraints

By mid-2024, vLLM V0 had grown to support dozens of models, quantization methods, attention backends, and parallelism strategies — but the foundational design had not scaled with this growth:

```
V0 Core Problems:

1. CPU overhead dominated GPU compute for fast GPUs
   Problem: Llama-3-8B on H100 completes in ~5ms
            Python scheduling + tensor preparation took ~3-5ms
            CPU was 40-50% of total step time → GPU idle half the time

2. Scheduler/Worker 0 colocation caused asymmetry
   Problem: Worker 0 (rank 0) shared a process with the scheduler
            Asymmetric topology → rank 0 had different timing characteristics
            Complex debugging; harder to reason about distributed state

3. V0 treated prefill and decode as fundamentally different
   Problem: Separate scheduling paths for prefill vs decode
            Could not mix them efficiently in a single batch
            Chunked prefill required complex feature flags

4. Prefix caching disabled by default
   Problem: V0's block metadata structures had high Python overhead
            Enabling prefix caching caused measurable throughput degradation
            → Feature existed but wasn't practical to use

5. No clean multimodal integration
   Problem: Vision encoding was synchronous and blocking
            Image preprocessing blocked the GPU compute loop
            Repeated encoding of same image in multi-turn conversations

6. Input tensor recreation from scratch every step
   Problem: Token IDs, positions, KV block mappings rebuilt every step
            At 5ms step time, tensor construction was ~1ms → 20% overhead
```

### V0 Deprecation Timeline

```
2024 Q1-Q2:  V1 design RFC published internally
2024 Q3-Q4:  V1 experimental preview (vllm.V1)
2025 Q1-Q2:  V1 becomes default (VLLM_USE_V1=1 by default)
2025 Q4:     V0 officially deprecated in v0.11.0 (December 2025)
2026:        V0 code removed; V1 is the only engine
```

---

## Change 1: ZeroMQ Execution Loop

### The Problem: CPU-GPU Coupling in V0

In V0, the execution loop ran all tasks in a single thread/process:

```
V0 Execution Loop (single process):

  ┌─────────────────────────────────────────────────────┐
  │  Loop iteration (repeats every ~5ms for H100)        │
  │                                                       │
  │  1. Tokenize new requests       (1-2ms CPU, Python)  │
  │  2. Schedule requests           (0.5ms CPU, Python)  │
  │  3. Build input tensors         (1ms CPU, NumPy)     │
  │  4. GPU forward pass            (5ms GPU compute)    │
  │  5. Sample next tokens          (0.5ms GPU)          │
  │  6. Detokenize outputs          (1ms CPU, Python)    │
  │  7. Stream responses            (0.5ms CPU, FastAPI) │
  │                                                       │
  │  Total: ~9.5ms wall clock for 5ms of GPU work        │
  │  GPU utilization: ~50%                               │
  └─────────────────────────────────────────────────────┘
```

### V1 Solution: ZeroMQ IPC and Process Isolation

V1 separates the execution into two distinct processes communicating via **ZeroMQ** (ZMQ) IPC sockets:

```
V1 Process Separation:

┌─────────────────────────────────────┐    ┌─────────────────────────────────┐
│      API Server Process             │    │      EngineCore Process          │
│                                     │    │                                   │
│  FastAPI HTTP server                │    │  Scheduler                        │
│  Request queue management           │    │  KV cache manager                 │
│  Tokenization (concurrent)          │◄──►│  Persistent batch manager         │
│  Detokenization (concurrent)        │ZMQ │  GPU model executor               │
│  Response streaming                 │IPC │  Worker coordination              │
│  Multimodal preprocessing           │    │  (pure compute loop)              │
│                                     │    │                                   │
└─────────────────────────────────────┘    └─────────────────────────────────┘

Key benefit: EngineCore runs a tight GPU loop with NO Python blocking
             API server handles all I/O-bound work in parallel
```

### ZMQ Communication Protocol

```
Request flow:
  1. API Server tokenizes request → sends (request_id, tokens) to EngineCore via ZMQ PUSH
  2. EngineCore adds to scheduler queue
  3. EngineCore runs GPU step → samples output tokens
  4. EngineCore sends (request_id, new_token) to API Server via ZMQ PUSH
  5. API Server detokenizes → streams to client

IPC transport: Unix domain sockets (intra-machine)
               TCP sockets (if EngineCore is on remote node)
Serialization: msgpack (fast, zero-copy for numpy arrays)

Latency: ZMQ IPC adds ~50-200 µs overhead
         (acceptable given 5ms+ GPU steps)
```

### Throughput Impact

```
V0 CPU overhead model:
  effective_step_time = max(gpu_time, cpu_time)  [serialized]
  At 5ms GPU: cpu=5ms → 50% GPU utilization

V1 CPU overhead model:
  effective_step_time = max(gpu_time, zmq_overhead)  [overlapped]
  At 5ms GPU: zmq=0.2ms → ~96% GPU utilization

Result: 1.7× throughput improvement at typical batch sizes
```

---

## Change 2: Unified Scheduler

### V0 Scheduler: Separate Prefill and Decode Logic

```
V0 Scheduler State Machine:

  Request states:
    WAITING   → in queue, not started
    RUNNING   → being decoded (prefill done)
    SWAPPED   → preempted to CPU

  V0 prefill schedule:
    Selects WAITING requests
    Allocates ALL prompt tokens at once
    Transitions to RUNNING state
    
  V0 decode schedule:
    Selects RUNNING requests
    Processes 1 token each
    
  Problems:
    Chunked prefill required flag: --enable-chunked-prefill
    Mixed batches (prefill + decode) had code duplication
    Pipeline parallelism needed per-stage virtual engines
    No single unified view of "what runs this step"
```

### V1 Scheduler: Unified Token Budget

V1's scheduler represents ALL scheduling decisions as a single dictionary:

```python
# V1 scheduler output — one unified representation
scheduled_requests: dict[request_id, num_tokens]

# Examples:
# Pure prefill:    {"req_1": 512}            # 512 prompt tokens
# Chunked prefill: {"req_1": 100}            # 100 prompt tokens (chunked)
# Decode:          {"req_2": 1}              # 1 output token
# Mixed batch:     {"req_1": 100, "req_2": 1, "req_3": 75}
#                  (chunked prefill + decode + another prefill chunk — same step!)
```

### Scheduler Logic

```
V1 Scheduling Algorithm:

  Token budget: max_num_batched_tokens (configurable, e.g., 32768)
  
  Each step:
    1. Sort waiting requests by priority (FCFS or custom)
    2. For each request, determine num_tokens to schedule:
         If new request:    min(remaining_prompt_tokens, chunk_size)
         If decoding:       1  (next output token)
         If chunked:        next chunk of remaining prompt
    3. Accumulate tokens until budget exhausted
    4. Emit {request_id: num_tokens} dict to workers
    5. Workers execute exactly this many tokens per request
  
  Result: chunked prefill is ON BY DEFAULT (not a flag)
          prefill and decode naturally coexist in same batch
          no special-casing for any request type
```

### Benefits of Unified Scheduler

```
1. Chunked prefill default-on:
   Long prompts no longer block decode requests
   Each step: some tokens from new request + 1 token per ongoing decode
   
   Example: 4096-token prompt with chunk_size=512:
     Step 1:  {"new_req": 512, "ongoing_req_1": 1, "ongoing_req_2": 1}
     Step 2:  {"new_req": 512, "ongoing_req_1": 1, "ongoing_req_2": 1}
     ...      (8 steps to complete prefill, decode requests not blocked)

2. TTFT improvement:
   Decode requests get tokens every step even during heavy prefill load
   No "prefill monopolizes the GPU" problem
   
3. Pipeline PP simplification:
   V1 PP uses centralized BatchScheduler (not virtual engines)
   Single scheduler can optimize microbatch balance globally
   
4. Feature composability:
   Any combination of: chunked prefill + prefix caching + TP + PP
   All work together — no conflict flags needed
```

---

## Change 3: Zero-Overhead Prefix Caching

### V0 Prefix Caching: High CPU Overhead

```
V0 KV cache block metadata:
  Each block tracked as Python dict with multiple attributes:
  {
    "block_hash": hash_value,
    "ref_count": int,
    "last_accessed": timestamp,
    "physical_block_id": int,
    "tokens": list[int],
    "num_hashes": int,
    ...
  }

  Cache eviction: LRU over Python dict → O(n) scan → slow
  Overhead: non-trivial % of scheduler time at high request rates
  
  Decision: disable prefix caching by default
            (throughput decrease at 0% hit rate outweighed benefits)
```

### V1 Prefix Caching: O(1) Operations

```
V1 redesigned data structures for minimal Python object creation:

  Block hash: uint64_t (single integer, computed incrementally)
  Block state: enum (CACHED / FREE / ALLOCATED)
  
  Incremental hash chain:
    block_hash = hash(prev_block_hash || tokens_in_block || optional_extra_metadata)
    
    This means:
      Sharing a 1024-token prefix with block_size=16:
        Block 0: hash(0 || tokens[0:16])
        Block 1: hash(block_0_hash || tokens[16:32])
        ...
        Block 63: hash(block_62_hash || tokens[1008:1024])
      
      Cache lookup: just compare final block_hash
      Cache hit: all 64 blocks found in O(64) = O(prefix_len/block_size)
      
  Eviction: O(1) via free list + hash map (not O(n) scan)
  
  Result: <1% throughput degradation at 0% cache hit rate
          → PREFIX CACHING NOW ON BY DEFAULT in V1
```

### Prefix Caching with TP

```
In V1's symmetric TP architecture:
  All TP workers cache the same block hash → prefix hit on all workers simultaneously
  Scheduler broadcasts "use cached blocks [X, Y, Z]" → all workers skip KV recompute
  KV blocks are stored per-worker (each worker holds KV for its head subset)
  
  System prompt caching example:
    1000-token system prompt: 
      First request:  compute all 1000 tokens, cache 62 blocks
      Second request: prefix hit → skip recompute, only compute 938→1000 overlap + new tokens
      Savings: up to 60-70% latency reduction for requests with shared system prompts
```

---

## Change 4: Symmetric Tensor-Parallel Architecture

### V0: Asymmetric Worker Topology

```
V0 Process Layout (TP=4 example):

  Process 0 (rank 0):
  ┌────────────────────────────────────┐
  │  Scheduler                         │
  │  BlockManager                      │
  │  CacheEngine (rank 0)              │
  │  Worker 0 (GPU 0)                  │
  │  ↕ IPC to rank 1, 2, 3            │
  └────────────────────────────────────┘

  Process 1 (rank 1): Worker 1 (GPU 1) only
  Process 2 (rank 2): Worker 2 (GPU 2) only
  Process 3 (rank 3): Worker 3 (GPU 3) only
  
  Problem: Rank 0 is "heavier" (scheduler overhead)
           Different scheduling latency for rank 0 vs ranks 1-3
           Hard to reason about "what state does rank 0 hold?"
           Rank 0 must always be alive for system to function
```

### V1: Symmetric Workers + Incremental Diff Protocol

```
V1 Process Layout (TP=4 example):

  EngineCore Process:
  ┌─────────────────────────────────┐
  │  Scheduler (SEPARATE PROCESS)   │
  │  BlockManager                   │
  │  Persistent Batch state         │
  │  ↕ ZMQ IPC to all workers       │
  └─────────────────────────────────┘

  Worker 0 (rank 0, GPU 0): ← identical to all other workers
  ┌─────────────────────────────────┐
  │  GPU model execution            │
  │  Cached request state           │
  │  ZMQ recv: apply incremental diff│
  └─────────────────────────────────┘
  
  Worker 1 (rank 1, GPU 1): identical
  Worker 2 (rank 2, GPU 2): identical
  Worker 3 (rank 3, GPU 3): identical

Communication optimization:
  V0: Full scheduler state sent to rank 0 every step
  V1: Workers cache local state; scheduler sends DIFFS ONLY
  
  Diff = {new_tokens: [...], new_block_table_updates: [...], finished_reqs: [...]}
  Size reduction: 10-100× smaller messages per step
```

### Distributed State Management

```
State synchronization model:

  EngineCore (authoritative):
    - Maintains ground truth request state
    - Computes scheduling decisions
    - Tracks KV cache block assignments
  
  Workers (cached copies):
    - Hold last-known request state
    - Apply incremental updates from EngineCore
    - Do NOT need full request list every step
    
  Step protocol:
    EngineCore: compute_diff(new_state, old_state) → send diff via ZMQ
    Worker: apply_diff(diff, cached_state) → run forward pass
    Worker: send output tokens → EngineCore
    EngineCore: update_state(output_tokens) → compute_diff for next step
```

---

## Change 5: Persistent Batch and Efficient Input Preparation

### V0: Full Tensor Reconstruction Every Step

```
V0 input preparation (every step, ~1ms overhead):

  1. Build token_ids tensor from scratch:      [batch × seq] zeros
  2. Build position_ids tensor from scratch:   [batch × seq] zeros
  3. Build attention_mask from scratch:        [batch × seq × seq] zeros
  4. Build block_table from scratch:           [batch × max_blocks] zeros
  5. Populate all tensors from Python dicts:   loop over all requests
  6. Copy to GPU:                              cudaMemcpy × 4 tensors
  
  At 5ms step time: 1ms reconstruction = 20% step overhead
```

### V1: Persistent Batch with Delta Updates

```
V1 Persistent Batch (initialized once, updated incrementally):

  Persistent state (lives across steps):
    token_ids_buf    [max_requests × max_seq_len]  ← GPU tensor, pre-allocated
    position_ids_buf [max_requests × max_seq_len]  ← GPU tensor, pre-allocated
    block_table_buf  [max_requests × max_blocks]   ← GPU tensor, pre-allocated
    
  Per-step update (from EngineCore diff):
    new_tokens: [(req_id, token, position), ...]     ← small list
    new_blocks: [(req_id, block_idx, block_id), ...]  ← small list
    
  Update using NumPy advanced indexing (fast, no Python loops):
    token_ids_buf[req_indices, pos_indices] = new_token_ids
    block_table_buf[req_indices, block_indices] = new_block_ids
    
  One final async cudaMemcpy for changed rows only
  
  At 5ms step time: ~0.1ms update = 2% step overhead (vs 20%)
```

### Batch Metadata Caching

```
Additional persistence:
  - Sampling parameters (temperature, top_p) cached per request
  - Output token arrays grown in-place (no reallocation)
  - Attention metadata (seqlens, cumulative positions) incrementally updated
  
Net result: Input preparation time reduced by ~10× vs V0
            Most relevant at high batch sizes and fast GPU step times
```

---

## Change 6: torch.compile and Piecewise CUDA Graphs

### V0 CUDA Graph Limitations

```
V0 used CUDA graphs for fixed-shape execution:
  - Capture graph at specific (batch_size, seq_len) pairs
  - At runtime: if shape matches → use graph; else fallback to eager
  
  Problem: CUDA graphs require STATIC control flow
           Many models have dynamic shapes (variable-length sequences)
           MoE models have conditional expert routing
           Multimodal models have variable vision token counts
           
  Result: Many operations fell back to eager mode → no graph benefit
```

### V1: torch.compile + Piecewise CUDA Graphs

```
V1 approach:

  torch.compile:
    JIT-compiles model forward pass with Triton-based kernel fusion
    Automatic operator fusion (LayerNorm + Linear → fused kernel)
    Works for diverse/dynamic model architectures
    Applied at model load time → no runtime overhead
    
  Piecewise CUDA Graphs:
    Instead of one CUDA graph for the entire forward pass,
    capture MULTIPLE SMALLER GRAPHS for static subgraphs:
    
    ┌──────────────────────────────────────────────────────┐
    │  Full Forward Pass                                    │
    │                                                       │
    │  [Embedding lookup] ← dynamic (variable tokens)      │
    │         ↓                                            │
    │  ┌─────────────────┐                                 │
    │  │  CUDA Graph 0   │ ← static attention block 0-15  │
    │  └─────────────────┘                                 │
    │         ↓                                            │
    │  [MoE routing] ← dynamic (conditional expert select) │
    │         ↓                                            │
    │  ┌─────────────────┐                                 │
    │  │  CUDA Graph 1   │ ← static expert compute        │
    │  └─────────────────┘                                 │
    │         ↓                                            │
    │  [Sampling] ← dynamic (variable output sizes)        │
    └──────────────────────────────────────────────────────┘
    
    Dynamic portions: run in eager mode (no graph)
    Static portions: captured as CUDA graphs → kernel launch overhead eliminated
    
  Result: Models with dynamic control flow benefit from CUDA graphs
          Previously had to choose between graphs (static) or eager (dynamic)
```

### Compilation Benefits

```
Kernel fusion examples via torch.compile:
  
  V0 (separate kernels):           V1 (fused kernel):
    rmsnorm(x)                       fused_rmsnorm_linear(x, W)
    linear(normed_x, W)              (one GPU kernel, one launch)
    
  Reduces kernel launch overhead from ~100s of µs to ~10s of µs per layer
  Most significant for decode (small batch, latency-dominated)
```

---

## Change 7: FlashAttention 3 Integration

### Mixed-Batch Attention Requirements

V1's unified scheduler produces mixed batches containing both prefill and decode requests in the same step:

```
Mixed batch step example:
  Request A: prefill, 100 tokens (context length: 0→100)
  Request B: decode, 1 token (context length: 300→301)
  Request C: prefill, 75 tokens (context length: 50→125)
  Request D: decode, 1 token (context length: 800→801)
  
  Attention characteristics:
    A: full causal attention over 100 new tokens
    B: single query against 300 cached KV tokens
    C: partial attention (75 new + 50 existing = 125 total)
    D: single query against 800 cached KV tokens
    
  Required: attention kernel that handles VARIABLE sequence lengths
            within a single GPU kernel call
```

### FlashAttention 3 Capabilities

```
FlashAttention 3 (released 2024):
  
  Key improvements over FA2:
    - Asynchronous softmax pipeline: overlaps softmax with matmul
    - Support for variable-length packed sequences (no padding waste)
    - Better H100 tensor core utilization (FP8 support)
    - Persistent kernels: kernel stays resident, avoids re-launch overhead
    - Flash-decode: optimized single-query decode path
    
  Why FA3 enables V1's mixed batches:
    - Variable-length sequence support: A (100), B (1), C (75), D (1) in one call
    - No padding: each token gets exactly the right amount of KV access
    - Paged KV attention: compatible with vLLM's block-based KV cache
    
  Performance vs FA2 (H100):
    Prefill: ~25% faster (better tensor core utilization)
    Decode:  ~15% faster (flash-decode path)
    Mixed:   significant improvement (variable-length support)
```

---

## Change 8: Multiprocessing for Multimodal and Preprocessing

### V0 Multimodal: Synchronous and Blocking

```
V0 multimodal processing:
  
  Image input arrives with request
  ↓
  EngineCore tokenizes text
  EngineCore encodes image (CLIP/SigLIP ViT) ← BLOCKS GPU COMPUTE LOOP
  EngineCore concatenates text + vision tokens
  ↓
  Forward pass
  
  Problem 1: Image encoding (ViT forward) blocks the GPU compute loop
             While ViT runs, the LLM decoder GPU is idle
             
  Problem 2: Multi-turn conversations re-encode same image every turn
             Same image → same encoding → wasted compute
```

### V1 Multimodal: Non-Blocking Preprocessing

```
V1 multimodal architecture:

  ┌────────────────────────┐    ┌──────────────────────────────┐
  │  Preprocessing Process  │    │  EngineCore Process          │
  │  (non-blocking)         │    │                              │
  │                         │    │                              │
  │  Image resize/normalize │    │  GPU compute loop            │
  │  ViT encoder forward    │◄──►│  (never blocks for images)  │
  │  Feature extraction     │    │                              │
  │                         │    │  Encoder cache lookup        │
  │  Preprocessing cache:   │    │  (skip ViT if cached)        │
  │  {image_hash → features}│    │                              │
  └────────────────────────┘    └──────────────────────────────┘

  Encoder cache:
    Cache key: hash(image_bytes)
    Cache value: vision features tensor [num_tokens × hidden]
    Eviction: LRU (configurable max_cached_images)
    
    Multi-turn benefit: 
      Turn 1: encode image (ViT forward) → cache
      Turn 2: cache hit → skip ViT → immediate → save ~100ms
      Turn 3: cache hit → immediate
    
    Chunked prefill with vision:
      Large image produces many vision tokens (e.g., 1344 tokens for high-res)
      EngineCore stores vision features in encoder cache across prefill chunks
      Each chunk accesses cached features → no re-encoding mid-prefill
```

---

## New KV Cache Manager

### V0 KV Cache: PagedAttention Blocks

```
V0 KV Cache Architecture:
  
  Fixed-size blocks: default 16 tokens per block
  Physical blocks: pre-allocated GPU memory pool
  Logical-to-physical mapping: per-request block table
  
  Block metadata: Python dict per block
    → High overhead for large caches
    → Slow eviction (O(n) scan for LRU)
    → Prefix caching disabled by default due to overhead
```

### V1 KV Cache: Redesigned for Zero Overhead

```
V1 KV Cache Architecture:
  
  Same block structure (16 tokens/block default)
  CHANGED: block metadata → compact arrays (no Python dicts)
  
  Block table:
    Numpy array: int32[num_requests × max_blocks_per_request]
    Updated via vectorized indexing (not Python loops)
    
  Free list:
    Circular array of free block IDs
    O(1) allocation: pop from front
    O(1) free: push to back
    
  Prefix cache index:
    Hash map: block_hash(uint64) → physical_block_id(int32)
    O(1) lookup, O(1) insert, O(1) eviction via LRU doubly-linked list
    
  Memory layout improvement (v0.12.0):
    Old: blocks stored with strides matching logical layout
         Effective DMA transfer size: ~32 KB (small → slow DMA)
    New: contiguous physical block layout
         Blocks of same layer stored contiguously in memory
         Effective DMA transfer size: ~2 MB (large → fast DMA)
         
  Impact on KV offloading:
    DMA throughput: 83.4 GB/s (DMA) vs 68.5 GB/s (custom CUDA)
    9× throughput improvement for CPU KV offloading
    4× TTFT reduction when KV must be fetched from CPU
```

### KV Cache Sizing

```
KV cache memory calculation:

  Per-token KV cache size:
    = 2 × num_kv_heads × head_dim × num_layers × dtype_bytes
    
  Llama-3.1-8B (BF16, 32 layers, 8 KV heads, 128 head_dim):
    = 2 × 8 × 128 × 32 × 2 = 131,072 bytes = 128 KB per token
    = 8,192 tokens per GB of KV cache
    
  With 40 GB available on A100 (after 16 GB weights):
    = 40 × 8,192 = 327,680 cached tokens
    = ~1280 simultaneous requests at 256-token average length
```

---

## Performance Results

### Throughput Benchmarks (V1 vs V0)

| Model | Benchmark | V0 | V1 | Improvement |
|---|---|---|---|---|
| Llama-3.1-8B | ShareGPT, H100, 100 QPS | Baseline | +70% | **1.7×** |
| Llama-3.1-70B | ShareGPT, 8× H100, 50 QPS | Baseline | +50% | **1.5×** |
| Qwen2-VL | VisionArena benchmark | Baseline | +80%+ | **>1.8×** |
| Llama-3.1-8B | P50 TTFT at high load | Baseline | -40% | **2.5× lower** |
| Any model | Prefix cache, 100% hit rate | Not default | Default on | **Up to 3× speedup** |

### Latency Breakdown

```
V0 Step Time Breakdown (Llama-3.1-8B, H100, batch=32):
  GPU forward pass:    5.0 ms  (55%)
  Input preparation:  1.5 ms  (17%)
  Tokenization:       1.0 ms  (11%)
  Detokenization:     0.7 ms   (8%)
  Scheduling:         0.5 ms   (5%)
  IPC/overhead:       0.4 ms   (4%)
  Total:              9.1 ms

V1 Step Time Breakdown (same hardware):
  GPU forward pass:    5.0 ms  (87%)  ← same GPU compute
  Input preparation:  0.2 ms   (3%)  ← 7.5× improvement (persistent batch)
  Tokenization:       0.0 ms   (0%)  ← overlapped in API server process
  Detokenization:     0.0 ms   (0%)  ← overlapped in API server process
  Scheduling:         0.3 ms   (5%)  ← slight improvement
  ZMQ IPC:           0.2 ms   (4%)  ← new overhead (but enables overlap)
  Total:              5.7 ms
  
  Improvement: 9.1ms → 5.7ms = 1.6× throughput gain
```

---

## V0 vs V1 Comparison Summary

| Feature | V0 | V1 |
|---|---|---|
| Execution architecture | Single process, serialized | Separate API + EngineCore processes |
| IPC mechanism | Python multiprocessing queue | ZeroMQ (msgpack) |
| Scheduler | Separate prefill/decode logic | Unified token budget |
| Chunked prefill | Flag (`--enable-chunked-prefill`) | **Default on** |
| Prefix caching | Optional (overhead, disabled default) | **Default on (zero overhead)** |
| TP worker topology | Asymmetric (rank 0 = scheduler) | **Symmetric (scheduler separated)** |
| Input preparation | Full tensors rebuilt each step | **Persistent batch + delta updates** |
| CUDA execution | CUDA graphs + eager fallback | **torch.compile + piecewise CUDA graphs** |
| Attention kernel | FlashAttention 2 | **FlashAttention 3** |
| Multimodal encoding | Synchronous, blocking | **Async, non-blocking + encoder cache** |
| KV cache metadata | Python dicts (high overhead) | **Compact arrays (O(1) operations)** |
| PP implementation | Virtual engines (per-stage) | **Centralized BatchScheduler** |
| Status | **Deprecated (v0.11.0)** | **Current production engine** |

---

## Migration from V0 to V1

### Breaking Changes

```
1. VLLM_USE_V1 default:
   V0.6.0-V0.10.x: VLLM_USE_V1=0 (V0 default)
   V0.11.0+:       VLLM_USE_V1=1 (V1 default, V0 removed)

2. Some experimental V0 features not yet ported to V1:
   - Check vLLM changelog for specific feature availability
   - V1 feature parity: >95% of V0 features supported

3. Configuration changes:
   V0: --enable-chunked-prefill  (flag needed)
   V1: chunked prefill always on (no flag needed)
   
   V0: --enable-prefix-caching   (flag needed)
   V1: prefix caching always on  (no flag needed)

4. Python API changes:
   V0: LLMEngine (low-level)
   V1: LLM (recommended), AsyncLLMEngine (async)
   
   Both APIs preserved for compatibility.
```

### Verifying V1 is Active

```bash
# Check engine version in logs
vllm serve meta-llama/Llama-3.1-8B-Instruct --port 8000
# Should see: "INFO Using vLLM V1 engine"

# Python
from vllm import LLM
llm = LLM("meta-llama/Llama-3.1-8B-Instruct")
print(llm.llm_engine.engine_version)  # "v1"

# Force V1 explicitly
VLLM_USE_V1=1 vllm serve ...

# Force V0 (deprecated, not available in v0.11.0+)
VLLM_USE_V1=0 vllm serve ...  # will error in v0.11.0+
```

---

## Known Limitations and Roadmap

### V1 Status (as of 2025–2026)

| Feature | Status |
|---|---|
| Dense model inference (TP, PP) | ✅ Fully supported |
| MoE + Expert Parallelism | ✅ Fully supported |
| Chunked prefill | ✅ Default on |
| Prefix caching | ✅ Default on |
| Multimodal (vision LLMs) | ✅ Supported with async encoder |
| PD Disaggregation (P2P NCCL) | ⚠️ Experimental |
| Speculative decoding | ✅ Supported |
| LoRA serving | ✅ Supported |
| torch.compile integration | ✅ Stable |
| Independent TP for prefill/decode | 🔄 Roadmap |
| Elastic EP (dynamic resize) | 🔄 Active development |
| CPU offload contiguous layout | ✅ v0.12.0 |

### Ongoing Development Areas

```
1. Elastic Expert Parallelism:
   Dynamically resize EP topology without model restart
   Enable recovery from single-GPU failures without full DP group teardown
   
2. Better KV Cache Transfer:
   CPU bridge for PD disaggregation (avoid direct GPU-GPU when topology differs)
   Independent TP sizes in prefill vs. decode instances
   
3. Further Scheduler Optimization:
   Speculative execution (predict token before GPU step completes)
   Better chunked prefill scheduling under SLO constraints
   
4. Quantization Integration:
   NVFP4 support for GB200 (already in v0.11.x for some models)
   Improved int4/int8 accuracy-throughput tradeoffs
```

---

## See Also

- [Tensor Parallelism](tensor-parallelism.md) — TP mechanics and V1 symmetric worker architecture
- [Pipeline Parallelism](pipeline-parallelism.md) — PP with V1 BatchScheduler improvements
- [Distributed LLM Serving](distributed-llm-serving.md) — Ray, NCCL, EP, PD disaggregation

---

*Based on vLLM V1 architecture (v0.11.x+, 2025–2026). Sources: vLLM blog "vLLM V1: A Major Upgrade to vLLM's Core Architecture" (Jan 2025), vLLM v0.11.0 release notes, vLLM GitHub architecture docs, ZeroMQ documentation, FlashAttention 3 paper.*
