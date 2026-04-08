# LLM Inference Serving

Large Language Model (LLM) inference serving is the process of deploying trained language models to generate text responses for user requests at production scale. Unlike training, inference must deliver low-latency, high-throughput responses under highly variable request patterns, while managing the unique computational challenges posed by autoregressive text generation and the associated key-value (KV) cache memory demands.

## Autoregressive Generation

Modern LLMs generate text one token at a time in a sequential, **autoregressive** loop. Each new token is sampled from a probability distribution conditioned on all previously generated tokens:

```
token_{t+1} = sample( p(· | token_0, token_1, ..., token_t) )
```

This process is inherently sequential: token `t+1` cannot be computed until token `t` is known. The loop continues until a stop condition is met — either an end-of-sequence (EOS) token is generated, a maximum token limit is reached, or a user-defined stop string appears.

```
Autoregressive Decoding Loop:
                                        ┌──────────────────┐
Prompt tokens ────────────────────────→ │                  │
 [The, quick, brown, fox]               │  LLM Forward     │
                                        │  Pass            │─→ token_5: "jumps"
                              ┌────────→│                  │
                              │         └──────────────────┘
                              │                  │
                              │          token_5: "jumps"
                              │                  │
                              │         ┌────────▼─────────┐
                              │         │  LLM Forward     │
                              │         │  Pass            │─→ token_6: "over"
                              └────────→│                  │
                                        └──────────────────┘
                                                 │
                                          (continues until EOS)
```

A typical response of 256 tokens therefore requires 256 sequential forward passes through the model.

## The Two Phases of Inference

LLM inference is divided into two fundamentally different computational phases with distinct characteristics:

### Phase 1: Prefill (Prompt Processing)

During prefill, the model processes all prompt tokens simultaneously in a single forward pass:

```
┌─────────────────────────────────────────────────────────────┐
│  PREFILL PHASE                                              │
│                                                             │
│  Input:  [tok_0, tok_1, tok_2, ..., tok_{N-1}]             │
│          (all N prompt tokens processed in parallel)        │
│                                                             │
│  Compute: O(N²) — full self-attention over all N tokens    │
│                                                             │
│  Output:                                                    │
│    1. KV cache entries for all N prompt tokens             │
│    2. The first generated output token (token_N)           │
│                                                             │
│  GPU Utilization: HIGH                                      │
│  Bottleneck: Compute (matrix multiplications)              │
└─────────────────────────────────────────────────────────────┘
```

Prefill is **compute-bound**: the GPU arithmetic units are fully occupied computing attention scores and feed-forward network (FFN) activations across all N tokens. GPU utilization is high, similar to training.

### Phase 2: Decode (Token Generation)

After prefill, the model generates tokens one at a time. Each decode step processes only a single new token but must attend over the entire context:

```
┌─────────────────────────────────────────────────────────────┐
│  DECODE PHASE (each step)                                   │
│                                                             │
│  Input:  [tok_{N+t}]  — a single token                     │
│                                                             │
│  Compute: O(T) — attention query over growing KV cache     │
│           where T = current total context length           │
│                                                             │
│  Output:  next token tok_{N+t+1}                           │
│           + new KV entry written to cache                  │
│                                                             │
│  GPU Utilization: LOW                                       │
│  Bottleneck: Memory bandwidth (reading KV cache)           │
└─────────────────────────────────────────────────────────────┘
```

Decode is **memory-bandwidth-bound**: for each step, the GPU must read the entire KV cache from HBM (High Bandwidth Memory), but only performs a tiny amount of arithmetic per byte read. The arithmetic intensity (FLOPs/byte) is far below the GPU's compute-to-bandwidth ratio (the "roofline").

### Compute vs. Memory Bound Analysis

```
Arithmetic Intensity Comparison:

  Compute-bound threshold (A100 80GB):
    Peak FLOPS: 312 TFLOPS (FP16)
    Memory BW:  2,000 GB/s
    Roofline:   312 / 2 ≈ 156 FLOPs/byte

  Prefill (large batch, N=2048 tokens):
    Attention: ~2×N²×d_model FLOPs ≈ high intensity
    Status: COMPUTE-BOUND ✓

  Decode (single token, T=1000 context):
    Must read: all KV tensors (K,V for all T previous tokens)
    Arithmetic: minimal (one query × K,V matrices)
    Intensity: ~1-10 FLOPs/byte → well below roofline
    Status: MEMORY-BOUND ✗ (GPU underutilized)

  Rule of thumb:
    Larger batch sizes during decode push toward compute-bound
    → batching many decode requests together is essential
```

## The KV Cache

To avoid recomputing attention keys and values on every decode step, transformers maintain a **Key-Value (KV) cache** — a memory store of the intermediate attention states for all previously processed tokens.

### Why KV Caching Is Necessary

Without KV caching, each decode step would need to recompute keys and values for every previous token, making inference O(T²) total FLOPs for T generated tokens. With caching, each step is O(T) — the new token's Q, K, V projections are computed, the new K and V are stored in the cache, and attention is computed against the cached K and V.

```
Without KV Cache (naïve):
  Step t: recompute K,V for tokens 0..t → O(t²) total work
  Overall: O(T³) — prohibitively expensive

With KV Cache:
  Step t: read cached K,V for 0..t-1, compute new K,V for t
  Each step: O(T) memory reads + O(d) compute
  Overall: O(T²) memory + O(T·d) compute — manageable
```

### KV Cache Memory Footprint

The KV cache is the dominant memory consumer in inference, second only to model weights:

```
KV Cache Size Formula:
─────────────────────────────────────────────────────────────
KV_bytes = 2 × n_layers × seq_len × n_kv_heads × head_dim
           × bytes_per_element

Where:
  2           = key tensor + value tensor
  n_layers    = transformer depth
  seq_len     = total context length (prompt + generated)
  n_kv_heads  = number of KV attention heads (may differ from Q
                heads in Grouped Query Attention)
  head_dim    = attention head dimensionality
  bytes_per_element = 2 for FP16/BF16

Example: OPT-13B at FP16, max context 2048 tokens
  = 2 × 40 × 2048 × 40 × 128 × 2
  = 1,677,721,600 bytes ≈ 1.6 GB per request

  800 KB per token — every token generated costs 800 KB!
─────────────────────────────────────────────────────────────

GPU Memory Budget (OPT-13B, A100 40GB):
┌──────────────────────────────────────────────────────────┐
│  Total GPU Memory:          40 GB                        │
│  ├── Model Weights:    ~26 GB  (65%)  — fixed            │
│  ├── KV Cache:         ~12 GB  (30%)  — dynamic          │
│  └── Activations:       ~2 GB   (5%)  — ephemeral        │
└──────────────────────────────────────────────────────────┘

At 1.6 GB/request max, the system can serve at most ~7-8
requests simultaneously if each uses its full context window.
```

### Grouped Query Attention (GQA) and KV Cache Reduction

Modern LLMs use Grouped Query Attention (GQA) or Multi-Query Attention (MQA) to reduce KV cache size:

```
Multi-Head Attention (MHA):
  n_q_heads == n_kv_heads  (e.g., 32 Q heads, 32 KV heads)
  KV cache: full size

Grouped Query Attention (GQA):
  n_q_heads > n_kv_heads  (e.g., 32 Q heads, 8 KV heads)
  Each KV head shared by n_q_heads/n_kv_heads query heads
  KV cache: reduced by factor of n_q_heads/n_kv_heads

Multi-Query Attention (MQA):
  n_kv_heads == 1  (single KV head, all Q heads attend to it)
  KV cache: minimal (but quality tradeoff)

LLaMA 3 (8B): 8 KV heads vs 32 Q heads → 4× KV reduction
LLaMA 3 (70B): 8 KV heads vs 64 Q heads → 8× KV reduction
```

## Key Performance Metrics

LLM serving performance is characterized by three primary metrics that reflect different aspects of user experience and system efficiency:

### Time to First Token (TTFT)

TTFT measures the latency from when a request is submitted until the first output token is generated. It is dominated by:

1. **Queuing delay** — how long the request waited to be scheduled
2. **Prefill latency** — time to process all prompt tokens
3. **Network round-trip** (for remote APIs)

```
TTFT Timeline:
                      Request arrives
                           │
    ┌──────────────────────▼──────────────────────┐
    │           Waiting Queue                      │
    │           (FCFS scheduling)                  │
    └──────────────────────┬──────────────────────┘
                           │ queuing_delay
    ┌──────────────────────▼──────────────────────┐
    │           Prefill Phase                      │
    │    process N prompt tokens → O(N²)           │
    └──────────────────────┬──────────────────────┘
                           │ prefill_latency
                           ▼
                    First token delivered
                           │
              TTFT = queuing_delay + prefill_latency

Typical values: 100ms – 5s (depending on prompt length,
                             system load, model size)
```

TTFT is critical for interactive applications where users perceive the initial delay as "thinking time." Chunked prefill strategies in modern systems like vLLM directly reduce TTFT by preventing long prefills from blocking decode steps.

### Time Per Output Token (TPOT) / Inter-Token Latency (ITL)

TPOT is the average time between successive output tokens, measuring the speed of the decode phase:

```
TPOT = Total_generation_time / (num_tokens_generated - 1)

Also called: Inter-Token Latency (ITL)

Typical values: 20ms – 200ms per token
                (50 tokens/sec – 5 tokens/sec)

Human reading speed: ~250 words/minute ≈ 4-5 tokens/second
  → For real-time streaming, TPOT < 200ms feels instant
  → TPOT > 500ms feels noticeably slow/choppy

Factors affecting TPOT:
  - Batch size (more requests = more tokens/step = lower TPOT/request)
  - Context length (longer KV cache = more bandwidth per step)
  - Model size (larger models = more compute per step)
  - Hardware memory bandwidth (HBM bandwidth is the bottleneck)
```

### Throughput

Throughput measures system-level output efficiency — how many tokens (or requests) the system can process per unit time across all concurrent users:

```
Output Throughput = total_tokens_generated / wall_clock_time
                  (tokens/second, system-wide)

Request Throughput = total_requests_completed / wall_clock_time
                   (requests/second)

These metrics measure server efficiency rather than user experience.
Higher throughput = more users served = lower cost per request.

Throughput vs. Latency Tradeoff:
┌─────────────────────────────────────────────────────────┐
│  Large batch size → higher throughput, higher latency   │
│  Small batch size → lower throughput, lower latency     │
│                                                         │
│  Throughput                                             │
│      │                          _________ (saturates)  │
│      │                  _______/                        │
│      │           ______/                               │
│      │   _______/                                      │
│      │__/                                              │
│      └──────────────────────────→ Batch Size           │
└─────────────────────────────────────────────────────────┘
```

### Normalized Latency (SLO-oriented)

Production systems often define Service Level Objectives (SLOs) combining both metrics:

```
Common SLO formulations:
  P99 TTFT < 2 seconds
  P99 TPOT < 100ms/token
  End-to-end latency < 30 seconds for 300-token response

Normalized latency = (TTFT + TPOT × output_length) / output_length
                   ≈ TTFT/output_length + TPOT
                   → approaches TPOT for long outputs
```

## Memory-Bound vs. Compute-Bound Regimes

Understanding when inference is memory-bound vs. compute-bound is essential for optimization:

```
Memory-Bound (typical decode):
┌──────────────────────────────────────────────────────────┐
│  GPU Timeline:                                           │
│  ┌──────────────────────────────────┐                   │
│  │  Memory reads (KV cache, weights) ████████████████   │
│  └──────────────────────────────────┘                   │
│  ┌──────────────────────────────────┐                   │
│  │  Compute (matrix multiply)       ████                │
│  └──────────────────────────────────┘                   │
│  → Compute sits idle waiting for memory transfers        │
│  → Solution: batch more requests to amortize BW cost    │
└──────────────────────────────────────────────────────────┘

Compute-Bound (typical prefill):
┌──────────────────────────────────────────────────────────┐
│  GPU Timeline:                                           │
│  ┌──────────────────────────────────┐                   │
│  │  Memory reads                    ████                │
│  └──────────────────────────────────┘                   │
│  ┌──────────────────────────────────┐                   │
│  │  Compute (attention, FFN)        ████████████████    │
│  └──────────────────────────────────┘                   │
│  → Memory feeds fast enough to keep compute busy        │
│  → Solution: reduce arithmetic (e.g., FlashAttention)  │
└──────────────────────────────────────────────────────────┘

The batch size threshold (B*) at which decode transitions
from memory-bound to compute-bound:

  B* ≈ (model_compute_capacity) / (memory_bandwidth × bytes_per_param)

For A100 SXM4:
  FP16 compute: 312 TFLOPS
  HBM BW:       2,000 GB/s
  B* ≈ 312e12 / (2e12 × 2) ≈ 78 sequences

Below B*: more requests → roughly linear throughput gain
Above B*: compute-saturated, diminishing returns
```

## Memory Fragmentation and the KV Cache Problem

Naive KV cache management leads to severe memory waste — a core problem that drove the development of systems like vLLM:

```
Pre-vLLM Memory Waste (3 Sources):

Source 1: Internal Fragmentation
  Request: max_length = 2048, actual output = 200 tokens
  ┌────────────────────────────────────────────────────┐
  │ [████ actual ████░░░░░░░░░░░░░░░░░░░░░░░ reserved ]│
  │  200 tokens used         1848 tokens WASTED        │
  └────────────────────────────────────────────────────┘

Source 2: External Fragmentation
  Contiguous allocation leaves gaps between freed requests:
  [REQ_A][free][REQ_B][free][REQ_C]
         ↑           ↑
    10 tok free   10 tok free
  New request needs 15 tokens → REJECTED (no contiguous 15-token block)

Source 3: Future Token Reservation
  Output length is unknown → must reserve for worst case at start

Combined effect:
  Only 20.4%–38.2% of KV cache memory actually holds token data
  in pre-vLLM systems (SOSP '23 measurement)
```

## Batching Strategies

### Static Batching

The simplest approach: group requests into fixed batches and process all to completion before admitting new requests:

```
Static Batching:
  Batch = {Req A (output: 200 tok), Req B (output: 50 tok)}

  Step 1–50:   [Req A gen] [Req B gen]
  Step 51–200: [Req A gen] [  PADDING ]  ← 150 wasted steps
  Step 201:    Batch complete → next batch admitted

  Problem: Short requests waste GPU time waiting for long ones
  GPU utilization: ~25–60% effective utilization
```

### Dynamic Batching

Admits new requests to a running batch when there is capacity, but still waits for requests to reach some minimum length before batching:

```
Dynamic Batching:
  Improvement over static, but still operates at batch granularity
  New requests wait until a batch slot opens
  Common in TensorRT-LLM and early TGI versions
```

### Continuous Batching (Iteration-Level Scheduling)

The key innovation in modern serving systems: scheduling decisions are made at every single decode step, not at batch boundaries:

```
Continuous Batching:
  Step t:   [Req A gen] [Req B gen] [Req C gen]
  Step t+1: [Req A gen] [Req B DONE→D starts] [Req C gen]
  Step t+2: [Req A gen] [Req D prefill] [Req C gen]
  Step t+3: [Req A gen] [Req D gen] [Req C DONE→E starts]

  Key: NO PADDING, NO WAITING
  New requests fill slots vacated by completed requests instantly
  GPU utilization: 85–95%+ effective utilization
```

Continuous batching is the foundation of systems like vLLM, Orca, and TGI (modern versions).

## Prefill-Decode Interference

Mixing prefill and decode requests in the same batch creates a tricky interference problem:

```
Mixed Batch (without chunked prefill):
  Step: [Req A decode (1 tok)] [Req B PREFILL (2048 tok)]
                                        ↑
              This single prefill monopolizes the entire step!
              Req A decode must wait 2048-token worth of compute.
              All other decode requests experience "jank" (latency spike)

TTFT spikes when long prefills enter the batch:
  ─────────────────────────────────────────────
  Token inter-latency:                    ─── ─── ─── ─── ──── ─── ───
  (decode steps between prefill spikes)              ↑ spike!
  ─────────────────────────────────────────────

Solution — Chunked Prefill:
  Break long prefills into token-budget-sized chunks
  Each step: small prefill chunk + all decode requests
  → Bounded latency impact, no decode starvation
```

## Hardware Considerations

Different GPU generations have different roofline characteristics relevant to LLM serving:

```
GPU Memory Bandwidth vs. Compute (2024):
┌─────────────────────┬──────────────┬──────────────┬──────────┐
│ GPU                 │ HBM BW (TB/s)│ FP16 (TFLOPS)│ Ratio    │
├─────────────────────┼──────────────┼──────────────┼──────────┤
│ A100 40GB SXM4      │ 2.0          │ 312          │ 156 F/B  │
│ A100 80GB SXM4      │ 2.0          │ 312          │ 156 F/B  │
│ H100 SXM5           │ 3.35         │ 989          │ 295 F/B  │
│ H200 SXM5           │ 4.8          │ 989          │ 206 F/B  │
│ RTX 4090            │ 1.0          │ 165          │ 165 F/B  │
└─────────────────────┴──────────────┴──────────────┴──────────┘

Higher bandwidth → better decode throughput
Higher compute → better prefill throughput
H100's NVLink bandwidth enables efficient multi-GPU KV sharing
```

## Quantization and Its Effect on Inference

Reducing numerical precision shrinks both model weights and KV cache:

```
Precision Comparison:
┌──────────┬──────────┬──────────┬────────────────────────────┐
│ Format   │ Bits     │ Weights  │ KV Cache Impact            │
├──────────┼──────────┼──────────┼────────────────────────────┤
│ FP32     │ 32 bits  │ 100%     │ 100% (rarely used)        │
│ BF16/F16 │ 16 bits  │ 50%      │ 50% (standard baseline)   │
│ INT8/FP8 │ 8 bits   │ 25%      │ 25% (with KV quantization)│
│ INT4     │ 4 bits   │ 12.5%    │ 12.5% (aggressive)        │
└──────────┴──────────┴──────────┴────────────────────────────┘

KV cache quantization (FP8) doubles the number of requests
that can be held in memory simultaneously, at modest quality cost.
```

## Speculative Decoding

An advanced technique that accelerates decode by using a fast "draft" model to propose multiple tokens, which the large "target" model verifies in parallel:

```
Speculative Decoding:
  1. Draft model (small, fast) generates K candidate tokens
  2. Target model (large, accurate) verifies all K in ONE forward pass
     (parallelism over K tokens — similar to prefill!)
  3. Accept tokens up to the first mismatch; reject remainder
  4. Target generates correction token for first mismatch

Benefit: When acceptance rate is high (~80%+), effective TPOT
  approaches: target_forward_pass_time / K_accepted_tokens
  vs. baseline: target_forward_pass_time / 1_token

  Speedup: 2×–4× TPOT improvement on many workloads
  Cost: Draft model memory + compute; no throughput gain
```

## Summary: The LLM Inference Serving Challenge

```
┌─────────────────────────────────────────────────────────────┐
│               LLM Inference: Key Challenges                 │
├──────────────────────┬──────────────────────────────────────┤
│ Challenge            │ Solution                             │
├──────────────────────┼──────────────────────────────────────┤
│ Sequential decoding  │ Continuous batching — overlap        │
│ (low GPU util)       │ many requests' decode steps          │
├──────────────────────┼──────────────────────────────────────┤
│ Memory fragmentation │ PagedAttention — OS-style virtual    │
│ (60-80% KV waste)    │ memory paging for KV blocks          │
├──────────────────────┼──────────────────────────────────────┤
│ Prefill-decode       │ Chunked prefill — split long         │
│ interference (jank)  │ prefills across multiple steps       │
├──────────────────────┼──────────────────────────────────────┤
│ Shared prompt waste  │ Prefix caching — reuse KV blocks     │
│ (same system prompt) │ for common token prefixes            │
├──────────────────────┼──────────────────────────────────────┤
│ Memory bandwidth     │ GQA/MQA, KV quantization, larger    │
│ bottleneck           │ batches, faster HBM (H100/H200)      │
├──────────────────────┼──────────────────────────────────────┤
│ Slow decode TPOT     │ Speculative decoding, CUDAGraphs,   │
│                      │ optimized attention kernels           │
└──────────────────────┴──────────────────────────────────────┘
```

---
*Related: [[vLLM Architecture]], [[PagedAttention]], [[KV Cache Management]], [[Continuous Batching]]*
