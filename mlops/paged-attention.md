# PagedAttention

PagedAttention is a memory management algorithm for transformer attention that stores the Key-Value (KV) cache in non-contiguous, fixed-size memory blocks inspired by operating system virtual memory paging. Introduced in the SOSP '23 paper "Efficient Memory Management for Large Language Model Serving with PagedAttention" (Kwon et al., UC Berkeley/Stanford/UCSD), it achieves near-perfect (~96%) KV cache utilization versus 20–38% in prior systems, enabling 2×–22× higher throughput for LLM serving.

## The OS Virtual Memory Analogy

PagedAttention's design is a direct translation of classical OS virtual memory management into the GPU KV cache domain:

```
┌──────────────────────┬──────────────────────────────────────┐
│  OS Virtual Memory   │  PagedAttention (vLLM)               │
├──────────────────────┼──────────────────────────────────────┤
│  Pages               │  KV Blocks                           │
│  Bytes               │  Tokens                              │
│  Processes           │  Inference Requests                  │
│  Virtual address     │  Logical KV block index              │
│  Physical address    │  Physical KV block ID (GPU RAM)      │
│  Page table          │  Block table (per request)           │
│  Free frame pool     │  Free block queue (doubly LL)        │
│  Page fault          │  On-demand block allocation          │
│  Copy-on-Write (CoW) │  Block CoW at fork/parallel sampling │
│  Swap to disk        │  Swap KV blocks to CPU RAM           │
│  Demand paging       │  Allocate blocks only when needed    │
└──────────────────────┴──────────────────────────────────────┘
```

Just as an OS allows processes to use more virtual memory than physical RAM by managing page residency, PagedAttention allows the inference engine to manage many concurrent requests by treating KV blocks as first-class memory objects that can be shared, copied, and evicted.

## Why Contiguous KV Cache Allocation Fails

Before PagedAttention, all LLM serving systems pre-allocated a single contiguous memory region per request sized to the **maximum possible sequence length**. This created three sources of waste:

```
Memory Waste in Contiguous Allocation:

Source 1: Internal Fragmentation
  Request A: max_length=2048 allocated, actual output=200 tokens
  ┌──────────────────────────────────────────────────────────┐
  │ [████████ actual 200 tokens ░░░░░░░░░░░░░░░░░░░░░░░░░░] │
  │  ← used (10%) ──────────── WASTED (90%) ───────────────→ │
  └──────────────────────────────────────────────────────────┘

Source 2: External Fragmentation
  After some requests finish:
  [REQ_A | free 200 | REQ_B | free 300 | REQ_C | free 100]
  New request needs 512 contiguous tokens → FAILS
  (Total free = 600, but no contiguous 512-slot region)

Source 3: Future Token Reservation
  Output length is unknown at request start
  Must reserve worst-case (max_length) upfront
  Tokens not yet generated occupy slots from the start

Measurement from SOSP '23:
  Prior systems use only 20.4%–38.2% of KV cache memory
  vLLM achieves 96.3% utilization
```

## Block-Level Memory Layout

PagedAttention divides the entire GPU KV cache into fixed-size **KV blocks** of `B` tokens each (default `B = 16`):

```
Physical GPU KV Cache Memory:
┌────────────┬────────────┬────────────┬────────────┬────────────┐
│  Block  0  │  Block  1  │  Block  2  │  Block  3  │  Block  4  │
│ 16 tok slots│16 tok slots│16 tok slots│16 tok slots│16 tok slots│
│  [Req A]   │  [Req B]   │   [FREE]   │  [Req A]   │  [Req C]   │
└────────────┴────────────┴────────────┴────────────┴────────────┘
        ↑                                     ↑
   Req A's first block                 Req A's second block
   (non-contiguous but logically sequential)

Note: Req A uses blocks 0 and 3 — not adjacent in physical memory.
      This is fine because block tables track the mapping.
```

Each physical block stores KV data for ALL transformer layers simultaneously, for up to `B` tokens:

```
Block Internal Layout:
┌──────────────────────────────────────────────────────────────┐
│  Physical Block (block_id = 7)                               │
│                                                              │
│  Layer 0:  key_cache[7, 0, :, :, :]  val_cache[7, 0, :, :]  │
│  Layer 1:  key_cache[7, 1, :, :, :]  val_cache[7, 1, :, :]  │
│  ...                                                         │
│  Layer L:  key_cache[7, L, :, :, :]  val_cache[7, L, :, :]  │
│                                                              │
│  Per slot: n_kv_heads key vectors + n_kv_heads value vectors │
│            each vector is head_dim floats                    │
└──────────────────────────────────────────────────────────────┘

Block size in bytes (OPT-13B, FP16, B=16):
  = 2 (K+V) × 16 (tokens) × 40 (heads) × 128 (head_dim) × 2 bytes
  = 327,680 bytes ≈ 320 KB per block

Tensor shapes stored in GPU memory:
  key_cache: [num_blocks, n_kv_heads, head_dim/x, block_size, x]
  val_cache: [num_blocks, n_kv_heads, head_dim, block_size]
  (Transposed layout for optimal GPU memory access patterns)
```

### Default Block Size Rationale

The choice of `B = 16` balances several factors:

```
Block size tradeoffs:
┌────────────────┬────────────────────────────────────────────┐
│ B = 4          │ ✓ Low internal fragmentation (≤3 tokens)  │
│                │ ✗ More blocks, larger block tables        │
│                │ ✗ More kernel iterations per attention    │
│                │ ✗ More frequent CoW copies                │
├────────────────┼────────────────────────────────────────────┤
│ B = 16 (default)│ ✓ 1 CUDA warp per block (32 threads)    │
│                │ ✓ ≤15 tokens wasted per sequence (~1.5%) │
│                │ ✓ Good memory locality within block       │
│                │ ✓ Reasonable CoW copy overhead            │
├────────────────┼────────────────────────────────────────────┤
│ B = 64         │ ✓ Fewer blocks, simpler management        │
│                │ ✗ Up to 63 tokens wasted per sequence     │
│                │ ✗ CoW copies are expensive                │
└────────────────┴────────────────────────────────────────────┘

Key insight: One CUDA warp (32 threads) processes exactly one
KV block → B=16 maps cleanly to warp-level parallelism
(each thread handles 2 tokens worth of attention computation).
```

## Block Tables: Logical to Physical Mapping

Every active request maintains a **block table** — an array mapping logical block indices to physical block IDs, along with a fill count for the last block:

```
Request A — "Alan Turing is a computer scientist and mathematician"
(20 tokens shown with block_size=4 for illustration)

Logical View (token sequence):
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ LBlock 0 │ LBlock 1 │ LBlock 2 │ LBlock 3 │ LBlock 4 │
│ [tok0-3] │ [tok4-7] │ [tok8-11]│[tok12-15]│[tok16-19]│
└──────────┴──────────┴──────────┴──────────┴──────────┘

Block Table (in-memory array):
┌─────────────────┬───────────────────┬──────────────┐
│ Logical Block # │ Physical Block ID │ Filled Slots │
├─────────────────┼───────────────────┼──────────────┤
│       0         │         7         │      4       │
│       1         │         1         │      4       │
│       2         │         3         │      4       │
│       3         │         2         │      4       │
│       4         │         9         │      4       │
└─────────────────┴───────────────────┴──────────────┘

Physical GPU memory (scattered, non-contiguous):
  Block 7: [tok0 ][tok1 ][tok2 ][tok3 ]
  Block 1: [tok4 ][tok5 ][tok6 ][tok7 ]
  Block 3: [tok8 ][tok9 ][tok10][tok11]
  Block 2: [tok12][tok13][tok14][tok15]
  Block 9: [tok16][tok17][tok18][tok19]
```

### Address Translation Formula

To access the KV data for token `t` in a given request:

```
Address Translation:
─────────────────────────────────────────────────────────────
logical_block  = t // block_size     # which block (integer division)
within_block   = t %  block_size     # position within block

physical_block = block_table[request_id][logical_block]

kv_key = key_cache[physical_block, layer, head, within_block, :]
kv_val = val_cache[physical_block, layer, head, :, within_block]

Physical slot number (used in slot_mapping tensor):
  phys_slot = physical_block × block_size + within_block
─────────────────────────────────────────────────────────────

This indirection adds one table lookup per token access,
but hardware-accelerated scatter/gather makes this fast.
```

## Block-wise Attention Computation

The attention algorithm is modified to operate over non-contiguous KV blocks using block tables:

```
Standard (Contiguous) Attention:
  For query q_i at position i:
    scores[j] = q_i · K[j] / sqrt(d_k)   for j = 0..i
    weights    = softmax(scores[0..i])
    output     = weights · V[0..i]
  (Requires K, V in contiguous memory)

PagedAttention:
  For query q_i at position i, context split into B-token blocks:
  
  running_max = -inf
  running_sum = 0
  running_out = zeros(d_v)
  
  For each block j = 0..num_blocks-1:
    phys_block = block_table[i // B][j]   # translate logical → physical
    K_j = gather(key_cache, phys_block)   # fetch non-contiguous block
    V_j = gather(val_cache, phys_block)
    
    # Score this block's tokens
    a_j  = q_i^T · K_j / sqrt(d_k)       # shape: [B]
    m_j  = max(a_j)                        # block-local maximum
    
    # Online softmax update (numerically stable):
    new_max = max(running_max, m_j)
    running_out = (running_out × exp(running_max - new_max)
                 + V_j · exp(a_j - new_max))
    running_sum = running_sum × exp(running_max - new_max)
                + sum(exp(a_j - new_max))
    running_max = new_max
  
  # Final normalization
  output = running_out / running_sum
```

The **online softmax** (based on Flash attention's algorithm) is essential — it allows accumulating partial results across blocks without ever materializing the full `i × i` attention score matrix.

## CUDA Kernel Design

vLLM implements PagedAttention in two CUDA kernels with different parallelization strategies:

### PagedAttention v1 (Single-Pass Kernel)

```
v1 Kernel Design:
─────────────────────────────────────────────────────────────
Grid:  (num_heads, num_seqs, 1)
  → One thread block per (sequence, head) pair
  → Total thread blocks: batch_size × n_kv_heads

Block: 128 threads = 4 warps

Execution:
  1. Each thread block handles one (seq, head) pair
  2. Iterates over ALL KV blocks in the sequence serially
  3. One warp processes one KV block at a time:
     - Fetch 16 key vectors from block
     - Compute 16 dot products with query
     - Update online softmax running state
     - Fetch 16 value vectors
     - Accumulate weighted sum
  4. Thread block writes final output

Suitable for:
  Short-to-medium sequences (< 512 tokens)
  When num_partitions = 1

Limitation:
  Long sequences create long serial loops in one thread block
  → poor GPU occupancy → compute-bound efficiency loss
─────────────────────────────────────────────────────────────
```

### PagedAttention v2 (Two-Pass Partitioned Kernel)

```
v2 Kernel Design:
─────────────────────────────────────────────────────────────
PARTITION_SIZE = 512 tokens per partition
num_partitions = ceil(seq_len / PARTITION_SIZE)

Phase 1 — Partitioned Attention Kernel:
  Grid:  (num_heads, num_seqs, num_partitions)
    → Each thread block handles one (seq, head, partition) tuple
    → Partitions run IN PARALLEL on the GPU!
  
  Each thread block:
    1. Processes PARTITION_SIZE / block_size KV blocks
    2. Computes partial online softmax over its partition
    3. Stores intermediate results:
         exp_sums[seq, head, partition]        — Σ exp(a - m)
         max_logits[seq, head, partition]      — partition max
         tmp_out[seq, head, partition, head_dim] — partial sum V

Phase 2 — Reduce Kernel:
  Grid:  (num_heads, num_seqs, 1)
    → One thread block per (seq, head) combines all partitions
  
  Stable log-sum-exp reduction:
    global_max = max(max_logits[p] for p in 0..P)
    
    For each partition p:
      rescale_factor = exp(max_logits[p] - global_max)
      weighted_out  += tmp_out[p] × rescale_factor × exp_sums[p]
      total_sum     += exp_sums[p] × rescale_factor
    
    final_output = weighted_out / total_sum
─────────────────────────────────────────────────────────────

When v2 is selected:
  if max_seq_len > ~512 tokens AND num_partitions > 1:
    → use v2 (parallelism across partitions >> overhead of reduce)
  else:
    → use v1 (no overhead of intermediate buffers)

GPU occupancy comparison (OPT-13B, batch=10, seq=2000):
  v1: occupancy ~12% (long serial loop per thread block)
  v2: occupancy ~85% (32 parallel thread blocks per seq × head)
```

### Three Fused CUDA Operations

vLLM uses three custom fused CUDA kernels to minimize kernel launch overhead and memory traffic:

```
Kernel 1 — Fused Reshape + KV Write:
  Operation: Q,K,V projections → split heads → write K,V to cache
  Input:  raw QKV tensor from attention projection
  Output: Q tensor (for attention compute)
          K,V written to correct slots in block cache
  Fusion: Avoids separate reshape + memcpy operations

Kernel 2 — Fused Block Read + Attention:
  Operation: Block table lookup → gather K,V → compute attention
  Input:  Q tensor, block_tables, key_cache, val_cache, slot_mapping
  Output: Attention output tensor
  Fusion: Gather from non-contiguous blocks in same kernel
          as arithmetic → hides memory latency behind compute

Kernel 3 — Fused Block Copy (for CoW):
  Operation: Copy N physical blocks in one kernel launch
  Input:  List of (src_block_id, dst_block_id) pairs
  Output: dst blocks filled with src block data
  Fusion: Batches ALL copy-on-write copies per step
          into ONE kernel launch → amortizes launch overhead
          (Critical: without this, CoW would add N launches/step)
```

## Copy-on-Write (CoW) Mechanism

Copy-on-write allows multiple request sequences to safely share KV blocks:

```
Reference Counting:
─────────────────────────────────────────────────────────────
Each physical block has a reference count:
  ref_count = 1  → exclusively owned by one sequence
  ref_count > 1  → shared (prompt prefix, parallel sampling)
  ref_count = 0  → free (in free_block_queue)
─────────────────────────────────────────────────────────────

Copy-on-Write Protocol:
  When sequence S needs to write token t to block B:
    if block_B.ref_count == 1:
      → write directly (exclusive ownership)
    else:  # ref_count > 1 (shared)
      1. new_block = allocate from free pool
      2. GPU memcpy(block_B → new_block)  ← batched in kernel 3
      3. block_table[S][last] = new_block  ← redirect S to new block
      4. block_B.ref_count -= 1           ← release S's claim on old
      5. new_block.ref_count = 1          ← S now owns new block
      6. write token t's KV to new_block  ← now safe to write

Efficiency:
  All CoW copies within one scheduling step are batched
  into a single CUDA kernel launch (kernel 3 above).
  This amortizes the 10–100µs kernel launch overhead
  regardless of how many blocks need copying.
```

### Copy-on-Write in Parallel Sampling

When a user requests multiple completions from one prompt (`n=5` in sampling params):

```
Parallel Sampling (n=5) Memory Timeline:

Prefill:
  Prompt: "The future of AI is" → 4 prompt blocks allocated
  block_table[seq_A] = [7, 1, 3, 2]  → ref_count[7]=ref_count[1]=...=5
  block_table[seq_B] = [7, 1, 3, 2]  ← ALL share same physical blocks!
  block_table[seq_C] = [7, 1, 3, 2]
  block_table[seq_D] = [7, 1, 3, 2]
  block_table[seq_E] = [7, 1, 3, 2]

Decode step 1 — seq_A generates "bright":
  Last block (block 2) has ref_count = 5 → CoW triggered
  ┌─────────────────────────────────────────────────────────┐
  │  Allocate new block 12                                  │
  │  Copy block 2 → block 12                               │
  │  block_table[seq_A][3] = 12   (seq_A now diverges)     │
  │  block_2.ref_count = 4        (4 sequences still share) │
  │  block_12.ref_count = 1       (seq_A exclusively owns)  │
  │  Write "bright" KV to block 12, slot 0                  │
  └─────────────────────────────────────────────────────────┘

After all 5 sequences diverge:
  seq_A: [7, 1, 3, 12] — unique last block
  seq_B: [7, 1, 3, 15] — unique last block
  seq_C: [7, 1, 3, 18] — unique last block
  seq_D: [7, 1, 3, 21] — unique last block
  seq_E: [7, 1, 3, 24] — unique last block
  
  Blocks 7, 1, 3: still shared (ref_count=5) → zero copy cost
  Memory saved: 3 blocks × 5 sequences = 12 blocks saved
  vs. 4 blocks × 5 sequences = 20 blocks without sharing
  → 60% memory savings for prompt portion
```

### Copy-on-Write in Beam Search

Beam search creates a dynamic tree of sequences where CoW enables efficient memory management:

```
Beam Search (width=4) Memory Evolution:

Step 0: All 4 beams share prompt blocks {0, 1, 3}
  beam_0: [0, 1, 3]   ref_count[0]=ref_count[1]=ref_count[3]=4
  beam_1: [0, 1, 3]
  beam_2: [0, 1, 3]
  beam_3: [0, 1, 3]

Step 3: beam_0 and beam_1 diverge from {0,1,3} shared history
  CoW allocates blocks {9,10} for beam_0's unique extension
  CoW allocates blocks {11,12} for beam_1's unique extension
  beam_0: [0, 1, 3, 9, 10]
  beam_1: [0, 1, 3, 11, 12]
  beam_2: [0, 1, 3]  (still fully shared)
  beam_3: [0, 1, 3]

Step 5: Beam pruning — beam_2 is eliminated
  free(beam_2):
    ref_count[0] -= 1 → still 3 (still referenced by others)
    ref_count[1] -= 1 → still 3
    ref_count[3] -= 1 → still 3
    (No blocks actually freed yet since other beams still use them)

  beam_3 then expands: CoW allocates blocks {13,14}
  When beam_3 eventually finishes:
    if ref_count[0] → 0: free to pool
    if ref_count[1] → 0: free to pool
    ...

Memory savings (SOSP '23, OPT-13B, ShareGPT):
  Parallel sampling: 16–30% KV cache reduction
  Beam search:       44.3%–66.3% KV cache reduction
```

## Three Primitive Sequence Operations

The SOSP paper formalizes three abstract operations on sequences, from which all decoding algorithms can be composed:

```
fork(parent_seq, child_seq):
  child_seq.block_table = copy of parent_seq.block_table
  For each physical block B in parent_seq.block_table:
    B.ref_count += 1
  Effect: child shares all of parent's blocks; no data copied

append(seq, new_token_kv):
  if seq.last_block.filled_slots < block_size:
    # Write to existing last block
    if seq.last_block.ref_count == 1:
      write directly to last slot
    else:
      CoW: copy last block → new block, write to new block
  else:
    # Last block is full: allocate new block
    new_block = free_pool.pop()
    seq.block_table.append(new_block)
    new_block.ref_count = 1
    write token KV to new_block slot 0

free(seq):
  for each physical block B in seq.block_table:
    B.ref_count -= 1
    if B.ref_count == 0:
      free_pool.push(B)
  seq.block_table = []
```

These three primitives implement:
- **Parallel sampling**: `fork(prompt_seq, sample_i)` × n, then `append` each independently
- **Beam search**: `fork(parent_beam, child_beam)` at each expansion step
- **Standard decoding**: just `append` on a single sequence

## Prefix Caching Extension

PagedAttention's block-level abstraction naturally extends to **prefix caching** — reusing KV blocks for common prompt prefixes across different requests:

```
Hash-Based Block Identification:
  block_hash = hash(token_ids_in_block + parent_block_hash)
               ──────────────────────────────────────────
               Chained hash: block identity encodes ALL
               preceding tokens (not just this block's tokens)

  This ensures: same block_hash ↔ identical token history
  (Two requests with identical first 32 tokens share first 2 blocks)

Prefix Cache Lookup During Scheduling:
  For each full block in new request's prompt:
    hash = compute_block_hash(block_token_ids, prefix_hash)
    if hash in prefix_cache:
      # Cache hit: reuse existing physical block
      reuse_physical_block(prefix_cache[hash])
      ref_count[reused_block] += 1
      mark block as "prefilled" (skip in prefill pass)
      prefix_hash = hash  # chain for next block
    else:
      # Cache miss: allocate new block for prefill
      new_block = free_pool.pop()
      prefix_cache[hash] = new_block
      must_prefill.append(new_block)

Eviction Policy (LRU):
  When free_pool is empty and new blocks are needed:
    Find LRU block in prefix_cache with ref_count == 0
    Remove from prefix_cache
    Push to free_pool
    (Block contents overwritten when next request uses it)

V1 improvement over V0:
  V0: O(n) eviction (linear scan for LRU candidate)
  V1: O(1) eviction using doubly-linked list
      (<1% overhead even at 0% cache hit rate)
      Enabled by default in V1
```

## Performance Characteristics

```
Memory Efficiency:
┌─────────────────────────────────────────────────────────────┐
│  System           │ KV Cache Utilization                    │
│  ─────────────────┼─────────────────────────────────────── │
│  FasterTransformer│ 20.4% (reserves max_length contiguous)  │
│  HuggingFace TGI  │ 38.2% (dynamic, but still contiguous)   │
│  vLLM             │ 96.3% (block fragmentation < 1 block/seq)│
└─────────────────────────────────────────────────────────────┘

Attention Kernel Overhead:
  PagedAttention vs. vanilla FlashAttention:
  Kernel latency overhead: +20%–26%
  (Due to block table indirection and scattered memory access)
  
  But system-level throughput is 2×–22× HIGHER
  → Memory efficiency gain vastly outweighs kernel overhead

Internal Fragmentation in vLLM:
  Waste per sequence: at most (block_size - 1) tokens = 15 tokens
  Average sequence length (ShareGPT): ~1000 tokens
  Fragmentation rate: 15/1000 ≈ 1.5% — nearly negligible

External Fragmentation:
  Zero. All blocks are identical size → no fragmentation between
  different-sized requests. Free pool contains interchangeable blocks.
```

## Implementation in vLLM Context

```
System Integration Flow:
  Scheduler.schedule():
    → Checks free_block_queue for available blocks
    → Allocates blocks for new requests
    → Returns blocks_to_copy for CoW operations

  CacheEngine (per Worker):
    → Holds actual GPU tensor: key_cache, val_cache
    → Executes swap_in, swap_out, copy operations on the tensors
    → Physical blocks are indices into these pre-allocated tensors

  ModelRunner.prepare_inputs():
    → Builds slot_mapping: token → physical slot (block × B + offset)
    → Builds block_tables: 2D array [num_seqs × max_blocks_per_seq]
    → These are the inputs to the attention kernel

  PagedAttention CUDA kernel:
    → Reads block_tables to find physical block IDs
    → Gathers KV data from non-contiguous locations
    → Computes attention with online softmax
    → Writes output

Pre-allocated Memory (at startup):
  Worker.initialize_kv_cache():
    available_memory = gpu_total × gpu_memory_utilization
                     - model_weights_memory
                     - activation_peak_memory
    num_blocks = available_memory // block_size_bytes
    
    key_cache = torch.empty(
        [num_blocks, n_kv_heads, head_dim//x, block_size, x],
        dtype=kv_dtype, device=cuda)
    val_cache = torch.empty(
        [num_blocks, n_kv_heads, head_dim, block_size],
        dtype=kv_dtype, device=cuda)
    
    # Blocks are never moved; only block_table indices change
```

---
*Related: [[KV Cache Management]], [[vLLM Architecture]], [[LLM Inference Serving]], [[Continuous Batching]]*
