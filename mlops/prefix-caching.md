# Automatic Prefix Caching (APC) in vLLM

Automatic Prefix Caching (APC) is vLLM's mechanism for detecting, storing, and reusing KV (Key-Value) cache blocks that correspond to repeated token prefixes across different requests. By avoiding redundant computation for shared prompt segments—such as long system prompts, few-shot examples, or multi-turn conversation history—APC dramatically reduces Time-To-First-Token (TTFT) and increases overall serving throughput. First merged in March 2024 (RFC #2614), APC was redesigned in vLLM V1 (January 2025) to operate with near-zero overhead and is now **enabled by default** in all V1 deployments.

---

## Table of Contents

1. [Motivation and Core Problem](#motivation-and-core-problem)
2. [Hash-Based Block Matching](#hash-based-block-matching)
3. [Per-Block Metadata](#per-block-metadata)
4. [Data Structure: Block Table Indirection](#data-structure-block-table-indirection)
5. [Eviction Policy (LRU with Enhancements)](#eviction-policy-lru-with-enhancements)
6. [V1 Architectural Improvements](#v1-architectural-improvements)
7. [Scheduling-Aware Caching](#scheduling-aware-caching)
8. [Multimodal Prefix Caching](#multimodal-prefix-caching)
9. [Use Cases and Workload Patterns](#use-cases-and-workload-patterns)
10. [Performance Characteristics](#performance-characteristics)
11. [Configuration and Enablement](#configuration-and-enablement)
12. [Interaction with Other Features](#interaction-with-other-features)
13. [Limitations and Trade-offs](#limitations-and-trade-offs)

---

## Motivation and Core Problem

In large language model (LLM) inference, the **prefill phase** is responsible for processing the input prompt and populating the KV cache. For each input token, the attention mechanism must compute key and value tensors for all previous tokens — making prefill cost proportional to `O(n²)` in sequence length. When many requests share identical prompt prefixes (e.g., all users send the same 2,000-token system prompt), the server recomputes the same KV tensors repeatedly, wasting both compute and time.

Without APC, every new request starts from a cold cache, regardless of how similar it is to prior requests. With APC, the server recognizes that a request's prefix has already been computed, retrieves the cached KV blocks, and begins generation immediately from the first non-cached token. For a 2,000-token system prompt where the KV blocks are fully cached, TTFT can be reduced by several times compared to cold-cache computation.

---

## Hash-Based Block Matching

The core insight of vLLM's APC design is that **every complete KV cache block can be uniquely identified by its content** using a deterministic hash. The hash is computed recursively across blocks:

```
block_hash_0 = hash(0, tokens_in_block_0)          # base case
block_hash_n = hash(block_hash_{n-1}, tokens_in_block_n)
```

More precisely, each block's hash is a function of:
- The hash of the immediately preceding block (which itself encodes all blocks before it)
- The token IDs contained within the current block

This recursive chaining means **the hash of block N implicitly encodes the complete token history from the start of the sequence through the end of block N**. Two blocks with identical hashes are guaranteed (absent hash collision) to have arisen from identical token prefixes.

### Key Properties

| Property | Detail |
|----------|--------|
| **Uniqueness** | Hash encodes entire prefix history, not just local block content |
| **Determinism** | Same tokens always produce the same hash, enabling cross-request reuse |
| **Composability** | Prefix sharing works across arbitrary-length shared segments |
| **Collision resistance** | Uses strong hash functions; collisions are computationally negligible in practice |

### Only Complete Blocks Are Hashed

An important constraint: **only fully-filled blocks are inserted into the hash table**. The trailing partial block at the end of a prompt is held in a separate structure and promoted to the hash table only when it becomes complete (filled to the block size, typically 16 tokens). This simplifies the design and avoids the complexity of partial-block matching, at the minor cost of not caching the last few tokens of an incomplete final block.

### Block Size Trade-offs

The block size (configurable, default 16 tokens) determines caching granularity:

- **Smaller blocks** → finer granularity, more hash lookups, better partial-prefix reuse
- **Larger blocks** → fewer lookups, less overhead, but coarser reuse (requires a longer shared prefix before any caching benefit)

For most production deployments with long system prompts, the default 16-token block size is a good balance.

---

## Per-Block Metadata

Each KV cache block participating in APC stores the following metadata alongside its tensor data:

| Field | Type | Purpose |
|-------|------|---------|
| `block_hash` | `int` | Content-based identifier; used for O(1) lookup in the hash table |
| `ref_count` | `int` | Number of active sequences currently pointing to this physical block |
| `last_accessed_time` | `float` | Timestamp of most recent access; used by LRU eviction policy |
| `total_access_count` | `int` | Cumulative number of times the block has been accessed; used for eviction tiebreaking |
| `prefix_length` | `int` | Number of tokens from sequence start through the end of this block; identifies "leaf" vs. "ancestor" blocks in the implicit prefix tree |

The `prefix_length` field is particularly important for the eviction policy: it allows the system to distinguish between blocks near the root of the prefix tree (short `prefix_length`, shared by many requests) and leaf blocks (long `prefix_length`, shared by fewer requests). Evicting root blocks destroys caching benefit for many future requests, while evicting leaf blocks has a more localized impact.

---

## Data Structure: Block Table Indirection

vLLM's APC implementation introduces an **indirection layer** between logical and physical block addressing:

```
Request sequence → Logical block table → Hash table lookup → Physical block address
```

All prefix sharing is achieved by multiple logical blocks pointing to the **same physical block** (identified by hash match). This is simpler than a full radix tree (used by SGLang's RadixAttention), but achieves equivalent behavior for the common case of full-block prefix sharing.

### Comparison with Radix Tree Approaches

SGLang's RadixAttention uses an explicit trie/radix tree where each node represents a shared prefix segment, with fine-grained prefix sharing at arbitrary token boundaries. vLLM's hash-based approach achieves the same asymptotic benefit (O(1) lookup per block) without the overhead of maintaining an explicit tree structure, at the cost of requiring block-granular (not token-granular) alignment.

| Aspect | vLLM Hash Table | SGLang Radix Tree |
|--------|----------------|-------------------|
| **Lookup complexity** | O(1) per block | O(prefix_length) |
| **Granularity** | Block-level (16 tokens) | Token-level |
| **Implementation complexity** | Low | Higher |
| **Partial prefix reuse** | Block-boundary only | Arbitrary |

---

## Eviction Policy (LRU with Enhancements)

When GPU KV cache memory is exhausted and space must be reclaimed, vLLM applies a three-tier eviction policy:

### Tier 1: Reference Count Gate

Only blocks with `ref_count == 0` are eligible for eviction. Blocks actively used by running requests (ref_count ≥ 1) are never evicted. This ensures correctness — evicting an in-use block would corrupt ongoing request computation.

### Tier 2: LRU Ordering

Among zero-reference blocks, the **Least Recently Used** block is selected for eviction first. The `last_accessed_time` field provides the ordering key. LRU is a well-understood, cache-optimal policy for workloads with temporal locality (recently used items are more likely to be used again).

### Tier 3: Prefix-Length Tiebreaker

When multiple zero-reference blocks have identical or similar last-access timestamps, the block with the **longest prefix_length** (i.e., the leaf node) is evicted first. This preserves "ancestor" blocks that correspond to widely-shared prefixes while evicting "leaf" blocks that correspond to more specific, less-reused contexts.

This three-tier policy mirrors the behavior of RadixAttention's eviction strategy and empirically produces high cache hit rates for typical serving workloads.

### Eviction Triggering

Eviction is triggered lazily — only when a new request cannot find sufficient free blocks and the cache is at capacity. The eviction loop removes blocks (starting from least-recently-used leaves) until enough space is freed for the incoming request.

---

## V1 Architectural Improvements

vLLM V1 (released January 2025) completely overhauled the APC implementation with several critical improvements:

### O(1) Cache Eviction

V0's eviction algorithm performed O(N) scans over all cached blocks to find the LRU candidate. At high cache occupancy (thousands of blocks), this scan created non-trivial CPU overhead. V1 restructured the metadata into sorted data structures enabling **constant-time O(1) eviction**, eliminating the scan entirely.

### Minimized Python Object Creation

V0 created Python objects for each block operation (lookup, insert, evict), contributing to garbage collection pressure at high throughput. V1 minimizes transient object creation by using more efficient internal representations (closer to NumPy arrays and integer buffers).

### Near-Zero Overhead at 0% Hit Rate

The combined effect of O(1) eviction and reduced Python overhead means that even when APC provides **zero benefit** (e.g., all requests are completely unique), V1's APC imposes less than 1% throughput degradation. This was sufficient justification to **enable APC by default in V1**, removing the opt-in requirement that existed in V0.

### Persistent Batch Integration

V1's scheduler uses a "Persistent Batch" pattern where input tensors are updated incrementally (diffs only) rather than reconstructed from scratch each step. APC integrates with this pattern: cache lookups and updates are performed as part of the incremental batch update, not as a separate pre-processing step.

---

## Scheduling-Aware Caching

An important limitation of early APC (V0) was that the scheduler did not account for cache hit ratios when making scheduling decisions. A request with 100% cache hits was treated identically to a completely cold request, even though the hot request requires far fewer GPU compute cycles (only non-cached tokens need real computation).

**vLLM v0.6.5** addressed this with an updated scheduling mechanism that **prioritizes cache-warm requests** over cold requests when the GPU is resource-constrained. By preferring requests whose prefixes are already in cache, the scheduler maximizes effective throughput and minimizes KV cache thrashing.

This optimization is especially impactful in mixed-workload scenarios where some requests share popular system prompts (highly cache-warm) while others are unique (cold), competing for the same batch slots.

---

## Multimodal Prefix Caching

In V1, prefix caching extends beyond token sequences to **multimodal inputs** including images, video frames, and audio. The block hash computation is augmented to include a content hash of any embedded media:

```
block_hash = hash(token_ids_hash, image_content_hash)
```

For multi-turn conversations involving the same image (e.g., a user asking multiple questions about an uploaded photo), this means:

1. On the first request, the vision encoder processes the image, generating potentially thousands of embedding vectors
2. These embeddings are stored in KV cache blocks with a hash that incorporates the image content hash
3. On subsequent requests with the same image, the vision encoder is **not re-invoked** — the cached KV blocks are reused directly

For large vision models where image encoding is a substantial fraction of TTFT, multimodal prefix caching can provide proportionally larger speedups than text-only caching.

### Encoder Cache

V1 also introduces a dedicated **encoder cache** that stores raw vision encoder outputs (embeddings) on GPU, distinct from the KV cache itself. This allows the vision embeddings to be reused across chunks during chunked prefill — the encoder runs once per unique image, and the outputs are reused when that image appears in any chunk.

---

## Use Cases and Workload Patterns

APC provides the most benefit in workloads with high token repetition across requests. The key use cases include:

### 1. Long System Prompts

Many production deployments have extensive system prompts encoding instructions, persona, safety guidelines, or domain knowledge — often 500–5,000 tokens. When all users of a service share the same system prompt, APC can cache the KV blocks for that prompt once and reuse them for every subsequent request.

**Example impact:** A 2,000-token system prompt at 16 tokens/block occupies 125 blocks. Without APC, each request processes all 125 blocks. With APC at 100% hit rate, these 125 blocks are retrieved from cache, and only the user's actual question tokens require new computation.

### 2. Few-Shot Learning Examples

Prompts with multiple in-context examples (few-shot prompting) often include 1,000–10,000 tokens of demonstrations that are identical across all requests. APC caches these shared demonstration blocks, making few-shot serving economically competitive with zero-shot (which requires no demonstration tokens at all).

### 3. Retrieval-Augmented Generation (RAG)

RAG pipelines inject retrieved document chunks into the prompt context. When multiple users query the same popular documents, those document chunks appear repeatedly across requests and can be cached. For RAG systems with a fixed document corpus, APC hit rates can be very high for popular documents.

**Note:** The effectiveness depends on prompt template consistency — the retrieved chunks must appear at identical token offsets across requests for the block hashes to match. RAG pipelines that consistently place system prompt → retrieved chunks → user query (in that order) benefit most.

### 4. Multi-Turn Conversations

In conversational applications, each turn appends the new message to the accumulated conversation history. The previous turns are identical across the current request and any future requests in the same conversation session. APC naturally caches growing conversation histories, with each turn adding only a small number of new blocks to the cache.

**Caveat:** This requires the full conversation to be re-submitted in the prompt (as is standard for stateless LLM APIs). The blocks from previous turns will be in cache if the server hasn't evicted them. Cache eviction risk increases with session length and under high server load.

### 5. Document Analysis / Code Review

Analyzing a large document or codebase across multiple queries (e.g., "summarize chapter 3", "what are the key findings", "extract all dates") involves the same large document context across many requests. APC caches the document's KV blocks once, making repeated querying nearly free in terms of prefill compute.

---

## Performance Characteristics

| Scenario | APC Impact |
|----------|-----------|
| **High cache-hit rate** (shared system prompts) | Several-times improvement in TTFT; proportionally higher throughput |
| **Zero cache-hit rate** (completely unique prompts) | < 1% throughput overhead (V1); negligible |
| **Partial cache-hit** (e.g., shared prefix + unique suffix) | Proportional benefit: only non-cached suffix tokens require compute |
| **Multi-turn conversation (growing history)** | TTFT stays roughly constant as history grows, since history is cached |
| **Multimodal (same image, multiple queries)** | Vision encoder cost eliminated after first request |

### Cache Hit Rate Estimation

Cache hit rate depends on:
- **Shared prefix length**: Longer shared prefixes → more cacheable blocks
- **Request diversity**: Fewer unique prompts → higher hit rate
- **Cache size**: More GPU memory allocated to KV cache → lower eviction rate
- **Temporal locality**: Requests sharing prefixes should arrive close together in time

Prometheus metrics exposed by vLLM include `vllm:gpu_cache_usage_perc` and `vllm:cache_config_info`, which can be used to monitor cache occupancy and hit rates in production.

---

## Configuration and Enablement

### vLLM V1 (Default On)

```bash
# APC is enabled by default in V1; no special flag required
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-prefix-caching
```

### Disabling APC

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --no-enable-prefix-caching
```

### Python API

```python
from vllm import LLM

llm = LLM(
    model="meta-llama/Llama-3.1-8B-Instruct",
    enable_prefix_caching=True,  # V1 default; explicit for clarity
)
```

### Block Size Tuning

```bash
vllm serve <model> --block-size 32  # Larger blocks: fewer lookups, coarser reuse
vllm serve <model> --block-size 8   # Smaller blocks: finer reuse, more lookup overhead
```

---

## Interaction with Other Features

### With Chunked Prefill

Chunked prefill processes long prompts in fixed-size token chunks. APC interacts naturally: before processing each chunk, the scheduler checks whether the tokens in that chunk's range are already cached. Cached chunks are skipped (KV blocks directly reused); non-cached chunks are computed and their resulting KV blocks added to the cache.

### With LoRA Adapters

LoRA adapters change the effective model weights, meaning KV values computed with adapter A cannot be reused for a request using adapter B. vLLM handles this by **prepending the LoRA adapter ID to the block hash**:

```
block_hash = hash(lora_id, prefix_hash, block_tokens)
```

This ensures LoRA-specific caching: requests using the same adapter share a cache, while different adapters maintain separate caches.

### With Speculative Decoding

APC applies during the prefill phase, which runs before speculative decoding begins (speculation operates only during the decode phase). The two features are complementary: APC reduces TTFT by caching prefill KV blocks, while speculative decoding reduces time-per-output-token (TPOT) during generation.

### With Tensor Parallelism

In tensor-parallel deployments, each rank holds a portion of the KV cache. APC is applied independently per rank — all ranks receive the same block-matching decisions from the scheduler (which runs on rank 0 or a dedicated process), ensuring consistent cache reuse across the distributed KV cache.

---

## Limitations and Trade-offs

### Block-Granular Reuse Only

APC requires shared prefixes to align on block boundaries. If two requests share the first 20 tokens but the block size is 16, only the first block (16 tokens) is cached; the partial second block (4 shared + variable suffix tokens) does not match and must be recomputed. This limitation primarily affects very short shared prefixes.

### Cache Contention Under High Load

Under heavy load with many distinct prefixes, cache eviction increases and hit rates degrade. The eviction policy is locally optimal (LRU) but cannot predict future request patterns. Production deployments may benefit from monitoring cache hit rates and adjusting GPU memory allocation accordingly.

### No Cross-Instance Sharing (Default)

By default, each vLLM instance maintains its own KV cache independently. Two instances serving the same model cannot share cached blocks. LMCache integration (external KV store) addresses this for multi-instance deployments at the cost of additional infrastructure.

### Correctness of Hash Matching

The system assumes that identical token IDs always produce identical KV tensors, which holds for deterministic models and fixed precision. Quantization format changes or model weight updates invalidate cached blocks (though vLLM restarts clear the cache, preventing stale data from being served).
