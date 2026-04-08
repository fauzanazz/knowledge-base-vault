# Continuous Batching

Continuous batching (also called iteration-level scheduling or in-flight batching) is a scheduling strategy for LLM inference servers that makes scheduling decisions at every single decode step rather than at batch boundaries. This eliminates idle GPU time caused by waiting for slow requests to finish before admitting new ones, achieving 2×–22× higher throughput compared to static batching on real-world workloads.

## The Static Batching Problem

Traditional deep learning inference uses **static batching**: a fixed set of requests is grouped together, processed until all requests complete, and only then does the next batch begin. For image classifiers and encoder models this is fine because all inputs have the same computational cost. For LLMs, it is catastrophically inefficient.

### Why LLM Outputs Have Variable Length

Unlike classification tasks where every input produces one output, LLM generation produces a variable number of tokens per request:

```
Request A: "Summarize this article in one sentence."
  → Output: "The article describes new renewable energy policies." (8 tokens)

Request B: "Write a 500-word essay on climate change."
  → Output: [500+ tokens over 500+ decode steps]

Request C: "What is 2+2?"
  → Output: "4" (1 token)
```

Output length is **unknown at request admission time** and follows a highly skewed distribution in production (most responses are short; a few are very long).

### Static Batching Failure Mode

```
Static Batch Example:
  Batch = {Req A (1 tok output), Req B (200 tok output), Req C (50 tok output)}

  ┌──────────────────────────────────────────────────────────────────────┐
  │ Step  1: [Req A decode] [Req B decode] [Req C decode]  ← all active  │
  │ Step  2: [Req A DONE  ] [Req B decode] [Req C decode]                │
  │          [  PADDING   ] [Req B decode] [Req C decode]  ← Req A done! │
  │ Step  3: [  PADDING   ] [Req B decode] [Req C decode]  ← wasted slot │
  │ ...                                                                   │
  │ Step 50: [  PADDING   ] [Req B decode] [Req C DONE  ]                │
  │          [  PADDING   ] [Req B decode] [  PADDING   ]                │
  │ Step 51: [  PADDING   ] [Req B decode] [  PADDING   ]                │
  │ ...                                                                   │
  │ Step 200:[  PADDING   ] [Req B DONE  ] [  PADDING   ]                │
  └──────────────────────────────────────────────────────────────────────┘

  Total steps: 200 (determined by slowest request)
  Effective computation: 1 + 50 + 200 = 251 token-steps
  Wasted computation:    (200-1) + (200-50) = 199 + 150 = 349 padding-steps
  GPU utilization:       251 / (251+349) = 41.8%

  New request D arrives at step 2:
  → Must wait 198 steps (until step 200) before being admitted
  → Queuing latency: 198 × step_time (unacceptable!)
```

The fundamental issue: **batch completion time is dictated by the longest request**, and all other request slots sit idle once their request finishes.

## Continuous Batching: Iteration-Level Scheduling

Continuous batching solves this by making scheduling decisions after **every single forward pass**:

```
Continuous Batching with Iteration-Level Scheduling:

  ┌──────────────────────────────────────────────────────────────────────┐
  │ Step  1: [Req A decode] [Req B decode] [Req C decode]                │
  │ Step  2: [Req A DONE→D] [Req B decode] [Req C decode]                │
  │          └──┬──┘                                                     │
  │             └→ Req A finishes! Req D admitted immediately            │
  │ Step  3: [Req D PREFILL] [Req B decode] [Req C decode]               │
  │ Step  4: [Req D decode ] [Req B decode] [Req C decode]               │
  │ ...                                                                   │
  │ Step 50: [Req D decode ] [Req B decode] [Req C DONE→E]               │
  │ Step 51: [Req D decode ] [Req B decode] [Req E PREFILL]              │
  │ Step 52: [Req D decode ] [Req B decode] [Req E decode ]              │
  └──────────────────────────────────────────────────────────────────────┘

  Key properties:
    ✓ No padding — every slot in every step processes real tokens
    ✓ Requests admitted the moment any slot becomes free
    ✓ GPU utilization approaches 100% for steady-state workloads
    ✓ Short requests do not block behind long ones
    ✓ Long requests never evict short requests from the batch
```

The term "iteration-level" refers to making scheduling decisions at each **iteration** of the decode loop (each forward pass), rather than at batch (multiple iterations) boundaries.

## The step() Loop

Continuous batching is implemented as a tight loop where each iteration:
1. Schedules what to run
2. Executes the forward pass
3. Processes outputs
4. Immediately loops back to scheduling

### V0 step() Implementation

```python
def step(self) -> List[RequestOutput]:
    """Single iteration of the engine loop — called in a tight loop."""
    
    # 1. SCHEDULING DECISION
    # Determines which requests to run this iteration
    seq_group_metadata_list, scheduler_outputs = self.scheduler.schedule()
    
    if scheduler_outputs.is_empty():
        # No requests to run this step (system idle or all waiting for memory)
        return []
    
    # 2. MEMORY OPERATIONS (before forward pass)
    # Execute any KV block swaps or copies determined by scheduler
    # These happen BEFORE the forward pass so KV data is in place
    # (swap_in: CPU→GPU, swap_out: GPU→CPU, copy: GPU→GPU CoW)
    
    # 3. FORWARD PASS
    output = self.model_executor.execute_model(
        seq_group_metadata_list=seq_group_metadata_list,
        blocks_to_swap_in=scheduler_outputs.blocks_to_swap_in,
        blocks_to_swap_out=scheduler_outputs.blocks_to_swap_out,
        blocks_to_copy=scheduler_outputs.blocks_to_copy,
    )
    
    # 4. OUTPUT PROCESSING
    request_outputs = self._process_model_outputs(
        output=output,
        scheduled_seq_groups=scheduler_outputs.scheduled_seq_groups,
        ignored_seq_groups=scheduler_outputs.ignored_seq_groups,
    )
    
    # request_outputs contains newly generated tokens for each request
    # Finished requests are included with is_finished=True
    return request_outputs
    # → immediately loop back to step() again
```

The critical property: step() has **no concept of a batch lifecycle**. Each call is independent — the scheduler freely adds/removes requests each iteration.

### Async Engine Loop (Production)

In production, `AsyncLLMEngine` runs the step loop in a background coroutine, allowing the FastAPI server to handle new HTTP requests concurrently with generation:

```
AsyncLLMEngine architecture:
  
  HTTP Server (asyncio event loop):
    POST /v1/chat/completions
      → tokenize → enqueue → return async generator to client
  
  Background coroutine (same event loop):
    while True:
      await asyncio.sleep(0)  # yield to HTTP handlers
      outputs = await self._run_engine_step()
      for output in outputs:
        if output.finished:
          send final token to client
        else:
          send partial output (streaming)

  Concurrency model:
    HTTP handlers add requests to request_queue
    Engine loop drains request_queue every step
    Streaming responses are sent as tokens are generated
    Single-process, single-event-loop (no threading)
```

## How Continuous Batching Handles Mixed Request Types

The key technical challenge: in one step, the batch may contain requests at very different stages — some in prefill, some in early decode, some near completion. All must be executed in a single forward pass.

### Concatenated Super-Sequence

vLLM concatenates all active requests' tokens into one long input tensor per step, with no padding between sequences:

```
Step state:
  Req A: decoding (context=100 tokens) → contributes 1 token (last generated)
  Req B: prefilling (prompt=50 tokens) → contributes 50 tokens (all prompt tokens)
  Req C: decoding (context=30 tokens)  → contributes 1 token (last generated)

input_ids tensor (concatenated, NO PADDING):
  [A_tok] [B_tok_0, B_tok_1, ..., B_tok_49] [C_tok]
   1 tok           50 tokens                  1 tok
   ─────────────── total: 52 tokens ───────────────

Supporting metadata:
  seq_start_locs = [0, 1, 51]        # where each seq starts in input_ids
  seq_lens       = [101, 50, 31]     # total context length including KV cache
  slot_mapping   = [physical_slots for each of 52 tokens]
  block_tables   = [block table arrays for A, B, C]

  Attention computation:
    Token A_tok:    attends to ALL 100 previous A tokens (via block table)
    Token B_tok_0:  attends to nothing (first token of prefill)
    Token B_tok_k:  attends to B_tok_0..B_tok_{k-1} only (causal within B)
    Token C_tok:    attends to ALL 29 previous C tokens (via C's block table)
    
    Cross-sequence attention: MASKED OUT by attention metadata
    (Req A tokens cannot attend to Req B or C tokens — they are distinct
    requests sharing a GPU batch, not a single conversation)
```

This concatenated representation gives vLLM's batching its name: **no right-padding** means no wasted GPU computation. Every position in the input tensor contributes to the output.

### Attention Mask for Mixed Batches

```
Attention mask for the concatenated [A, B, C] batch:
(1 = can attend, 0 = masked)

                   [A]  [B0 B1 B2...B49]  [C]
  A_tok attends:   [1 ]  [0  0  0...  0]  [0]  ← only to own KV cache
  B_tok_0 attends: [0 ]  [1  0  0...  0]  [0]  ← first B token, empty context
  B_tok_1 attends: [0 ]  [1  1  0...  0]  [0]  ← causal within B only
  ...
  B_tok_k attends: [0 ]  [1  1  1...  1  0...0]  [0]
  C_tok attends:   [0 ]  [0  0  0...  0]  [1]  ← only to own KV cache

For decode tokens (A_tok, C_tok):
  They attend to their KV-CACHED history (not visible in input_ids)
  This is implemented via block_tables: kernel looks up cached K,V
  The current token's K,V is written to KV cache via slot_mapping
  
For prefill tokens (B_tok_0..49):
  Standard causal mask within the sequence
  New K,V written to newly-allocated KV blocks
```

## Prefill vs. Decode Phase Mixing

Modern continuous batching systems mix prefill and decode requests in the same step. This creates a fundamental scheduling tension:

### The Interference Problem

```
Prefill characteristics:
  - Processes N tokens → O(N²) attention → compute-bound
  - High GPU SM utilization
  - Fills the GPU compute pipeline

Decode characteristics:
  - Processes 1 token per sequence → O(T) memory reads → memory-bound
  - Low arithmetic intensity
  - Bottlenecked by HBM bandwidth

Mixed step without chunking:
  If Req B has N=4096 prompt tokens:
    Step: [1 decode token from A] [4096 prefill tokens from B] [1 decode from C]
    
    Problem: The 4096-token prefill MONOPOLIZES the step
    Req A and C experience ~10×–50× higher inter-token latency (jank!)
    TTFT for all waiting requests increases dramatically
```

### Chunked Prefill: The Solution

**Chunked prefill** (introduced in vLLM V0 as optional, default in V1) splits long prefill requests into budget-sized chunks:

```
Chunked Prefill (chunk_size = 512 tokens/step):

Without chunked prefill:
  Step 1: [PREFILL 4096 tok of Req A alone]  ← 4096-token latency spike!
  Step 2: [DECODE all running requests]

With chunked prefill (chunk_size=512):
  Step 1: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D]  ← bounded!
  Step 2: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 3: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 4: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 5: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 6: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 7: [PREFILL 512 tok of Req A] + [DECODE Req B, C, D, E]
  Step 8: [PREFILL last 512 tok → Req A enters decode]

Benefits:
  ✓ Decode requests not starved: each step includes all decode requests
  ✓ Bounded ITL: max inter-token latency ≤ step_time × 1 chunk overhead
  ✓ Better GPU utilization: mixing compute-heavy prefill + memory-heavy decode
    is more balanced than either alone (improves arithmetic intensity)
  ✓ TTFT predictability: long prefills don't cause unpredictable latency spikes

Cost:
  ✗ TTFT for prefill request increases linearly with chunks
    (First token takes: ceil(N/chunk_size) × step_time instead of 1 step)
  ✗ More complex scheduler logic
```

### Scheduling Priority Order

```python
# Simplified V0 chunked-prefill scheduler:
def schedule():
    budget = SchedulingBudget(
        max_num_seqs=256,
        max_num_batched_tokens=8192  # includes both prefill chunks + decode
    )
    
    # Priority 1: Schedule all currently-decoding requests
    # (Decode requests get priority to minimize jank)
    for req in running_queue:
        if req.is_decoding:
            if budget.can_schedule(req, num_tokens=1):
                schedule_decode(req, budget)
    
    # Priority 2: Continue in-progress chunked prefills
    # (A prefill already started must continue until complete)
    for req in running_queue:
        if req.is_prefilling:
            chunk_size = min(budget.remaining_tokens, chunk_budget)
            schedule_prefill_chunk(req, chunk_size, budget)
    
    # Priority 3: Swap in preempted requests (if memory allows)
    for req in swapped_queue:
        if can_swap_in(req) and budget.can_schedule(req):
            swap_in_and_schedule(req, budget)
    
    # Priority 4: Admit new requests from waiting queue
    for req in waiting_queue:
        if budget.can_schedule(req) and memory_sufficient(req):
            admit_and_schedule_prefill(req, budget)
    
    return scheduled_requests, scheduler_outputs

# V1 scheduler is simpler — no prefill/decode distinction in representation:
# Scheduler output format: {request_id: num_tokens_to_process_this_step}
# Both prefill chunks and decode single-tokens use the same format
```

## Throughput Analysis

### Why Continuous Batching Improves Throughput

The fundamental insight is that GPU throughput is limited by **batch size** (for decode) and **non-padding tokens** (for all phases):

```
Throughput Model (decode-dominated workload):

  Static batching:
    Batch size B throughout = B×output_len computations
    But effective batch shrinks as requests finish
    Average effective batch = B/2 (for uniform output lengths)
    Throughput ≈ (avg_batch/2) × throughput_per_seq

  Continuous batching:
    When any request finishes, immediately admit a new one
    Average batch size = B (full utilization)
    Throughput ≈ B × throughput_per_seq
    
    Improvement: ~2× for uniform lengths, much more for skewed distributions

For skewed length distributions (Pareto tail — common in real workloads):
  Static: rare long requests dominate batch time
    → short requests waste most of the batch duration padding
    → utilization may drop to 10–20%
  
  Continuous: short requests complete and are replaced immediately
    → batch stays full with diverse lengths
    → utilization remains 80–95%
```

### Throughput Results from SOSP '23

```
Benchmark: OPT-13B on A100-40GB, ShareGPT dataset
           (Workload: real-world chat conversations, highly variable lengths)

System              Avg Batch Size    Max Sustainable Rate
─────────────────────────────────────────────────────────────────
vLLM                    30.42         ████████████████████ (baseline)
Orca (Oracle)*          13.62         ████████             (0.37× relative)
Orca (Max)               7.00         ████▌                (0.20× relative)
FasterTransformer        7.00         ██                   (0.05× relative)
HuggingFace TGI         N/A           █▌                   (0.04× relative)
─────────────────────────────────────────────────────────────────
* Orca Oracle: uses oracle future output lengths (impossible in practice)

Key observations:
  1. vLLM achieves 4.3× larger average batch size than Orca Oracle
     → PagedAttention memory efficiency directly enables larger batches
  2. Throughput improvement: 2×–22× vs prior art
     → Range because different workloads benefit differently
  3. vLLM vs FasterTransformer: 20× improvement
     → FasterTransformer uses contiguous KV → severe fragmentation
```

### Batch Size vs. Latency Trade-off

```
Throughput-Latency Frontier (decode-dominated):

  Throughput
  (tok/s)   ↑
            │                         ×─── saturation (compute-bound)
            │                      ×
            │                   ×
            │               ×
            │           ×
            │       ×
            │   ×
            └───────────────────────────────────────→ Batch Size
                   ↑           ↑
              optimal for    optimal for
              latency SLO    throughput SLO

Operating Point Selection:
  Interactive (chat) applications:
    → Small-medium batch, prioritize low TTFT and TPOT
    → Typical: 16–64 concurrent sequences
  
  Batch/offline processing:
    → Maximum batch size, maximize throughput
    → Typical: 256–512+ concurrent sequences (GPU compute-bound)
  
  Mixed SLO:
    → Target P95 TPOT < 100ms while maximizing throughput
    → Dynamically adjust batch size based on SLO attainment
```

## Scheduling Budget Constraints

The scheduler enforces hard limits each step to keep step latency bounded:

```
Budget Parameters:

max_num_seqs = 256
  → Maximum concurrent sequences (prefill + decode combined)
  → Limits memory for block tables, input metadata
  → GPU attention kernel complexity grows with batch size

max_num_batched_tokens = 8192
  → Maximum total tokens processed in one step
  → Limits step duration (≈ max_step_latency constraint)
  → Shared budget between prefill chunks and decode tokens
  
  Example allocation:
    64 decode sequences × 1 token each  =   64 tokens
    Remaining for prefill: 8192 - 64    = 8128 tokens
    → Up to 8128/chunk_size = 15 prefill chunks per step

Admission Decision Each Step:
  for req in waiting_queue:
    tokens_needed = min(req.remaining_prefill, chunk_size)
    
    if (current_seqs + 1 <= max_num_seqs
        AND current_tokens + tokens_needed <= max_num_batched_tokens
        AND free_blocks >= ceil(tokens_needed / block_size) + watermark):
      admit(req)
    else:
      break  # stop trying to admit; rest of waiting queue must wait

Note: FCFS ordering in waiting queue means requests admitted in arrival order
      (subject to the budget constraints above)
```

## Impact on Key Metrics

### TTFT (Time to First Token)

```
TTFT with continuous batching:

  TTFT = queuing_delay + prefill_latency

  queuing_delay:
    Static: wait for full batch to complete → O(batch_size × avg_output_length)
    Continuous: wait only until a slot is available → much shorter
    
    For steady-state serving:
      queuing_delay ≈ arrival_rate × step_time / available_slots
    
  prefill_latency:
    Without chunked prefill: process all N prompt tokens in one step
      → O(N²) attention → can be 100ms–2s for long prompts
    With chunked prefill: N tokens split across K steps
      → First chunk takes O(chunk_size²) attention → bounded per-step time
      → Total TTFT = K × step_time (but step_time is shared with decode)

  TTFT comparison (4096-token prompt, 512 chunk_size):
    Static: queuing + single 4096-token prefill = long, unpredictable
    Continuous+chunked: queuing + 8 × 512-token chunks
      Each chunk shares step with decode → lower per-step overhead
```

### TPOT (Time Per Output Token) / ITL

```
TPOT with continuous batching:

  Step time (decode-dominated) ≈ f(batch_size, context_length, model_size)
  TPOT per request = step_time (since each step generates 1 token per sequence)
  
  Effect of batch size on TPOT:
    Larger batch → longer step_time (more tokens to process per step)
    → each individual request's TPOT increases
    → throughput increases (more total tokens/step)
  
  With chunked prefill, TPOT can spike when prefill chunks are large:
    TPOT_spike = step_time_with_large_chunk / 1_decode_token
    
  V1 FlashAttention 3 helps here:
    Better handles mixed prefill+decode batches (variable-length attention)
    Reduces the per-step overhead of mixed batches
    → Lower TPOT even with active prefill chunks in the batch

Typical TPOT values (LLaMA-2-70B, H100):
  Batch size 1:    ~5ms/token  (200 tokens/sec)
  Batch size 32:  ~20ms/token  (1600 tokens/sec total, 50 tok/sec each)
  Batch size 128: ~60ms/token  (2130 tokens/sec total, 17 tok/sec each)
```

## V1 Improvements to Continuous Batching

vLLM V1 (alpha January 2025) introduces several improvements to the continuous batching pipeline:

### Persistent Batch (InputBatch)

```
V0: Rebuild input tensors from scratch every step
  # Python overhead for each step:
  input_ids = []
  for seq in scheduled_sequences:
    input_ids.append(seq.get_last_token())  # Python list append
  input_ids = torch.tensor(input_ids).cuda()  # CPU→GPU transfer
  # Repeated N times per second → significant Python GIL overhead

V1: Persistent Batch (InputBatch)
  # Pre-allocated NumPy arrays, updated incrementally:
  class InputBatch:
    input_ids:     np.ndarray  # shape [max_seqs]
    positions:     np.ndarray
    slot_mapping:  np.ndarray  # shape [max_seqs × max_blocks]
    block_table:   np.ndarray
    
    def update(self, step_delta):
      # Remove finished sequences (index operations)
      # Update positions for continuing decode sequences (+= 1 in-place)
      # Add new sequences (append to pre-allocated region)
      # Uses NumPy in-place ops → avoids Python object creation
    
    def to_gpu(self):
      # Single torch.as_tensor() + .cuda() call
      # Only copies the active portion of pre-allocated arrays
      # Much less CPU→GPU data transfer than rebuilding tensors

Performance improvement:
  Input preparation overhead:
    V0: ~2–5ms per step (Python loop + tensor construction)
    V1: ~0.1–0.5ms per step (NumPy in-place + single transfer)
  
  At 100 steps/second: saves 200–450ms/sec of CPU overhead
```

### Scheduler Simplification

```
V0 Scheduler:
  Tracks prefill vs. decode state explicitly
  Different code paths for prefill requests vs. decode requests
  Complex interaction between 3 queues (waiting/running/swapped)
  Chunked prefill as optional add-on → complex logic

V1 Scheduler:
  Unified representation: {request_id: num_tokens_to_process}
  Whether a request is prefilling or decoding is irrelevant to scheduler
  → Both map to "process N tokens for this request this step"
  
  Two queues only: waiting, running
  Swapped queue removed (recompute is default preemption)
  
  Scheduling output format (simpler):
    scheduled: Dict[request_id, num_tokens]  # unified
    preempted: List[request_id]
    # No blocks_to_swap_in/out (recompute default)
    blocks_to_copy: List[(src, dst)]  # CoW only
```

### CPU-GPU Pipeline Overlap

```
V1 Pipeline:
  GPU: [forward pass t-1]
  CPU: [tokenize new requests] [update InputBatch for step t]
  ↓
  GPU: [forward pass t]
  CPU: [detokenize outputs from t-1] [send to clients]
  ↓
  GPU: [forward pass t+1]
  CPU: [process new request arrivals] [update InputBatch for t+1]

V0 Pipeline (sequential):
  CPU: [schedule + prepare inputs for t]
  GPU: [forward pass t]
  CPU: [process outputs from t] [send to clients]
  GPU: idle while CPU processes!
  ← CPU is on the critical path of GPU scheduling

V1 improvement:
  Tokenization, detokenization, multimodal preprocessing
  all run in separate processes concurrently with GPU forward pass
  → GPU is never idle waiting for CPU preprocessing
  → Throughput improvement: up to 1.7× over V0 for some workloads
```

## Comparison with Alternative Approaches

```
Batching Strategy Comparison:
┌──────────────────────────┬────────────────────────────────────────────────┐
│ Strategy                 │ Characteristics                                │
├──────────────────────────┼────────────────────────────────────────────────┤
│ Static Batching          │ Fixed batch until all complete                 │
│                          │ Simple, but poor GPU utilization               │
│                          │ High queuing latency for new requests          │
│                          │ Suitable only for offline, uniform workloads   │
├──────────────────────────┼────────────────────────────────────────────────┤
│ Dynamic Batching         │ Admits new requests between batches            │
│                          │ Still has inter-batch barriers                 │
│                          │ Better than static but not iteration-level     │
├──────────────────────────┼────────────────────────────────────────────────┤
│ Continuous Batching      │ Iteration-level scheduling                     │
│ (Orca, vLLM)             │ No padding, no barriers                        │
│                          │ Maximum GPU utilization                        │
│                          │ Suitable for online serving                    │
├──────────────────────────┼────────────────────────────────────────────────┤
│ Continuous + Chunked     │ Continuous batching + bounded prefill chunks   │
│ Prefill (vLLM V1)        │ Prevents prefill from starving decode          │
│                          │ Predictable TPOT even with long prompts        │
│                          │ Current best practice for production serving   │
├──────────────────────────┼────────────────────────────────────────────────┤
│ Disaggregated Prefill    │ Separate clusters for prefill and decode       │
│ (future/experimental)    │ Prefill cluster: compute-optimized (A100)      │
│                          │ Decode cluster: memory-bandwidth-optimized     │
│                          │ Eliminates prefill-decode interference         │
│                          │ High infrastructure complexity                 │
└──────────────────────────┴────────────────────────────────────────────────┘
```

---
*Related: [[vLLM Architecture]], [[PagedAttention]], [[KV Cache Management]], [[LLM Inference Serving]]*
