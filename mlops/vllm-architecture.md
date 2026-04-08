# vLLM Architecture

vLLM is an open-source LLM inference and serving engine developed at UC Berkeley that achieves state-of-the-art throughput through two core innovations: **PagedAttention** (a paged memory management system for KV caches inspired by OS virtual memory) and **continuous batching** (iteration-level request scheduling). The system has evolved through two major generations — V0 (the original SOSP '23 architecture) and V1 (a ground-up redesign released in January 2025 as alpha, targeting 1.7× higher throughput through architectural simplification and deeper CPU-GPU pipelining).

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                       vLLM Serving Stack                            │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Entry Points                                                 │   │
│  │  ┌──────────┐  ┌──────────────┐  ┌───────────────────────┐  │   │
│  │  │ vllm.LLM │  │AsyncLLMEngine│  │  OpenAI-Compatible    │  │   │
│  │  │(offline) │  │(online, V0)  │  │  FastAPI Server       │  │   │
│  │  └──────────┘  └──────────────┘  └───────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │ generate(prompts)                    │
│                              ▼                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  LLMEngine / EngineCore                                       │   │
│  │                                                               │   │
│  │  ┌──────────────┐  ┌────────────────┐  ┌────────────────┐   │   │
│  │  │  Scheduler   │  │ Model Executor  │  │ Tokenizer /   │   │   │
│  │  │  + BlockSpace│  │ (UniProc/Multi │  │ Detokenizer   │   │   │
│  │  │  Manager     │  │  Proc/Ray)     │  │               │   │   │
│  │  └──────────────┘  └────────────────┘  └────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│              ┌───────────────┼───────────────┐                      │
│              ↓               ↓               ↓                      │
│  ┌─────────────────┐ ┌─────────────┐ ┌─────────────────┐           │
│  │  Worker (GPU 0) │ │Worker GPU 1 │ │ Worker (GPU N)  │           │
│  │ ┌─────────────┐ │ │ [TP rank 1] │ │ [TP rank N-1]  │           │
│  │ │ ModelRunner │ │ │             │ │                 │           │
│  │ │ CUDAGraph   │ │ │  NCCL sync  │ │  NCCL sync      │           │
│  │ │ FlashAttn   │ │ │  AllReduce  │ │  AllReduce      │           │
│  │ │ Sampler     │ │ │             │ │                 │           │
│  │ └─────────────┘ │ │             │ │                 │           │
│  │ ┌─────────────┐ │ │             │ │                 │           │
│  │ │ CacheEngine │ │ │             │ │                 │           │
│  │ │ GPU KV tens │ │ │             │ │                 │           │
│  │ │ CPU swap buf│ │ │             │ │                 │           │
│  │ └─────────────┘ │ │             │ │                 │           │
│  └─────────────────┘ └─────────────┘ └─────────────────┘           │
│                                                                     │
│  GPU Memory Layout (per Worker):                                    │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Model Weights (~65%) │ KV Cache (~30%) │ Activations (~5%) │   │
│  │                       │[Blk0][Blk1]...  │                   │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## Component Breakdown

### LLMEngine (V0)

`LLMEngine` is the central orchestrator in V0. It contains three major components and runs the `step()` loop:

```
LLMEngine responsibilities:
  1. Accept requests via add_request()
  2. Call scheduler.schedule() each step
  3. Call model_executor.execute_model() with scheduler output
  4. Process model outputs (detokenize, check stop conditions)
  5. Stream outputs to clients

Internal components:
  ┌──────────────────────────────────────────────────────────────┐
  │  Scheduler          │ Decides which requests run each step  │
  │  BlockSpaceManager  │ Manages KV block allocation + tables  │
  │  ModelExecutor      │ Coordinates GPU workers               │
  │  Tokenizer          │ Encodes prompts to token IDs          │
  │  Detokenizer        │ Decodes token IDs to text (streaming) │
  └──────────────────────────────────────────────────────────────┘

step() loop (called in tight loop by AsyncLLMEngine):
  seq_group_metadata, scheduler_output = scheduler.schedule()
  output = executor.execute_model(seq_group_metadata,
                                  scheduler_output.blocks_to_swap_in,
                                  scheduler_output.blocks_to_swap_out,
                                  scheduler_output.blocks_to_copy)
  outputs = process_model_outputs(output, scheduler_output)
  return outputs  # → streamed to clients
```

### AsyncLLMEngine (V0)

`AsyncLLMEngine` wraps `LLMEngine` with an async interface for production serving:

```
AsyncLLMEngine:
  - Thin async wrapper around _AsyncLLMEngine
  - Exposes generate(prompt, sampling_params) as async generator
  - Manages request_id → output_queue mapping
  - Runs engine loop in background coroutine (asyncio.Task)
  - Forwards to LLMEngine for actual scheduling/execution

  generate() API:
    async def generate(prompt, sampling_params, request_id):
      # Tokenize prompt
      # Add to LLMEngine queue
      # Yield tokens as they are generated (streaming)
      while not finished:
        await asyncio.sleep(0)  # yield to event loop
        if new_tokens_available(request_id):
          yield new_tokens

  Background loop:
    while True:
      outputs = await asyncio.get_event_loop().run_in_executor(
          None, self.engine.step)  # run step() in thread pool
      for output in outputs:
        self._request_tracker.process_output(output)
```

### EngineCore (V1)

V1 introduces `EngineCore` — a separate, isolated process dedicated exclusively to scheduling and model execution:

```
EngineCore Process (V1):
  ┌──────────────────────────────────────────────────────────────┐
  │  Isolated from API server                                    │
  │  Communicates via ZMQ IPC (EngineCoreRequests messages)     │
  │                                                              │
  │  V1 Scheduler:                                               │
  │    - No prefill/decode distinction in representation         │
  │    - Unified output: {request_id: num_tokens}               │
  │    - Hash-based prefix cache (default on)                   │
  │    - Constant-time LRU eviction (doubly-linked list)        │
  │    - Only 2 queues: waiting + running                       │
  │                                                              │
  │  Model Executor (V1):                                        │
  │    - UniProcExecutor or MultiProcExecutor                   │
  │    - Workers cache request state → only diffs transmitted   │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘

AsyncLLM (V1 entry point):
  ┌──────────────────────────────────────────────────────────────┐
  │  FastAPI ↔ AsyncLLM                                          │
  │    ├── Tokenization process (separate)                       │
  │    ├── Detokenization process (separate)                     │
  │    └── Multimodal preprocessing process (separate)          │
  │  ZMQ IPC → EngineCore process                               │
  └──────────────────────────────────────────────────────────────┘
```

## Scheduler

The Scheduler is the brain of vLLM's continuous batching. It maintains queues of requests and decides what to execute each step.

### Request State Machine

```
Request Lifecycle:
                         ┌──────────────────────────────┐
New Request ────────────→│      WAITING queue            │
                         │  FCFS (First-Come-First-Served)│
                         │  [req_E, req_D, req_C, req_B] │
                         └──────────────┬───────────────┘
                                        │ schedule() admits req when
                                        │ KV blocks available + budget OK
                                        ▼
                         ┌──────────────────────────────┐
                         │      RUNNING queue            │
                         │  [req_A (decode), req_B (pfll)]│
                         └──────────────┬───────────────┘
                              ┌──────────┤ memory pressure:
                              │          │ preempt most-recently-started
                              ▼          │
              ┌───────────────────────┐  │
  (V0 only)   │   SWAPPED queue       │  │  request completes
              │ [req_X (CPU blocks)]  │  │  → free blocks
              └───────────┬───────────┘  │
                          │ GPU free     │
                          └── swap back ─┘

V1 change: SWAPPED queue removed. Default preemption is recompute.
           Request evicted → returned to WAITING queue → re-prefills.
```

### Scheduling Algorithm

```python
# Scheduling logic (conceptual, V0 chunked-prefill mode):

def schedule() -> SchedulerOutput:
    output = SchedulerOutput()
    budget = SchedulingBudget(
        max_num_seqs=256,
        max_num_batched_tokens=8192
    )
    
    # Step 1: Schedule decode requests (highest priority)
    for req in running_queue:
        if req.is_decoding:
            can_append = block_manager.can_append_slot(req)
            if can_append == NEED_COW:
                output.blocks_to_copy.append(can_append.block_pair)
            elif can_append == NEED_NEW_BLOCK:
                block_manager.append_slot(req)
            budget.subtract(num_seqs=1, num_tokens=1)
    
    # Step 2: Schedule chunked prefills
    for req in running_queue:
        if req.is_prefilling:
            chunk = min(req.remaining_tokens, budget.remaining_tokens)
            block_manager.allocate_slots(req, chunk)
            budget.subtract(num_seqs=1, num_tokens=chunk)
    
    # Step 3: Swap in preempted requests
    for req in swapped_queue:
        if budget.remaining and block_manager.can_swap_in(req):
            blocks = block_manager.swap_in(req)
            output.blocks_to_swap_in.extend(blocks)
            budget.subtract(num_seqs=1, num_tokens=1)
    
    # Step 4: Admit new requests from waiting queue
    while waiting_queue and budget.has_capacity:
        req = waiting_queue.peek()
        if block_manager.can_allocate(req):
            waiting_queue.pop()
            block_manager.allocate(req)
            running_queue.add(req)
            budget.subtract(...)
        else:
            # Insufficient memory: may preempt running request
            if should_preempt(req):
                victim = select_preemption_victim()
                preempt(victim, output)
    
    return output

# Preemption victim selection:
def select_preemption_victim():
    # LIFO: most recently added to running queue
    return running_queue.last()
    # For multi-seq groups: return entire group (gang eviction)
```

### Scheduling Budget Constraints

```
Budget limits per step:
  max_num_seqs = 256
    → Bounds GPU memory for input metadata tensors
    → Bounds CUDA kernel complexity

  max_num_batched_tokens = 8192
    → Bounds step duration (latency SLO enforcement)
    → Shared between prefill chunks and decode tokens

  watermark = max_num_seqs × 1 (free blocks reserved)
    → Prevents over-admission that immediately triggers preemption
    → Running decode requests need 1 block/block_size steps

Admission check:
  new_request admitted iff:
    current_seqs + 1 ≤ max_num_seqs
    AND current_tokens + request_chunk_tokens ≤ max_num_batched_tokens
    AND free_blocks - needed_blocks ≥ watermark
```

## Worker Architecture

Each `Worker` represents one GPU (or one TP rank in tensor-parallel setups):

```
Worker Components:
  ┌──────────────────────────────────────────────────────────────┐
  │  GPUModelRunner                                              │
  │  ├── InputBatch (V1) / tensor builder (V0)                  │
  │  │     Pre-allocated NumPy arrays, updated incrementally     │
  │  │     (input_ids, positions, slot_mapping, block_tables)    │
  │  ├── CUDAGraphRunner                                         │
  │  │     Captured CUDA graphs for decode-only steps           │
  │  │     Batch sizes: [1, 2, 4, 8, 16, 32, ...] captured      │
  │  ├── Attention Backend (FlashAttention 2/3)                  │
  │  │     PagedAttention v1 or v2 kernel selection              │
  │  ├── Model (e.g., LlamaForCausalLM)                         │
  │  │     Transformer layers with paged KV cache               │
  │  └── Sampler                                                 │
  │        Temperature, top-p, top-k, repetition penalty        │
  │                                                              │
  │  CacheEngine                                                 │
  │  ├── gpu_cache: List[(key_tensor, val_tensor)] × n_layers   │
  │  │     Pre-allocated fixed tensors (never reallocated)       │
  │  ├── cpu_cache: same structure in pinned CPU RAM (swap)     │
  │  └── Operations: swap_in, swap_out, copy (CoW)              │
  └──────────────────────────────────────────────────────────────┘
```

### Worker Initialization Sequence

```
Worker startup sequence (called once during system init):

1. init_device():
   ├── torch.cuda.set_device(local_rank)
   ├── Check gpu_memory_utilization (default 0.9 = use 90% of VRAM)
   ├── dist.init_process_group(backend='nccl')
   │    → Establishes NCCL communicators for AllReduce
   ├── Set TP/PP/DP rank and process groups
   └── Instantiate GPUModelRunner

2. load_model():
   ├── Instantiate model architecture (e.g., LlamaForCausalLM)
   ├── Load weights from HuggingFace hub or local checkpoint
   ├── Apply quantization if configured (AWQ, GPTQ, FP8, INT4)
   ├── model.eval()  # disable dropout, training-only layers
   └── Optional: torch.compile(model) for kernel fusion

3. initialize_kv_cache():
   a. Get per-layer KV cache spec:
      (n_kv_heads, head_dim, kv_dtype) for each layer
      (Hybrid models like MoE/SSM may have different specs per layer)
   
   b. Profile GPU memory via dummy forward pass:
      ├── Run forward pass with max_num_seqs × max_num_batched_tokens
      ├── torch.cuda.memory_reserved() → peak GPU memory
      └── available_kv_memory = gpu_total × gpu_memory_utilization
                               - model_memory - activation_peak

   c. Compute num_blocks:
      block_bytes = 2 × block_size × n_kv_heads × head_dim × dtype_bytes
      num_gpu_blocks = available_kv_memory // block_bytes
      num_cpu_blocks = cpu_swap_space_bytes // block_bytes
   
   d. Allocate KV cache tensors (ONCE, never reallocated):
      key_cache: Tensor[num_gpu_blocks, n_kv_heads, head_dim//x, block_size, x]
      val_cache: Tensor[num_gpu_blocks, n_kv_heads, head_dim, block_size]
      (Transposed layout for optimal GPU memory access patterns)
   
   e. Bind KV tensors to attention modules

4. CUDA Graph capture (unless --enforce-eager):
   For batch_size in [1, 2, 4, 8, 16, 32, 64, 128, 256]:
     ├── Run warmup decode forward pass (dummy inputs)
     ├── torch.cuda.CUDAGraph.capture_begin()
     ├── Run decode forward pass (shape must be static for graph)
     ├── torch.cuda.CUDAGraph.capture_end()
     └── Store graph + input/output tensor references
   
   At decode time: select graph with smallest batch_size ≥ actual
   → Replay pre-captured kernel sequence (no dynamic kernel launch overhead)
   → Prefill NOT captured (shapes vary with prompt length)
```

## ModelRunner: Input Preparation

The `ModelRunner` (or `GPUModelRunner` in V1) prepares the input tensors for each forward pass:

### V0 Input Preparation (Rebuild Each Step)

```python
def prepare_model_input(seq_group_metadata_list):
    input_tokens = []
    input_positions = []
    slot_mapping = []      # physical KV slot for each token
    block_tables = []      # [num_seqs, max_blocks] 2D array
    seq_lens = []
    
    for seq_group in seq_group_metadata_list:
        for seq_id, seq_data in seq_group.seq_data.items():
            # DECODE: contribute 1 token (last generated)
            if seq_group.is_prompt == False:
                token = seq_data.get_last_token_id()
                pos   = seq_data.get_len() - 1
                tokens_to_add = [token]
                positions_to_add = [pos]
            
            # PREFILL: contribute all prompt tokens (or next chunk)
            else:
                tokens_to_add = seq_data.get_unprocessed_tokens()
                positions_to_add = range(len(tokens_to_add))
            
            for pos in positions_to_add:
                # Compute physical slot
                logical_block  = pos // block_size
                within_block   = pos % block_size
                physical_block = seq_group.block_tables[seq_id][logical_block]
                slot = physical_block * block_size + within_block
                slot_mapping.append(slot)
            
            input_tokens.extend(tokens_to_add)
            input_positions.extend(positions_to_add)
            block_tables.append(seq_group.block_tables[seq_id])
            seq_lens.append(seq_data.get_len())
    
    return ModelInputForGPU(
        input_ids     = torch.tensor(input_tokens,    device='cuda'),
        positions     = torch.tensor(input_positions, device='cuda'),
        slot_mapping  = torch.tensor(slot_mapping,    device='cuda'),
        block_tables  = torch.tensor(block_tables,    device='cuda'),
        seq_lens      = torch.tensor(seq_lens,        device='cuda'),
        attn_metadata = build_flash_attention_metadata(...),
    )
```

### V1 Persistent Batch (Incremental Update)

```python
class InputBatch:
    """Pre-allocated arrays updated incrementally — no rebuild per step."""
    
    def __init__(self, max_num_reqs, max_model_len, block_size):
        # Pre-allocated on CPU (avoid malloc per step)
        self.input_ids   = np.empty([max_num_reqs], dtype=np.int32)
        self.positions   = np.empty([max_num_reqs], dtype=np.int64)
        self.slot_mapping = np.empty([max_num_reqs], dtype=np.int64)
        self.block_table  = np.empty(
            [max_num_reqs, max_model_len // block_size], dtype=np.int32)
        self.req_ids = {}  # request_id → row index
    
    def add_request(self, req_id, prompt_tokens, block_table):
        idx = self._next_free_row()
        self.req_ids[req_id] = idx
        self.input_ids[idx]     = prompt_tokens[-1]
        self.positions[idx]     = len(prompt_tokens) - 1
        self.block_table[idx]   = block_table
        # slot_mapping computed from block_table
    
    def remove_request(self, req_id):
        idx = self.req_ids.pop(req_id)
        # Compact by moving last element to idx (O(1))
    
    def update_decode_step(self, req_ids_to_update, new_tokens, new_slots):
        # In-place NumPy ops: update positions += 1, input_ids = new_token
        idxs = [self.req_ids[r] for r in req_ids_to_update]
        self.input_ids[idxs]  = new_tokens   # vectorized NumPy assignment
        self.positions[idxs] += 1             # in-place increment
        self.slot_mapping[idxs] = new_slots
    
    def to_gpu(self, active_count):
        # Single bulk CPU→GPU copy of only the active portion
        return {
            'input_ids':   torch.as_tensor(self.input_ids[:active_count]).cuda(),
            'positions':   torch.as_tensor(self.positions[:active_count]).cuda(),
            'block_table': torch.as_tensor(self.block_table[:active_count]).cuda(),
        }
```

## Forward Pass: The Continuous Batch Execution

Once inputs are prepared, the model forward pass processes the concatenated super-sequence:

```
Forward Pass Execution (per transformer layer):

Input:
  input_ids:    [A_tok, B_tok0, B_tok1, ..., B_tok49, C_tok]  # 52 tokens
  positions:    [100,   0,      1,      ..., 49,       30   ]
  slot_mapping: [phys_slot for each of 52 tokens]
  block_tables: [A_table, B_table, C_table]  # for A: KV lookup, B: write, C: lookup
  seq_lens:     [101, 50, 31]

Per-layer execution:
  ┌──────────────────────────────────────────────────────────────┐
  │  1. Input Projection                                         │
  │     q, k, v = linear(input_hidden_states)  # batch matmul   │
  │     Shape: [52, n_heads × head_dim] each                    │
  │                                                              │
  │  2. KV Cache Write (fused kernel 1)                          │
  │     reshape_and_cache(k, v, key_cache, val_cache,           │
  │                        slot_mapping)                         │
  │     → Writes k[i], v[i] to kv_cache[slot_mapping[i]]       │
  │     → Scattered write to non-contiguous physical slots      │
  │                                                              │
  │  3. Paged Attention (fused kernel 2)                        │
  │     output = paged_attention(q, key_cache, val_cache,       │
  │                               block_tables, seq_lens,        │
  │                               context_lens, ...)            │
  │     → Computes attention for each token using its block table│
  │     → Decode tokens: attends over full KV history in blocks │
  │     → Prefill tokens: standard causal attention + block write│
  │     → Cross-sequence masking enforced by attn_metadata      │
  │                                                              │
  │  4. Output Projection + FFN                                  │
  │     Standard transformer operations on output tensor        │
  │     For TP: NCCL AllReduce after attention + FFN            │
  └──────────────────────────────────────────────────────────────┘

Output:
  hidden_states: [52, d_model]
  → After final layer, extract last-token hidden states per sequence
  → Pass to sampler to produce next tokens
```

### CUDA Graph Optimization for Decode

```
CUDA Graph for decode-only steps:

Why capture decode but not prefill?
  Decode: fixed tensor shapes each step
    [batch_size, 1] for decode tokens
    [batch_size, max_blocks] for block tables
    → Can capture a static computational graph
  
  Prefill: variable shapes (different prompt lengths)
    → Cannot be captured as a static graph

CUDA Graph mechanics:
  Capture time (startup):
    # Run with batch_size=32, shapes are static:
    g = torch.cuda.CUDAGraph()
    with torch.cuda.graph(g):
        output = model(static_input_ids, static_positions, ...)
    # GPU kernel sequence recorded as DAG
  
  Replay time (each decode step with batch=32):
    # Update static input tensors in-place (no re-capture needed)
    static_input_ids[:32]    = new_token_ids
    static_positions[:32]    = new_positions
    static_slot_mapping[:32] = new_slots
    g.replay()  # execute pre-recorded kernel sequence
    # No Python interpreter involved during replay!
    # No kernel launch overhead per individual kernel
    result = static_output_tensor.clone()

Performance impact:
  Without CUDA graphs:
    Each forward pass: hundreds of individual CUDA kernel launches
    Each launch: ~5–50µs overhead (kernel launch latency)
    100 kernels × 20µs = 2ms overhead per step

  With CUDA graphs (decode):
    One graph replay = one "macro kernel launch"
    Overhead: ~0.1ms for graph replay vs. 2ms for individual launches
    → ~20× reduction in per-step overhead at small batch sizes
    → Most impactful for single-sequence or small-batch decode
```

## Distributed Execution

### Tensor Parallelism

vLLM uses Megatron-LM style tensor parallelism to split model computation across multiple GPUs:

```
Tensor Parallel (TP=4) Architecture:

                ┌──────────────────────────┐
                │  EngineCore / Scheduler  │
                │  (CPU, single process)    │
                └──────────────┬───────────┘
                               │ broadcast SchedulerOutput
               ┌───────────────┼───────────────────────────┐
               ↓               ↓               ↓           ↓
         Worker (GPU0)   Worker (GPU1)   Worker (GPU2)   Worker (GPU3)
         TP rank 0       TP rank 1       TP rank 2       TP rank 3

         Each worker holds:
         ├── Q projection shard (n_heads/4 heads)
         ├── K, V projection shard (n_kv_heads/4 heads)
         ├── FFN shard (d_ffn/4 neurons each)
         └── KV cache for ITS OWN attention head shards

         After each attention layer: AllReduce (NCCL)
         After each FFN layer:       AllReduce (NCCL)

KV Cache in TP:
  Each GPU stores K,V only for its own head shard
  No KV cache communication within a step
  → KV cache effectively partitioned across GPUs
  → Each GPU needs n_kv_heads/TP × head_dim × block_size bytes per block

Pipeline Parallelism (PP):
  Layers split across GPUs (GPU0: layers 0-15, GPU1: layers 16-31)
  Different GPUs process different layers (pipelined micro-batches)
  Less commonly used in vLLM (TP is primary parallelism strategy)
```

### Multi-Process Worker Communication

```
V0 Worker Communication:
  SchedulerOutput → pickled Python object → sent to each worker via:
    - UniProcExecutor: direct Python call (same process)
    - MultiProcExecutor: Python multiprocessing IPC
    - RayDistributedExecutor: Ray's object store

V1 Worker Communication:
  SchedulerOutput → compact binary message → sent via:
    ZMQ push/pull socket over shared memory
    rpc_broadcast_mq: shared memory ring buffer (zero-copy)
    worker_response_mq: IPC socket for return values
  
  V1 improvement:
    Workers cache request state across steps
    Only DIFFS (newly added/removed requests, new tokens) transmitted
    Not full metadata for all active requests each step
    → Lower per-step IPC overhead, especially at large batch sizes

V0 vs V1 worker asymmetry:
  V0: Scheduler colocated with Worker 0 (driver worker)
      → Worker 0 does more work (scheduling + model execution)
      → Asymmetric complexity, harder to debug
  
  V1: Scheduler in separate EngineCore process
      → All workers symmetric (same model execution work)
      → Cleaner separation of concerns
      → Easier to scale independently
```

## End-to-End Request Flow

```
Complete request lifecycle through vLLM:

1. REQUEST ARRIVAL
   Client: POST /v1/chat/completions HTTP/1.1
   FastAPI handler:
     → tokenize prompt: [tok_0, tok_1, ..., tok_{N-1}]
     → create EngineCoreRequest{
         request_id,
         prompt_token_ids,
         sampling_params: {temperature, top_p, max_tokens, ...},
         arrival_time,
         priority
       }
     → enqueue in scheduler.waiting_queue (FCFS: append to tail)

2. SCHEDULING (each step)
   scheduler.schedule():
     a. Try to schedule running decode requests (1 token each)
     b. Continue any in-progress chunked prefills
     c. Try to swap in preempted requests
     d. Admit new requests from waiting_queue if budget allows
     e. Return SchedulerOutput:
          {
            scheduled_seq_groups: [(seq_metadata, is_prompt), ...]
            blocks_to_swap_in:  [(cpu_blk, gpu_blk), ...]   # CPU→GPU
            blocks_to_swap_out: [(gpu_blk, cpu_blk), ...]   # GPU→CPU
            blocks_to_copy:     [(src_blk, dst_blk), ...]   # CoW copies
          }

3. KV CACHE MEMORY OPERATIONS (before forward pass)
   CacheEngine.swap_in(blocks_to_swap_in)   # CPU RAM → GPU VRAM
   CacheEngine.swap_out(blocks_to_swap_out)  # GPU VRAM → CPU RAM
   CacheEngine.copy(blocks_to_copy)          # CoW: GPU block → GPU block
   (These are PCIe/NVLink transfers + CUDA memcpy operations)

4. INPUT PREPARATION (in ModelRunner)
   Build input tensors (V0: rebuild; V1: incremental update):
     input_ids:    concatenation of all scheduled tokens
     positions:    absolute token positions
     slot_mapping: physical KV slot per token (for cache write)
     block_tables: 2D array [num_seqs × max_blocks] (for cache read)
     seq_lens:     total context length per sequence
     attn_metadata: FlashAttention-specific metadata

5. FORWARD PASS (one GPU step, all sequences in parallel)
   For each transformer layer:
     a. Compute Q, K, V projections (fused linear)
     b. Write new K, V to KV cache via slot_mapping (kernel 1)
     c. Run PagedAttention kernel (v1 or v2) using block_tables (kernel 2)
     d. Compute FFN output
     e. AllReduce across TP workers (if TP > 1)
   
   Output: hidden_states[num_tokens, d_model]

6. SAMPLING
   Extract last-token hidden state per sequence:
     hidden[A] = hidden_states[0]    (Req A: 1 decode token → index 0)
     hidden[B] = hidden_states[50]   (Req B: 50 prefill tokens → last index)
     hidden[C] = hidden_states[51]   (Req C: 1 decode token → index 51)
   
   Apply lm_head (unembedding): [d_model] → [vocab_size]
   Apply sampling:
     temperature scaling: logits /= temperature
     top-k filtering:     set bottom-k logits to -inf
     top-p (nucleus):     filter cumulative probability
     multinomial sample:  draw one token
   Check stop conditions: EOS token, max_tokens, stop strings

7. POST-PROCESSING
   Append sampled tokens to sequence state
   Update block_manager with new token positions
   For finished sequences:
     → free KV blocks (ref_count decrement)
     → send final output to client with finish_reason
   For continuing sequences:
     → stream new token text to client (detokenize)
     → sequence remains in running queue

8. REPEAT from step 2 (no barrier: immediately schedule next step)
   The continuous batching loop: no waiting between steps
```

## V0 vs. V1 Architecture Comparison

```
┌──────────────────────┬────────────────────────┬─────────────────────────┐
│ Feature              │ V0                     │ V1                      │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Scheduler            │ Separate prefill/decode│ Unified {req: n_tokens} │
│ representation       │ state tracking         │ no phase distinction     │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Scheduler queues     │ 3: waiting/running/    │ 2: waiting/running      │
│                      │ swapped                │                         │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Chunked prefill      │ Optional, complex      │ Default, seamless       │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Prefix caching       │ Optional, O(n) evict   │ Default, O(1) LRU      │
│                      │ (20% overhead @ 0% hit)│ (<1% overhead)         │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Worker architecture  │ Scheduler + Worker0    │ Scheduler isolated in  │
│                      │ colocated (asymmetric) │ own process (symmetric) │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Input preparation    │ Rebuild tensors from   │ Persistent InputBatch   │
│                      │ scratch every step     │ (incremental NumPy ops) │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ CPU-GPU overlap      │ Limited                │ Tokenize/detokenize/    │
│                      │                        │ multimodal in parallel  │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Preemption default   │ Swap (multi-seq)       │ Recompute (default)     │
│                      │ Recompute (single-seq) │ Swap (multi-seq only)   │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ IPC mechanism        │ Python pickle / Ray    │ ZMQ + shared memory     │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Attention backend    │ FlashAttention 2       │ FlashAttention 3        │
│                      │                        │ (better mixed batches)  │
├──────────────────────┼────────────────────────┼─────────────────────────┤
│ Throughput vs V0     │ Baseline               │ Up to 1.7× higher      │
└──────────────────────┴────────────────────────┴─────────────────────────┘
```

## Key Design Decisions

```
┌──────────────────────────────┬──────────────────────────────────────────┐
│ Design Decision              │ Rationale                                │
├──────────────────────────────┼──────────────────────────────────────────┤
│ Block size = 16 tokens       │ 1 CUDA warp/block; <1.5% fragmentation  │
├──────────────────────────────┼──────────────────────────────────────────┤
│ FCFS scheduling              │ Simple, prevents starvation              │
├──────────────────────────────┼──────────────────────────────────────────┤
│ LIFO eviction (preemption)   │ Evict newest → least invested compute    │
├──────────────────────────────┼──────────────────────────────────────────┤
│ Gang eviction for multi-seq  │ Partial eviction breaks beam search      │
├──────────────────────────────┼──────────────────────────────────────────┤
│ Recompute > Swap (V1)        │ PCIe bandwidth too costly; recompute     │
│                              │ overhead <20% of swap overhead           │
├──────────────────────────────┼──────────────────────────────────────────┤
│ Hash-based prefix cache      │ O(1) lookup; LRU eviction natural       │
├──────────────────────────────┼──────────────────────────────────────────┤
│ No padding (super-sequence)  │ Maximum GPU utilization (100% real tok) │
├──────────────────────────────┼──────────────────────────────────────────┤
│ CUDA graphs for decode       │ Eliminates kernel launch overhead        │
├──────────────────────────────┼──────────────────────────────────────────┤
│ ZMQ IPC (V1)                 │ Low-latency, zero-copy IPC              │
├──────────────────────────────┼──────────────────────────────────────────┤
│ Persistent Batch (V1)        │ Eliminates CPU tensor rebuild per step  │
└──────────────────────────────┴──────────────────────────────────────────┘
```

## Performance Numbers

```
Memory Efficiency (OPT-13B, SOSP '23):
  FasterTransformer: 20.4% KV cache utilization
  HuggingFace TGI:   38.2% KV cache utilization
  vLLM:              96.3% KV cache utilization

Throughput (OPT-13B, A100-40GB, ShareGPT dataset):
  vLLM vs HuggingFace TGI:     22× higher throughput
  vLLM vs FasterTransformer:   20× higher throughput
  vLLM vs Orca (oracle):        2.7× higher throughput

Kernel overhead:
  PagedAttention vs. vanilla FlashAttention: +20–26% kernel latency
  But system-level throughput: +2×–22× (memory efficiency wins)

V1 improvements over V0:
  Up to 1.7× throughput improvement
  Primarily from: persistent batch, CPU-GPU overlap, O(1) prefix cache
```

---
*Related: [[PagedAttention]], [[KV Cache Management]], [[Continuous Batching]], [[LLM Inference Serving]]*
