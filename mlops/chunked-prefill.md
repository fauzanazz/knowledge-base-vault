# Chunked Prefill in vLLM

Chunked prefill is a scheduling technique that splits long prompt processing into fixed-size token chunks, interleaving prefill computation with ongoing decode operations. By preventing long prompts from monopolizing GPU resources, it reduces Time-To-First-Token (TTFT) for queued requests, stabilizes inter-token latency (ITL) for concurrent users, and improves overall system fairness. First introduced in vLLM as an optional feature, chunked prefill became deeply integrated into the unified scheduler architecture of vLLM V1 (January 2025).

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Core Mechanism](#core-mechanism)
3. [Decode Interleaving](#decode-interleaving)
4. [vLLM V1 Unified Scheduler Architecture](#vllm-v1-unified-scheduler-architecture)
5. [Concurrent Partial Prefills](#concurrent-partial-prefills)
6. [Inter-Prefill Budget Control](#inter-prefill-budget-control)
7. [Hybrid Chunked Prefill Mode](#hybrid-chunked-prefill-mode)
8. [Interaction with Prefix Caching](#interaction-with-prefix-caching)
9. [Interaction with Disaggregated Prefill](#interaction-with-disaggregated-prefill)
10. [Performance Characteristics and Benchmarks](#performance-characteristics-and-benchmarks)
11. [TTFT Reduction Analysis](#ttft-reduction-analysis)
12. [Configuration Reference](#configuration-reference)
13. [Trade-offs and Limitations](#trade-offs-and-limitations)

---

## Problem Statement

In standard continuous batching (without chunked prefill), the prefill phase is **atomic**: a request's entire prompt must be processed before any output tokens are generated. This creates two distinct problems:

### Problem 1: Head-of-Line Blocking (Long Prompt → Short Request Starvation)

Consider a server processing a 50,000-token request alongside many shorter requests:

```
Without chunked prefill:
  Step 1: Prefill 50,000 tokens (takes ~500ms on H100)
  Step 2: Decode - all waiting short requests now start
```

Short requests are completely blocked for the entire 500ms prefill of the long prompt, even if their own prefill would take only a few milliseconds. This head-of-line blocking makes TTFT unpredictable and increases p99 latency dramatically.

### Problem 2: ITL Spikes During Large Prefills

Even for the long request itself, if it arrives while other requests are decoding, those decoding requests are stalled during the prefill step. Their inter-token latency (ITL) — the time between consecutive output tokens — spikes massively for the duration of the prefill.

```
Normal ITL (decode-only steps): 10ms per token
During 50K-token prefill: 510ms gap (50ms prefill + 10ms decode)
```

For real-time applications (chatbots, streaming responses), ITL spikes of 10–50× the normal value are unacceptable.

### Root Cause

The fundamental issue is the **prefill-decode coupling**: the prefill phase is compute-heavy (processing many tokens in parallel), while the decode phase is memory-bandwidth-heavy (processing one token per sequence). When combined in continuous batching, a large prefill dominates the step time, effectively pausing all decode operations.

---

## Core Mechanism

Chunked prefill addresses this by splitting the prefill into fixed-size token chunks, each processed within a maximum token budget:

```python
max_num_batched_tokens = 512  # configurable; typical range 512-4096
```

For a 50,000-token prompt:

```
Chunk 1 (Step 1):  tokens[0:512]     → compute KV cache for tokens 0-511
Chunk 2 (Step 2):  tokens[512:1024]  → compute KV cache for tokens 512-1023
...
Chunk N (Step N):  tokens[49664:]    → final chunk + generate first output token
```

Each step processes at most `max_num_batched_tokens` prefill tokens. The KV cache computed in each chunk is retained in memory; subsequent chunks build on this cached state using standard causal attention.

### Token Budget Model

The `max_num_batched_tokens` budget controls how many total tokens (prefill + decode) are processed per step:

```
Step budget = max_num_batched_tokens
             ≥ prefill_tokens_this_step + decode_tokens_this_step
```

Where `decode_tokens_this_step = num_currently_decoding_requests × 1` (one token per request per step).

This budget is the fundamental scheduling unit in vLLM's architecture.

---

## Decode Interleaving

The critical feature that differentiates chunked prefill from simple prefill throttling is **decode interleaving**: each chunked-prefill step is a "hybrid batch" that simultaneously processes:

1. **Prefill tokens**: the current chunk of the long prompt
2. **Decode tokens**: one token per currently-decoding sequence

```
Step composition with chunked prefill:
┌─────────────────────────────────────────────────────┐
│  Prefill chunk: [t₀, t₁, ..., t₅₁₁]  (512 tokens) │
│  Decode tokens: [d₀, d₁, ..., d₃₁]   (32 tokens)   │
│  Total: 544 tokens in one forward pass              │
└─────────────────────────────────────────────────────┘
```

By including decode tokens in every step, chunked prefill ensures that:
- **Decoding sequences are never stalled** — they continue generating at every step
- **ITL remains bounded** — the maximum ITL is `step_time` regardless of prefill length
- **TTFT for queued requests** — is bounded by `(queue_position × chunk_time)` rather than `(sum of all preceding full prefill times)`

### Attention Computation in Hybrid Batches

The attention computation in a hybrid batch must handle two qualitatively different query types:

- **Prefill queries**: attend to all previous tokens (causal mask within the chunk + full KV cache for prior chunks)
- **Decode queries**: each attends to its own full sequence history (KV cache lookup only)

vLLM's PagedAttention implementation handles this natively: prefill queries use a block of Q, K, V tokens from the current chunk, while decode queries use Q from the current token and K, V from the paged KV cache.

---

## vLLM V1 Unified Scheduler Architecture

vLLM V1 takes chunked prefill from a feature to a fundamental architectural principle: **the scheduler no longer distinguishes between "prefill" and "decode" phases**.

### The Unified Token Budget Model

V1's scheduler expresses all scheduling decisions as a single dictionary mapping request IDs to token counts:

```python
# V1 scheduler output
schedule = {
    "req_A": 512,   # prefill request: process 512 tokens this step
    "req_B": 1,     # decode request: generate 1 token this step
    "req_C": 256,   # partially-prefilled request: process next 256 tokens
    "req_D": 4,     # speculative decode: 4 candidate tokens
}
```

This unified representation naturally handles all scheduling scenarios:
- **Chunked prefill**: allocate `chunk_size` tokens to a prefilling request
- **Standard decode**: allocate 1 token per decoding request
- **Speculative decode**: allocate `num_speculative_tokens` to decode requests
- **Mixed batches**: arbitrary combinations of the above

### Persistent Batch and Incremental Updates

V1's scheduler maintains a "Persistent Batch" — the current set of active requests and their token allocations. Rather than reconstructing the full batch tensor from scratch each step, it applies incremental updates:

```
Step N batch: {req_A: 512 prefill tokens}
Step N+1 batch: {req_A: 512 more prefill tokens, req_B: 1 decode token, ...}
```

Only the changes (diffs) are recomputed, reducing CPU overhead from batch preparation.

### FIFO Ordering Within Chunked Prefill

Despite chunking, V1's scheduler maintains FIFO ordering: requests complete their prefill phase before entering the "running" pool of decoding requests. This ensures that a request's response ordering is consistent with its arrival order, preventing starvation of requests at the front of the queue.

---

## Concurrent Partial Prefills

For workloads where multiple medium-length prompts arrive simultaneously, chunked prefill alone doesn't help if all chunks are sequenced behind one another. PR #10235 (V0) added support for **concurrent partial prefills**:

### Configuration Parameters

```bash
--max-num-partial-prefills=N     # Allow up to N requests in simultaneous chunked prefill
--max-long-partial-prefills=N    # Limit concurrent "very long" prompt chunked prefills
--long-prefill-threshold=X%      # Classify prompts longer than X% of context window as "long"
                                 # Default: 4% of context window
```

### Effect

Without concurrent partial prefills (V0 default):
```
Step 1: chunk 1 of request A (alone)
Step 2: chunk 2 of request A (alone)
Step 3: chunk 1 of request B (alone)  ← B waited for A to complete
```

With `--max-num-partial-prefills=2`:
```
Step 1: chunk 1 of A + chunk 1 of B (simultaneously)
Step 2: chunk 2 of A + chunk 2 of B (simultaneously)
Step 3: both begin decoding
```

**Measured impact:**
- p90 TTFT cut in half for medium-length requests
- Up to 30× improvement in TTFT for large-prompt scenarios with high concurrency

### Long Prompt Fairness

The `--long-prefill-threshold` parameter prevents a single extremely long prompt from saturating the concurrent prefill budget, which would starve medium-length requests. By limiting the number of "very long" prompts in concurrent prefill, short and medium requests maintain responsive TTFT even when the server is processing large documents.

---

## Inter-Prefill Budget Control

PR #33743 (2026) introduced a more sophisticated control for prefill batching: **inter-prefill budget**.

### Problem: Prefill Saturation

When the GPU is already fully loaded by one prefill request (e.g., a 1024-token chunk saturates compute capacity), adding a second prefill request to the same batch **does not improve throughput** (the GPU is already at capacity) but **degrades TTFT for both requests** by competing for GPU resources.

### The `--inter-prefill-budget` Parameter

```bash
vllm serve <model> --inter-prefill-budget 1024
```

This parameter prevents co-scheduling of multiple prefill requests when the token budget is already saturated by a single request. The scheduler will not start prefilling a second request if the current step's prefill tokens already meet or exceed the inter-prefill budget.

**Measured impact on Gemma-3-27B:**
- Median TTFT reduction: **up to 37%** compared to uncontrolled prefill batching
- P99 TTFT improvement: significant reduction in worst-case latency

### Relationship to `max_num_batched_tokens`

| Parameter | Controls |
|-----------|---------|
| `max_num_batched_tokens` | Total tokens (prefill + decode) per step |
| `inter-prefill-budget` | Maximum prefill tokens before blocking new prefill starts |

Together, these two parameters give fine-grained control over the balance between:
- Saturating GPU compute on prefill (high throughput)
- Minimizing TTFT for queued requests (low latency)

---

## Hybrid Chunked Prefill Mode

PR #26625 introduced a **Hybrid** mode that dynamically enables or disables chunking based on the current server state:

### Logic

```
Active decode requests present? 
  YES → use chunked prefill (limit prefill chunk size)
  NO  → use full prefill (process entire prompt in one pass)
```

When no decode requests are active (e.g., at low load or at system startup), the server is in "prefill-only" mode. In this case, chunking adds overhead (multiple GPU launches, KV cache write overhead) without providing any benefit (there are no decode requests to keep alive). Hybrid mode detects this and bypasses chunking.

### Performance Impact

| Scenario | Without Hybrid | With Hybrid |
|----------|---------------|-------------|
| Low concurrency, no decodes active | Full prefill (good) | Full prefill (good) |
| Low concurrency, 1-2 decodes active | Chunked prefill (moderate overhead) | Switches to chunked (correct behavior) |
| High concurrency, many decodes | Chunked prefill (good) | Chunked prefill (good) |

**Measured improvements with Hybrid mode:**
- 10–20% lower TTFT at low concurrency
- 2–5% higher total token throughput (from eliminated unnecessary chunking overhead)

---

## Interaction with Prefix Caching

Chunked prefill and Automatic Prefix Caching (APC) interact synergistically:

### Cache-Aware Chunk Processing

Before processing each chunk, the scheduler checks the APC hash table for matching KV blocks within that chunk's token range. If a chunk's tokens are already cached:
- The chunk step is **skipped** (no GPU compute required)
- KV blocks are retrieved directly from the cache
- The scheduler proceeds to the next chunk

This means that for a request where 80% of the prompt is cached (e.g., long system prompt), chunked prefill processes only the remaining 20% of non-cached tokens, with the cached chunks passing through nearly instantaneously.

### Practical Effect

For a 10,000-token prompt where the first 8,000 tokens are cached:
```
Without APC: process all 10,000 tokens in chunks (20 steps at 512 tokens/step)
With APC:    8,000-token prefix served from cache; only process 2,000 new tokens (4 steps)
```

The combination of APC + chunked prefill provides **multiplicative latency reduction**: APC reduces the effective prompt length, and chunked prefill ensures remaining tokens don't block other requests.

---

## Interaction with Disaggregated Prefill

Disaggregated prefill (PD disaggregation) separates prefill-specialized instances from decode-specialized instances. Chunked prefill plays a key role in the PD architecture:

### KV Transfer Enablement

In a disaggregated setup, completed KV blocks must be transferred from the prefill (P) instance to the decode (D) instance via NCCL or RDMA. With chunked prefill, KV blocks can be transferred **incrementally** as each chunk completes:

```
P instance:                          D instance:
  Chunk 1 complete → transfer KV₁ →   Receives KV₁
  Chunk 2 complete → transfer KV₂ →   Receives KV₂
  ...
  Final chunk → transfer KVₙ → KV complete → begin decode
```

This pipeline overlaps KV transfer with continued prefill computation on the P instance, reducing end-to-end latency compared to waiting for full-prefill completion before any transfer begins.

---

## Performance Characteristics and Benchmarks

### Latency Impact by Scenario

| Scenario | Without Chunked Prefill | With Chunked Prefill | Improvement |
|----------|------------------------|---------------------|-------------|
| Single long prompt (no other requests) | Full prefill (baseline) | Chunked prefill (slight overhead) | -5% to 0% (neutral to slight regression) |
| Long prompt + 10 concurrent decoding requests | Long prompt delays decodes by full prefill time | Decodes continue uninterrupted | Decode ITL: near-zero regression |
| Short requests queued behind long prompt | Blocked for full prefill duration | Service at next chunk boundary | TTFT: several to 30× improvement |
| Many medium prompts simultaneously | Sequential prefill | Concurrent chunked prefill | p90 TTFT: ~2× improvement |

### Throughput Trade-offs

Chunked prefill slightly reduces raw prefill throughput due to:
- Multiple GPU kernel launches per prompt (instead of one)
- Overhead of including decode tokens in each step (slightly larger batch)
- KV cache book-keeping for incremental writes

In practice, these overheads are minimal (< 5%) and are more than compensated by the improved latency distribution and higher effective GPU utilization from interleaved decoding.

---

## TTFT Reduction Analysis

### Mathematical Model

Without chunked prefill, TTFT for request arriving at position P in a queue:
```
TTFT_cold = sum(prefill_time[0..P-1]) + prefill_time[P]
```

With chunked prefill (chunk size C, N tokens per preceding request):
```
TTFT_chunked = P × step_time + N/C × step_time + step_time
             ≈ (P + N/C + 1) × step_time
```

For N=50,000 tokens and C=512:
- `N/C = 97.6` steps for the long request to complete
- Queued request waits at most `(P + 1) × step_time` ≈ one additional step after it reaches the front of the queue

This is a dramatic improvement: the queued request no longer waits for the entire 50,000-token prefill but only for its own chunk allocation.

---

## Configuration Reference

### Basic Configuration

```bash
# Enable chunked prefill with custom chunk size
vllm serve meta-llama/Meta-Llama-3.1-8B-Instruct \
    --enable-chunked-prefill \
    --max-num-batched-tokens 512

# Conservative: small chunks for maximum latency fairness
vllm serve <model> \
    --enable-chunked-prefill \
    --max-num-batched-tokens 256

# Aggressive: large chunks for maximum throughput
vllm serve <model> \
    --enable-chunked-prefill \
    --max-num-batched-tokens 4096
```

### Advanced Multi-Prefill Configuration

```bash
vllm serve <model> \
    --enable-chunked-prefill \
    --max-num-batched-tokens 1024 \
    --max-num-partial-prefills 4 \
    --max-long-partial-prefills 1 \
    --long-prefill-threshold 0.04 \
    --inter-prefill-budget 1024
```

### Python API

```python
from vllm import LLM, EngineArgs

llm = LLM(
    model="meta-llama/Meta-Llama-3.1-8B-Instruct",
    enable_chunked_prefill=True,
    max_num_batched_tokens=512,
)
```

### Chunk Size Guidelines

| Workload | Recommended `max_num_batched_tokens` |
|----------|-------------------------------------|
| Latency-critical (chatbot) | 256–512 |
| Balanced (general serving) | 512–2048 |
| Throughput-optimized (batch processing) | 2048–8192 |
| Very long context (100K+) | 4096–8192 |

---

## Trade-offs and Limitations

### When Chunked Prefill Hurts Throughput

For workloads with no concurrent decode traffic (e.g., pure batch processing of long documents), chunked prefill adds unnecessary overhead:
- Multiple GPU launches instead of one
- Slightly higher memory for tracking chunk state
- No decode requests benefit from the fairness

**Recommendation:** Use Hybrid mode (`--chunked-prefill-mode hybrid`) which automatically disables chunking when no decode requests are active.

### Memory Overhead

Each partially-completed prefill request holds its accumulated KV cache blocks in GPU memory until prefill completes. With many concurrent partial prefills, this increases minimum GPU memory requirements:

```
Memory for partial prefills ≈ max_num_partial_prefills × average_partial_cache_size
```

This must be accounted for when setting `--gpu-memory-utilization`.

### Interaction with CUDA Graphs

CUDA graphs capture static execution graphs for fixed-shape computations. Chunked prefill creates **variable batch compositions** (different number of prefill + decode tokens each step), which requires careful graph capture:

- Pure decode steps: eligible for full CUDA graph capture
- Mixed prefill+decode steps: use piecewise graphs or eager mode
- V1 with `torch.compile`: handles this automatically via `FULL_AND_PIECEWISE` mode

### Attention Complexity

Each chunked prefill step must maintain correct causal attention across chunks. For a prompt processed in N chunks, chunk K must attend to the KV cache of chunks 0 through K-1 (already cached) plus its own Q, K, V tensors. This is handled correctly by vLLM's PagedAttention implementation, but requires that KV blocks from earlier chunks remain in GPU memory throughout the prefill process.
