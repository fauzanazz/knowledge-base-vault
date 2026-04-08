# KV Cache Management

KV cache management in LLM serving systems refers to the allocation, tracking, sharing, and reclamation of GPU memory used to store transformer attention key-value states across the decode steps of autoregressive generation. In vLLM, this is handled by the `BlockSpaceManager` (V0) / KV Cache Manager (V1), which implements a paged memory allocator analogous to an operating system's physical page manager. Efficient KV cache management is the primary factor distinguishing high-throughput LLM serving from naive implementations.

## Why KV Cache Management Is Complex

Each active inference request holds a growing KV cache that:

1. **Grows dynamically** — one new block potentially allocated per `block_size` decode steps
2. **Has unknown final size** — output length is not known at request admission time
3. **Can be shared** — multiple request sequences (beam search branches, parallel samples) share common prefix KV data
4. **Must be preemptible** — when memory is exhausted, some requests must be temporarily evicted to serve others
5. **Must be efficiently tracked** — the GPU attention kernel needs O(1) lookup of physical block locations

```
KV Cache Lifecycle:
                          ┌──────────────────────────────────┐
Request arrives ─────────→│  Scheduler: allocate_slots()     │
                          │  Check free_block_queue           │
                          │  Assign physical blocks           │
                          └──────────────┬───────────────────┘
                                         │ blocks assigned
                          ┌──────────────▼───────────────────┐
                          │  Running: decode steps            │
                          │  Each new token → may need block  │
                          │  Shared blocks: CoW on divergence │
                          └──────────────┬───────────────────┘
              ┌───────────────────────────┤
              │ memory pressure           │ request completes
              ▼                           ▼
   ┌──────────────────────┐   ┌──────────────────────────────┐
   │  Preemption:         │   │  free_slots()                │
   │  Swap or Recompute   │   │  ref_count[block] -= 1       │
   │  (see §4)            │   │  if 0: push to free_pool     │
   └──────────────────────┘   └──────────────────────────────┘
```

## BlockSpaceManager Architecture

The `BlockSpaceManager` is the central component managing all KV cache memory:

```
┌──────────────────────────────────────────────────────────────┐
│                    BlockSpaceManager                         │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  GPU Block Allocator                                    │ │
│  │                                                         │ │
│  │  free_block_queue: [blk_45, blk_12, blk_78, ...]       │ │
│  │  (doubly-linked list, O(1) push/pop)                    │ │
│  │                                                         │ │
│  │  Physical blocks: pre-allocated in GPU VRAM             │ │
│  │  block_ref_counts: [1, 0, 5, 2, 0, 1, ...]             │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  CPU Block Allocator (Swap Space)                       │ │
│  │                                                         │ │
│  │  cpu_free_block_queue: [cpu_blk_0, cpu_blk_1, ...]     │ │
│  │  (blocks in CPU RAM — used for swap-out preemption)     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Per-Request Block Tables                               │ │
│  │                                                         │ │
│  │  block_table[req_A] = [PhysBlock(7), PhysBlock(1), ...] │ │
│  │  block_table[req_B] = [PhysBlock(7), PhysBlock(1), ...] │ │
│  │      ↑ req_A and req_B share blocks 7 and 1!           │ │
│  │  block_table[req_C] = [PhysBlock(3), PhysBlock(9), ...] │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Prefix Cache (V1, optional in V0)                      │ │
│  │                                                         │ │
│  │  hash(token_ids) → PhysBlock                            │ │
│  │  LRU eviction doubly-linked list                        │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Physical Block Representation

Each physical block is a lightweight metadata object (the actual KV data lives in pre-allocated GPU tensors):

```python
class PhysicalTokenBlock:
    device: str          # "cuda" or "cpu"
    block_number: int    # index into key_cache/val_cache tensors
    block_size: int      # tokens per block (e.g., 16)
    ref_count: int       # number of sequences referencing this block
    last_accessed: float # timestamp for LRU eviction
    computed: bool       # True if KV has been computed (prefix cache)
    block_hash: int      # hash of token_ids (for prefix cache lookup)
```

## Block Allocation Lifecycle

### Initial Allocation (Request Admission)

When a new request is admitted from the waiting queue, the scheduler calls `allocate_and_set_block_tables`:

```
Initial Allocation for Request with N prompt tokens:

  num_prompt_blocks = ceil(N / block_size)
  
  Check feasibility:
    if free_block_queue.size() < num_prompt_blocks:
      → request cannot be admitted this step
      → either try to preempt another request
      → or leave request in waiting queue
  
  If feasible:
    prompt_blocks = []
    for i in range(num_prompt_blocks):
      block = free_block_queue.pop()       # O(1) dequeue
      block.ref_count = 1
      prompt_blocks.append(block)
    
    block_table[request_id] = prompt_blocks
    
    # Last block may be partially filled:
    last_block.num_filled_slots = N % block_size
    # If N % block_size == 0, last block is full → next token
    # will require a new block allocation

Example (N=20 tokens, block_size=16):
  Block 0: slots 0-15 → 16 tokens (full)
  Block 1: slots 0-3  → 4 tokens (partial, 12 slots reserved for decode)
```

### Incremental Allocation During Decode

As decode progresses, new tokens are generated. A new physical block is needed when the last block fills up:

```
Each decode step for a running request:

  allocate_slot_for_decode(request):
    last_block = block_table[request][-1]
    
    if last_block.num_filled_slots < block_size:
      # Space in existing block — no new allocation needed
      last_block.num_filled_slots += 1
      slot = last_block.block_number × block_size
             + (last_block.num_filled_slots - 1)
      return (slot, None)  # no CoW needed
    
    else:  # last block is full
      # Need new block
      if free_block_queue.is_empty():
        → trigger preemption of another request
      
      new_block = free_block_queue.pop()
      new_block.ref_count = 1
      new_block.num_filled_slots = 1
      block_table[request].append(new_block)
      slot = new_block.block_number × block_size + 0
      return (slot, None)

Memory growth pattern for a 100-token output (block_size=16):
  Token 1-16:  uses block A (pre-allocated during prefill)
  Token 17:    allocates block B
  Token 33:    allocates block C
  Token 49:    allocates block D
  Token 65:    allocates block E
  Token 81:    allocates block F
  Token 97:    allocates block G
  → 7 blocks total for 100 output tokens
```

### Deallocation on Completion

When a request finishes (EOS token, max_tokens reached, or stop string matched):

```
free_slots(request_id):
  for each block in block_table[request_id]:
    block.ref_count -= 1
    if block.ref_count == 0:
      if block in prefix_cache:
        # Keep in prefix cache (may reuse for future requests)
        # LRU position updated to "just freed"
        prefix_cache.update_lru(block)
      else:
        # Return to free pool immediately
        free_block_queue.push(block)  # O(1)
  
  del block_table[request_id]

Note: Reference-counted blocks shared between sequences
  (e.g., parallel samples sharing prompt blocks) are only
  returned to the free pool when ALL sharing sequences finish.
```

## Reference Counting

Reference counting is the mechanism that enables safe block sharing without copying:

```
Reference Count Rules:
─────────────────────────────────────────────────────────────
Initial allocation:     ref_count = 1
fork (sequence splits): ref_count += 1 per child
free (sequence ends):   ref_count -= 1
                        if ref_count == 0 → return to pool

Invariant:
  ref_count == number of sequences whose block_table contains
               a reference to this physical block

Safety guarantee:
  A block with ref_count > 0 will NEVER be returned to the
  free pool or overwritten. Only ref_count == 0 blocks are
  available for reallocation.
─────────────────────────────────────────────────────────────

Reference Count Evolution Example:
  (block_size=4 for illustration, parallel sampling n=3)
  
  Time 0: Prompt "Hello" processes → 2 blocks allocated
    block[7].ref_count = 1  (owned by seq_0)
    
  Time 1: fork(seq_0, seq_1), fork(seq_0, seq_2)
    block[7].ref_count = 3  (shared by seq_0, seq_1, seq_2)
    
  Time 2: seq_1 generates token → CoW on last block
    Allocate block[12], copy block[7] → block[12]
    block[7].ref_count = 2  (seq_0 and seq_2 still share)
    block[12].ref_count = 1  (seq_1 exclusively owns)
    
  Time 3: seq_0 finishes generation
    block[7].ref_count = 1  (only seq_2 now)
    
  Time 4: seq_2 finishes generation
    block[7].ref_count = 0  → pushed to free_block_queue
    block[12].ref_count = 0  → pushed to free_block_queue
```

## Memory Sharing Scenarios

### Scenario 1: Parallel Sampling

When a user requests N independent completions from one prompt (sampling `n > 1`):

```
User request: "Write a haiku about autumn" → generate 4 variations
  SamplingParams(n=4, temperature=1.0)

Phase 1 — Prefill:
  Prompt tokenized → 8 tokens → 1 physical block needed
  Allocate block[5]: ref_count = 1 (initial)
  block_table[seq_0] = [block[5]]
  
  fork(seq_0, seq_1): block[5].ref_count = 2
  fork(seq_0, seq_2): block[5].ref_count = 3
  fork(seq_0, seq_3): block[5].ref_count = 4
  
  All 4 sequences now share block[5]:
  block_table[seq_0] = [block[5]]  ← all point to same physical block
  block_table[seq_1] = [block[5]]
  block_table[seq_2] = [block[5]]
  block_table[seq_3] = [block[5]]
  
  Memory used: 1 block (not 4 blocks!)

Phase 2 — First decode step:
  seq_0 generates "Crimson" → append to block[5]
  block[5].ref_count = 4 > 1 → CoW triggered:
    Allocate block[11], copy block[5] → block[11]
    block_table[seq_0][-1] = block[11]
    block[5].ref_count = 3
    Write "Crimson" KV to block[11]
  
  seq_1 generates "Falling" → append, CoW:
    Allocate block[14], copy block[5] → block[14]
    block_table[seq_1][-1] = block[14]
    block[5].ref_count = 2
    Write "Falling" KV to block[14]
  
  seq_2 generates "Amber" → CoW:
    block[5].ref_count → 1 after copy
    (seq_3 still shares block[5])
  
  seq_3 generates "Golden" → CoW (ref_count=1→still 1, but must still CoW
    because CoW is triggered when ref>1 at time of WRITE attempt):
    Actually: at this point ref_count=1 (only seq_3 holds block[5])
    → NO CoW needed! Write directly to block[5]

Final state:
  block_table[seq_0] = [block[11]]  (diverged)
  block_table[seq_1] = [block[14]]  (diverged)
  block_table[seq_2] = [block[17]]  (diverged)
  block_table[seq_3] = [block[5]]   (took ownership)
  
  4 blocks used total (1 shared prompt block effectively, each owns their copy)
  vs. 4 blocks without sharing (no savings at first step)
  For longer prompts: savings = (n-1) × num_prompt_blocks
```

### Scenario 2: Beam Search

Beam search maintains `k` candidate sequences (beams) at each step, keeping only the highest-scoring ones. The block table structure enables efficient tree memory management:

```
Beam Search (k=4) — Step-by-Step Memory View:

Initial: 4-token prompt → 1 block
  block[0].ref_count = 1

Prefill done → fork into k=4 beams:
  block[0].ref_count = 4
  beam[0].block_table = [block[0]]
  beam[1].block_table = [block[0]]
  beam[2].block_table = [block[0]]
  beam[3].block_table = [block[0]]

Step 1: Each beam generates one token (all blocks full → new blocks)
  CoW: each beam gets its own new block, block[0] stays shared
  beam[0].block_table = [block[0], block[1]]
  beam[1].block_table = [block[0], block[2]]
  beam[2].block_table = [block[0], block[3]]
  beam[3].block_table = [block[0], block[4]]
  block[0].ref_count = 4  (still shared)
  
  Memory: 5 blocks (1 shared + 4 unique)
  vs. 8 blocks without sharing = 37.5% savings

Step 2: Beams scored; beam[2] wins, and it forks:
  (Beam search may select the same beam multiple times
   when it's the best candidate → fork copies its block table)
  fork(beam[2], new_beam_candidate):
    block[3].ref_count = 2  (beam[2] and candidate share)

Step 5: Beam[3] is pruned (score too low):
  free(beam[3]):
    block[0].ref_count -= 1  → 3
    block[4].ref_count -= 1  → 0 → pushed to free pool!
  
  Pruned beam's unique blocks freed immediately
  → Available for new decode allocations

Final completion:
  When best beam finishes, free all its blocks
  Shared block[0] freed only when ALL references drop to 0

Memory savings reported (SOSP '23, OPT-13B, ShareGPT):
  Beam search (k=4):  44.3%–66.3% KV cache reduction
```

### Scenario 3: Shared System Prompts (Prefix Caching)

Many production deployments use a fixed system prompt for all requests (e.g., "You are a helpful assistant..."). Prefix caching shares these KV blocks across all requests:

```
Prefix Cache Interaction:

Request 1: "You are a helpful assistant. [User: What is 2+2?]"
  ─────────────────────────────────────────────────────────────
  Prompt = [sys_tok_0..15][sys_tok_16..31][user_tok_0..3]
              ↑ Block A               ↑ Block B     ↑ Block C
  
  Block A: hash = H(tok_0..15)
  Block B: hash = H(tok_0..31) = H(tok_16..31 | parent=H(tok_0..15))
  Block C: hash = H(tok_0..35) = H(user_tok | parent=H(tok_0..31))
  
  prefix_cache[H_A] = physical_block_7   (newly computed)
  prefix_cache[H_B] = physical_block_1
  prefix_cache[H_C] = physical_block_3

Request 2 (same system prompt): "You are a helpful assistant. [User: What is 3+3?]"
  ─────────────────────────────────────────────────────────────
  Scheduling check:
    Block A: hash H_A → HIT → reuse physical_block_7, ref_count[7]++
    Block B: hash H_B → HIT → reuse physical_block_1, ref_count[1]++
    Block C: hash H_C → MISS (different user query) → allocate new
    
  Prefill for Request 2:
    Blocks 7, 1: SKIP (already computed, just included in block_table)
    Block C (new): must compute KV for user tokens
    
  TTFT improvement:
    Without prefix cache: prefill all 36 tokens → N² attention
    With prefix cache: prefill only 4 user tokens
    → ~9× less prefill compute for this request!

Eviction (when pool is exhausted):
  LRU policy applied to prefix_cache entries with ref_count == 0:
    Find least recently used unreferenced block
    Remove from prefix_cache dict
    Push to free_block_queue
    (The physical block tensor is overwritten on next allocation)

V1 LRU Implementation:
  doubly-linked list tracks LRU order:
    Most recently used ← [blk_A ↔ blk_B ↔ blk_C ↔ blk_D] → Least recently used
  
  On hit: move block to front of list (O(1) with doubly-linked list)
  On evict: pop from back of list (O(1))
  Overall eviction: O(1) — critical for high request rates
```

## Preemption and Swapping

When the GPU KV cache is exhausted and a new request cannot be admitted, the scheduler must **preempt** one or more running requests to free memory:

### Gang Eviction Policy

```
Preemption Selection Rule:
  Policy: FCFS with Last-In-First-Out (LIFO) eviction
  
  The most recently started request is preempted first
  Rationale: It has invested the least prefill compute
             (minimizes wasted work)

  For multi-sequence groups (beam search):
    The ENTIRE group is evicted together ("gang eviction")
    Rationale: Partial eviction of a beam group would leave
               the group in an inconsistent state (some beams
               freed, others still running → broken)
    
  Implication: One large beam search group can starve other
               requests in pathological cases (mitigated by
               admission control)
```

### Recovery Method 1: Recomputation (Default in V1)

```
Preemption via Recomputation:
  
  Step 1: Evict the selected request
    For each physical block in block_table[request]:
      ref_count[block] -= 1
      if ref_count == 0: free_block_queue.push(block)
    del block_table[request]
  
  Step 2: Re-queue the request
    Add request back to WAITING queue
    (at head of queue to avoid re-starvation, or configured)
    Request retains: prompt tokens, sampling_params, arrival_time
    Request loses: ALL KV cache (must recompute from scratch)
  
  Step 3: Resumption (when blocks available again)
    Request goes through full prefill again
    KV cache rebuilt from prompt + already-generated tokens
    (Already-generated tokens are re-fed as part of the prompt)

Overhead analysis:
  CPU→GPU bandwidth: 0 (no data movement needed)
  GPU compute overhead: Full prefill cost for prompt
  Memory during preemption: 0 extra (blocks freed immediately)
  
  In practice (SOSP '23):
    Recompute overhead < 20% of swap overhead
    Best for: Single-sequence requests, short sequences

Trade-off summary:
  ✓ No PCIe bandwidth consumed
  ✓ No CPU swap memory needed
  ✗ Requires full prefill recomputation on resumption
  ✗ Cannot preserve multi-sequence groups (beam search)
```

### Recovery Method 2: Swapping

```
Preemption via Swap-Out:

  Step 1: Copy GPU KV blocks → CPU RAM
    For each GPU block in block_table[request]:
      cpu_block = cpu_free_queue.pop()
      memcpy(gpu_block → cpu_block)  ← PCIe transfer
      cpu_block_table[request].append(cpu_block)
    free_block_queue.push_all(gpu_blocks)  ← GPU blocks now free
  
  Step 2: Move request to SWAPPED queue (V0) or mark as swapped
    Request: gpu_blocks freed, cpu_blocks holding KV state

  Step 3: Resumption (swap-in)
    When sufficient GPU blocks are available:
      for each cpu_block in cpu_block_table[request]:
        gpu_block = free_block_queue.pop()
        memcpy(cpu_block → gpu_block)  ← PCIe transfer back
        new_block_table[request].append(gpu_block)
      cpu_free_queue.push_all(cpu_blocks)
    Move request from SWAPPED → RUNNING

PCIe Bandwidth Usage:
  Swap out: KV_size_of_request / PCIe_bandwidth seconds
  A100 PCIe 4.0 (64 GB/s bidirectional):
    1 GB of KV data → ~16ms to swap out
  
  For OPT-13B at 2 KB/token × 1000 tokens = 2 GB:
    Swap out + in: ~64ms additional latency

Swap space configuration:
  --swap-space 4  (GB per GPU, default)
  cpu_num_blocks = (4 GB) / block_bytes
  
  Maximum requests that can be swapped simultaneously:
    = cpu_num_blocks × block_size / avg_tokens_per_request

When swapping is preferred over recompute:
  Multi-sequence groups (beam search): recompute impossible
                                       (cannot reconstruct all beam paths)
  Long sequences: swap cheaper than recomputing long prefill
  Large block sizes: fewer copies needed to swap

Swap vs. Recompute Decision:
┌────────────────────┬──────────────────┬──────────────────┐
│ Factor             │ Prefer Swap      │ Prefer Recompute │
├────────────────────┼──────────────────┼──────────────────┤
│ Sequence type      │ Multi-seq (beam) │ Single sequence  │
│ Sequence length    │ Long (>1K tokens)│ Short (<500 tok) │
│ Block size         │ Large (32, 64)   │ Small (4, 8)     │
│ PCIe bandwidth     │ Fast NVLink      │ Slow PCIe 3.0    │
│ CPU memory         │ Available        │ Scarce           │
│ GPU compute        │ Busy             │ Available        │
└────────────────────┴──────────────────┴──────────────────┘
```

## Scheduling and Memory Decisions

The scheduler checks memory availability before every scheduling decision:

```
can_allocate(request) → bool:
  num_required_blocks = ceil(request.num_tokens / block_size)
  return free_block_queue.size() >= num_required_blocks

can_append_slot(sequence) → Optional[CoWBlock]:
  last_block = block_table[sequence][-1]
  if last_block.num_filled_slots < block_size:
    if last_block.ref_count == 1:
      return None  # Write directly, no allocation needed
    else:
      return CoW_needed(last_block)  # Need to copy-on-write
  else:
    if free_block_queue.is_empty():
      return PREEMPTION_NEEDED
    return NEW_BLOCK_NEEDED

get_num_free_blocks() → int:
  return free_block_queue.size()  # O(1)

Budget constraints checked each step:
  max_num_seqs:           256 (configurable)
  max_num_batched_tokens: 8192 (configurable)
  GPU memory headroom:    watermark of N free blocks maintained
                          (prevents over-admission that would
                          immediately trigger preemption)
```

### Watermark Policy

vLLM maintains a **watermark** — a minimum number of reserved free blocks — to avoid thrashing:

```
Watermark Mechanism:
  watermark_blocks = max_num_seqs × 1  (at least 1 free block
                                         per possible running sequence)
  
  Admission check:
    if free_blocks - blocks_needed < watermark_blocks:
      → do NOT admit new request this step
      (even if technically enough blocks exist for the new request)
  
  Rationale:
    Running decode requests will each need 1 new block roughly
    every block_size steps. If we leave zero free blocks,
    we'll immediately need to preempt.
    Watermark ensures running requests can continue a few steps
    before we need to admit/preempt again.

  Example:
    max_num_seqs = 256, block_size = 16
    watermark = 256 blocks = 256 × 320KB ≈ 80MB reserved headroom
    (small relative to typical 12GB KV cache budget)
```

## Memory Statistics and Monitoring

The BlockSpaceManager exposes memory statistics for monitoring and adaptive scheduling:

```
Key Metrics:
  num_total_gpu_blocks:  Total blocks in GPU allocator
  num_free_gpu_blocks:   Currently available blocks
  num_used_gpu_blocks:   Blocks held by active sequences
  num_total_cpu_blocks:  Total blocks in CPU swap space
  num_free_cpu_blocks:   Available CPU swap blocks
  
  gpu_cache_usage_sys:   num_used / num_total (should be 80–96%)
  cpu_cache_usage_sys:   cpu_used / cpu_total
  
  prefix_cache_hit_rate: (blocks reused from cache) /
                         (total blocks allocated)
                         (logged per request if prefix caching on)

Healthy system profile (V1 defaults):
  GPU cache usage:      70–96% (high utilization)
  Preemption rate:      <5% of requests (infrequent preemption)
  Prefix cache hit:     40–80% (for typical chat workloads with
                                shared system prompts)
  CoW copy rate:        <10% of steps (rare for most workloads)
```

## V0 vs V1 BlockSpaceManager Differences

```
┌────────────────────────┬───────────────────┬─────────────────────┐
│ Feature                │ V0                │ V1                  │
├────────────────────────┼───────────────────┼─────────────────────┤
│ Data structure         │ Dict of lists     │ Array-based         │
│                        │ (Python objects)  │ (NumPy arrays)      │
├────────────────────────┼───────────────────┼─────────────────────┤
│ Prefix caching         │ Optional, O(n)    │ Default, O(1) LRU  │
│                        │ eviction          │ doubly-linked list  │
├────────────────────────┼───────────────────┼─────────────────────┤
│ Preemption default     │ Swap (multi-seq)  │ Recompute (default) │
│                        │ Recompute (single)│ Swap (multi-seq only)│
├────────────────────────┼───────────────────┼─────────────────────┤
│ Scheduler queues       │ 3: waiting/running│ 2: waiting/running  │
│                        │ /swapped          │ (swapped removed)   │
├────────────────────────┼───────────────────┼─────────────────────┤
│ Block state transfer   │ Metadata + data   │ Only diff from prev │
│ to workers             │ per step          │ step (incremental)  │
├────────────────────────┼───────────────────┼─────────────────────┤
│ Prefix cache overhead  │ O(n) eviction     │ <1% overhead at 0%  │
│                        │ = 20% overhead at │ hit rate (constant- │
│                        │ 0% hit rate       │ time LRU)           │
└────────────────────────┴───────────────────┴─────────────────────┘
```

## CacheEngine: Physical KV Operations

The `CacheEngine` (one per Worker/GPU) is responsible for actually executing swap and copy operations on the KV tensors:

```
CacheEngine maintains:
  gpu_cache: List[Tuple[Tensor, Tensor]]
    → gpu_cache[layer] = (key_cache_tensor, val_cache_tensor)
    → key_cache_tensor: shape [num_gpu_blocks, n_kv_heads, head_dim//x, block_size, x]
    → val_cache_tensor: shape [num_gpu_blocks, n_kv_heads, head_dim, block_size]

  cpu_cache: List[Tuple[Tensor, Tensor]]
    → Same structure but pinned CPU memory (for fast PCIe transfer)

CacheEngine operations:

  swap_in(src_to_dst: Dict[int, int]):
    # src = CPU block numbers, dst = GPU block numbers
    for layer in range(num_layers):
      for cpu_block, gpu_block in src_to_dst.items():
        gpu_cache[layer][0][gpu_block] ← cpu_cache[layer][0][cpu_block]  # K
        gpu_cache[layer][1][gpu_block] ← cpu_cache[layer][1][cpu_block]  # V
    
  swap_out(src_to_dst: Dict[int, int]):
    # src = GPU block numbers, dst = CPU block numbers
    (reverse of swap_in)

  copy(src_to_dst: Dict[int, int]):
    # Batched CoW copies: all GPU→GPU
    # Implemented as a SINGLE fused CUDA kernel (kernel 3)
    for layer in range(num_layers):
      ops.copy_blocks(gpu_cache[layer], src_to_dst)
      # ops.copy_blocks executes one CUDA kernel for all block copies
      # Avoids N separate kernel launches for N CoW operations

PCIe optimization for swap:
  cpu_cache uses pin_memory=True → enables DMA transfer
  Non-blocking async transfers overlap with compute where possible
  V1 explores deeper CPU-GPU overlap for swap operations
```

---
*Related: [[PagedAttention]], [[vLLM Architecture]], [[Continuous Batching]], [[LLM Inference Serving]]*
