# Speculative Decoding in vLLM

Speculative decoding is a **lossless** latency optimization technique for autoregressive language model inference. It exploits the asymmetry between token proposal and token verification: a small, fast "draft" mechanism proposes multiple candidate tokens, which the large target model then verifies in a single forward pass. When the draft is correct, multiple tokens are accepted per target model execution — reducing wall-clock latency without changing the statistical output distribution. vLLM supports five distinct speculative decoding strategies, making it one of the most comprehensive inference engines for speculative techniques.

---

## Table of Contents

1. [Core Mechanics and Theoretical Basis](#core-mechanics-and-theoretical-basis)
2. [Acceptance / Rejection Algorithm](#acceptance--rejection-algorithm)
3. [Method 1: Draft Model Speculation](#method-1-draft-model-speculation)
4. [Method 2: N-Gram / Prompt Lookup Decoding](#method-2-n-gram--prompt-lookup-decoding)
5. [Method 3: Medusa (Multi-Head Prediction)](#method-3-medusa-multi-head-prediction)
6. [Method 4: EAGLE (Extrapolation Algorithm)](#method-4-eagle-extrapolation-algorithm)
7. [Method 5: MTP (Multi-Token Prediction)](#method-5-mtp-multi-token-prediction)
8. [Continuous Batching Integration](#continuous-batching-integration)
9. [Performance Characteristics](#performance-characteristics)
10. [When to Use Speculative Decoding](#when-to-use-speculative-decoding)
11. [Configuration Reference](#configuration-reference)
12. [Dynamic Speculation Length](#dynamic-speculation-length)
13. [Interaction with Other Features](#interaction-with-other-features)

---

## Core Mechanics and Theoretical Basis

Standard autoregressive LLM decoding generates exactly one token per forward pass. For a model with 70 billion parameters, this means each token requires processing tens of billions of multiply-accumulate operations. The key observation enabling speculative decoding is:

> **Token verification is much cheaper than token generation.**

In a single causal transformer forward pass, the model can evaluate the probability of *all* input tokens simultaneously (the attention mask ensures causality). If we have K candidate tokens t₁, t₂, ..., tₖ that were proposed by a fast draft mechanism, the target model can evaluate all K positions in **one forward pass** of cost only marginally greater than a single-token decode step (since the additional computation is proportional to K, while the base computation is proportional to sequence length).

### The Efficiency Equation

```
Standard: 1 target forward pass → 1 token
Speculative (ideal): 1 draft forward pass + 1 target forward pass → K tokens
```

The net speedup factor is approximately:
```
speedup ≈ (K_accepted + 1) × (1 / (1 + cost_ratio))
```

Where:
- `K_accepted` = average number of accepted draft tokens per verification step
- `cost_ratio` = (draft cost) / (target cost per equivalent token)

At low QPS (memory-bandwidth-bound regime), the cost ratio for a 10:1 model size difference is approximately 0.1, and EAGLE acceptance rates of ~0.8 yield 2–2.5× speedup.

---

## Acceptance / Rejection Algorithm

Speculative decoding maintains the **exact output distribution** of the target model through a modified rejection sampling procedure. The algorithm ensures that accepting draft tokens never biases the output toward the draft model's distribution.

### Standard Speculative Sampling (Chen et al., 2023)

For each draft token position `i` with draft probability `q(xᵢ)` and target probability `p(xᵢ)`:

```
1. Draft model generates tokens t₁, t₂, ..., tₖ using its own sampling
2. Target model computes p(t₁|context), p(t₂|t₁,context), ..., p(tₖ|t₁..tₖ₋₁,context)
3. For each position i:
   - Accept tᵢ with probability min(1, p(tᵢ) / q(tᵢ))
   - If accepted: continue to i+1
   - If rejected: sample a corrected token from max(0, p - q) distribution; stop
4. If all K tokens accepted: sample one additional token from p(·|t₁..tₖ, context)
```

**Key property:** The output distribution of tokens emitted by this procedure is identical to the distribution that would result from sampling directly from the target model at every step. Speculative decoding is provably lossless.

### Tree-Based Verification (EAGLE, Medusa)

More advanced methods generate a *tree* of draft proposals rather than a single chain:

```
                    t₁ₐ → t₂ₐₐ
                  /         \ t₂ₐᵦ
root context → 
                  \
                    t₁ᵦ → t₂ᵦₐ
```

The target model runs tree attention over all branches simultaneously, verifying multiple paths in one forward pass. The longest verified prefix from any branch is accepted, giving a higher expected number of accepted tokens per verification step compared to linear chains.

---

## Method 1: Draft Model Speculation

The classic speculative decoding setup uses a smaller companion model (the "draft model") to propose tokens. The draft model and target model share vocabulary, enabling direct probability comparison.

### Architecture

```
Request → Draft Model (small, fast) → K candidate tokens
                                           ↓
       Target Model (large, slow) → single forward pass → accept/reject
```

### Vocabulary Constraint

Draft and target models **must share identical vocabularies**. This requirement is automatically satisfied by models from the same family (e.g., Llama-68M drafting for Llama-2-70B), but complicates cross-family combinations. The shared vocabulary ensures that the probability ratio `p(t) / q(t)` is well-defined for any token t.

### Size Ratio Guidelines

| Draft : Target Ratio | Notes |
|---------------------|-------|
| 1:10 to 1:20 | Sweet spot: fast enough drafting, high enough acceptance |
| 1:100+ | Draft is very fast but acceptance rate may be low due to capability gap |
| > 1:5 | Draft overhead can exceed gains |

### vLLM Configuration

```python
from vllm import LLM

llm = LLM(
    model="meta-llama/Meta-Llama-3.1-70B-Instruct",
    tensor_parallel_size=4,
    speculative_config={
        "model": "ibm-fms/llama3-70b-accelerator",
        "draft_tensor_parallel_size": 1,  # Draft can use fewer GPUs
        "num_speculative_tokens": 5,       # K draft tokens per step
    }
)
```

Note that vLLM supports different tensor parallel sizes for the draft and target models, enabling the draft to run on a subset of the GPUs while the target uses all of them.

### Example Pairings

| Target Model | Draft Model | Notes |
|---|---|---|
| Llama-2-70B | Llama-68M | Same family, classic pairing |
| Llama-3.1-70B | ibm-fms/llama3-70b-accelerator | Purpose-trained accelerator |
| Any 70B+ | Corresponding 7–8B variant | Approximate; verify vocab match |

---

## Method 2: N-Gram / Prompt Lookup Decoding

N-gram speculation requires **no draft model** and adds minimal memory overhead. Instead, it predicts upcoming tokens by looking up n-gram patterns in the input prompt.

### Mechanism

1. **Build n-gram index**: At request start, index all n-grams (contiguous token sub-sequences of length 1 through `prompt_lookup_max`) from the input prompt
2. **Lookup during decode**: At each step, look up the last `n` generated tokens in the index; if a match is found, propose the continuation tokens as draft tokens
3. **Verify and accept**: Target model verifies via standard rejection sampling

### Best Use Cases

N-gram speculation excels when the model output heavily quotes or paraphrases the input — the proposed continuations are likely identical to what the model would generate anyway:

- **Summarization**: Model quotes input sentences verbatim
- **Question Answering**: Model copies relevant facts from the provided context
- **Code completion**: Model re-uses variable names and patterns from the provided code
- **Translation with provided reference**: Model produces tokens matching the source

**Reported performance:** Up to **2.8× speedup** on CNN/DailyMail summarization with Llama-3-70B (4×H100) at QPS=1.

### Configuration

```python
llm = LLM(
    model="facebook/opt-6.7b",
    speculative_config={
        "method": "ngram",
        "prompt_lookup_max": 4,   # Maximum n-gram length to match
        "prompt_lookup_min": 1,   # Minimum n-gram length to match
        "num_speculative_tokens": 5,
    }
)
```

### Suffix Decoding Extension

**Suffix Decoding** is a variant that extends n-gram lookup beyond the input prompt to match suffixes of the *generated output* against a pre-built corpus of common continuations. This is particularly effective for code generation with a corpus of common code patterns, or for domain-specific applications with predictable phrasings.

---

## Method 3: Medusa (Multi-Head Prediction)

Medusa takes a different architectural approach: instead of a separate draft model, it adds **multiple lightweight prediction heads** directly to the target model's final transformer block. Each head independently predicts one future token position.

### Architecture

```
Input → [Target Model Transformer Blocks] → Last Hidden State
                                                    ↓
                                         ┌─── Medusa Head 1 → logits for t+1
                                         ├─── Medusa Head 2 → logits for t+2
                                         ├─── Medusa Head 3 → logits for t+3
                                         └─── Medusa Head K → logits for t+K
```

Each Medusa head is a 2-layer MLP applied to the last transformer hidden state.

### Tree Attention Verification

The Cartesian product of top-N candidates from each head forms a candidate tree:

```
Head 1 candidates: [a, b, c]
Head 2 candidates: [x, y, z]
Tree: ax, ay, az, bx, by, bz, cx, cy, cz (9 paths)
```

Tree attention processes all paths simultaneously using a specialized masking pattern. The target model's lm_head reuses the same hidden states, so the overhead of verification is primarily the attention computation over the candidate tree tokens.

### Training Requirement

Medusa heads must be **fine-tuned on task data** — they are not useful out of the box because they need to learn the target model's likely next-token distributions. Pre-trained Medusa heads are available for popular models (Vicuna, Llama-2) but require fine-tuning for custom models or fine-tuned variants.

### Performance

Medusa typically achieves **1.4–1.64× speedup** on common benchmarks. This is lower than EAGLE because Medusa heads (linear layers operating on the final hidden state) have less predictive power than EAGLE's full transformer layer approach.

---

## Method 4: EAGLE (Extrapolation Algorithm for Greater Language-Model Efficiency)

EAGLE (published ICML 2024) is currently the state-of-the-art speculative decoding method for most tasks, offering the highest acceptance rates and speedups among available methods.

### Key Innovation: Feature-Level Drafting

Unlike Medusa (which predicts from the final hidden state using linear heads) or external draft models (which start from tokens), EAGLE uses **1–2 full transformer layers** that take the *target model's hidden states* as input:

```
Target model hidden state at step t →
  EAGLE Layer (lightweight transformer) →
  Predicted hidden states for t+1, t+2, ... →
  LM head (shared with target) →
  Token probability distributions
```

By operating at the feature level rather than the token level, EAGLE has access to much richer information about the model's internal state, leading to much higher acceptance rates than Medusa or draft models.

### EAGLE Evolution

| Version | Key Improvement | Typical Speedup |
|---------|-----------------|-----------------|
| **EAGLE-1** (ICML 2024) | Single lightweight transformer layer as draft | 2.0–2.5× |
| **EAGLE-2** | Dynamic speculation tree: prunes low-probability branches based on draft confidence | 2.5–3.0× |
| **EAGLE-3** | Further architectural improvements; current SOTA | 2.5–3.5× |

### Dynamic Tree in EAGLE-2+

Early EAGLE used a fixed speculation tree structure (fixed branching factor at each depth). EAGLE-2 adapts the tree dynamically based on the draft model's confidence:

```
High-confidence predictions → deep tree (explore further)
Low-confidence predictions → prune early (don't waste verification budget)
```

This adaptive strategy maximizes the expected accepted tokens per verification step.

### vLLM Configuration

```python
llm = LLM(
    model="meta-llama/Meta-Llama-3-8B-Instruct",
    tensor_parallel_size=4,
    speculative_config={
        "model": "yuhuili/EAGLE-LLaMA3-Instruct-8B",
        "draft_tensor_parallel_size": 1,
        "num_speculative_tokens": 2,  # Can often be higher for EAGLE-2+
        "method": "eagle",
    }
)
```

### Published Speedup Comparison

The following comparison is from published benchmarks across common chat/reasoning tasks:

| Method | Average Speedup |
|--------|----------------|
| **EAGLE** | **2.16×** |
| Draft Model (SpS) | 1.83× |
| Hydra | 1.80× |
| PLD (n-gram) | 1.55× |
| **Medusa** | **1.44×** |

EAGLE outperforms all alternatives on average, with particularly strong performance on structured tasks where the feature-level draft can accurately predict patterns in the model's reasoning.

---

## Method 5: MTP (Multi-Token Prediction)

Some models are natively trained with multi-token prediction heads — predicting multiple future tokens simultaneously as part of the training objective. These models can leverage MTP for speculation without any additional architecture.

### Native Integration: DeepSeek-V3

DeepSeek-V3 is trained with dedicated MTP heads as part of its architecture. vLLM V1 integrates native MTP-based speculation for DeepSeek-V3, enabling speculation without a separate draft model or additional fine-tuning:

```python
llm = LLM(
    model="deepseek-ai/DeepSeek-V3",
    speculative_config={
        "method": "deepseek_mtp",
        "num_speculative_tokens": 1,
    }
)
```

MTP heads are architecturally part of the main model, so there is no "separate draft model" overhead — the draft predictions are computed as part of the regular forward pass, making MTP-based speculation particularly efficient.

---

## Continuous Batching Integration

vLLM integrates speculative decoding with continuous batching via two specialized execution runners:

### Draft Runner

Executes the draft model (or equivalent mechanism) to produce `num_speculative_tokens` candidate token proposals for each actively-decoding sequence in the batch. The draft runner manages a separate KV cache for the draft model when using a separate draft model.

### Target Runner

Verifies all candidates from all sequences in a single batched forward pass of the target model. The verification pass processes:
- All `num_speculative_tokens` candidate positions per sequence
- All sequences in the current decode batch simultaneously

This batching means verification overhead grows sub-linearly with batch size.

### Mixed Batch Handling

Different requests in a batch can have different statuses simultaneously:
- Some requests may be in prefill phase (no speculation)
- Some may be in speculative decode phase
- Some may have just finished prefill and are entering decode

The scheduler handles these mixed states transparently, applying speculation only to requests in the decode phase.

### KV Cache for Draft + Target

When using a separate draft model, vLLM allocates and manages KV caches for both models simultaneously. The memory budget must account for both:

```
Total KV Memory = target_model_kv + draft_model_kv
```

The scheduler coordinates block allocation for both KV caches as sequences grow.

---

## Performance Characteristics

### Conditions for Speedup

Speculative decoding improves latency **only when the system is memory-bandwidth bound**, not compute bound. This condition holds at low-to-moderate QPS:

| QPS Regime | GPU Bottleneck | Spec Decode Effect |
|-----------|----------------|-------------------|
| Low QPS (< 10 req/s for 70B) | Memory bandwidth | Significant speedup (1.5–3×) |
| Moderate QPS | Transitioning | Diminishing returns |
| High QPS (compute-bound) | GPU compute | Can reduce throughput (overhead > gain) |

### Why High QPS Hurts

At high QPS:
- The GPU is already fully utilized during target model execution
- Adding draft model execution increases total compute, reducing throughput
- The gain (fewer target passes) is outweighed by the cost (extra draft passes)

### Task Dependency

Acceptance rates vary significantly by task type:

| Task Type | Typical Acceptance Rate | Notes |
|-----------|------------------------|-------|
| Code completion | High (0.7–0.9) | Predictable syntax patterns |
| Summarization | Very high (0.8–0.95) | Output often quotes input |
| Open-ended chat | Moderate (0.5–0.7) | More creative variability |
| Math reasoning | Moderate (0.5–0.75) | Structured but variable |
| Creative writing | Low (0.3–0.5) | High diversity |

---

## When to Use Speculative Decoding

### ✅ Use Speculative Decoding When:

- **Low-to-moderate QPS**: Server is not compute-bound; memory bandwidth is the limiting factor
- **Latency-critical single requests**: Reducing TTOT (time-to-output-tokens) matters more than throughput
- **High-repetition tasks**: Summarization, code generation from context, RAG answers
- **Using EAGLE or n-gram**: Minimal memory overhead, high acceptance rates
- **DeepSeek-V3 or MTP-trained models**: Native MTP heads with no extra architecture

### ❌ Avoid Speculative Decoding When:

- **High QPS, compute-bound**: Draft model overhead reduces throughput
- **Creative tasks with low acceptance**: Overhead exceeds minimal accepted tokens per step
- **Memory-constrained deployment**: Cannot accommodate both draft + target KV caches
- **Long output sequences at high QPS**: Better to maximize batch size than per-request speed

### Decision Checklist

```
1. Is TTOT (latency) or throughput the primary goal?
   → Latency: consider spec decode
   → Throughput: evaluate carefully; test at target QPS

2. What is the typical task?
   → Summarization / RAG / code: n-gram or EAGLE strongly recommended
   → Chat / general: EAGLE with companion model

3. What hardware is available?
   → Single GPU: n-gram or Medusa (no extra model memory)
   → Multiple GPUs: draft model or EAGLE with separate TP sizing

4. Is a draft model available for the target?
   → Yes: use draft model or EAGLE
   → No: use n-gram or Medusa
```

---

## Configuration Reference

### Server-Side (vLLM Serve)

```bash
# Draft model approach
vllm serve meta-llama/Meta-Llama-3.1-70B-Instruct \
    --speculative-model ibm-fms/llama3-70b-accelerator \
    --num-speculative-tokens 5 \
    --speculative-draft-tensor-parallel-size 1

# N-gram approach
vllm serve <model> \
    --speculative-model "[ngram]" \
    --num-speculative-tokens 5 \
    --ngram-prompt-lookup-max 4

# EAGLE approach
vllm serve meta-llama/Meta-Llama-3-8B-Instruct \
    --speculative-model yuhuili/EAGLE-LLaMA3-Instruct-8B \
    --speculative-draft-tensor-parallel-size 1 \
    --num-speculative-tokens 2
```

### Python API

```python
from vllm import LLM

# Draft model
llm = LLM(
    model="meta-llama/Meta-Llama-3.1-70B-Instruct",
    tensor_parallel_size=4,
    speculative_config={
        "model": "ibm-fms/llama3-70b-accelerator",
        "draft_tensor_parallel_size": 1,
        "num_speculative_tokens": 5,
    }
)

# N-gram
llm = LLM(
    model="facebook/opt-6.7b",
    speculative_config={
        "method": "ngram",
        "prompt_lookup_max": 4,
        "prompt_lookup_min": 1,
        "num_speculative_tokens": 5,
    }
)

# EAGLE
llm = LLM(
    model="meta-llama/Meta-Llama-3-8B-Instruct",
    tensor_parallel_size=4,
    speculative_config={
        "model": "yuhuili/EAGLE-LLaMA3-Instruct-8B",
        "draft_tensor_parallel_size": 1,
        "num_speculative_tokens": 2,
        "method": "eagle",
    }
)
```

---

## Dynamic Speculation Length

A known limitation of current vLLM speculative decoding is that `num_speculative_tokens` is a fixed value set at server startup. At high QPS, this fixed value may cause throughput regression because:

- A high `num_speculative_tokens` at high QPS means more draft compute per step
- At high QPS, acceptance rates also tend to drop (batch diversity increases)

**Dynamic speculation length** (under active development) automatically adjusts `num_speculative_tokens` based on system load metrics:
- High load / high QPS → reduce or disable speculation
- Low load / low QPS → increase speculation aggressiveness

This would eliminate the need for operators to manually tune speculation parameters based on expected QPS patterns.

---

## Interaction with Other Features

### With Prefix Caching (APC)

Speculative decoding and APC are complementary:
- APC reduces **prefill** time (TTFT)
- Speculative decoding reduces **decode** time (TPOT)
- Both can be enabled simultaneously for maximum end-to-end latency reduction

### With Structured Output (Guided Decoding)

When guided decoding constrains outputs via token masks, these masks must also be applied to draft tokens. vLLM's V1 roadmap includes integrating structured output state machines with speculative decoding's tree-scoring verification, so that both features work correctly together.

### With Chunked Prefill

Speculation operates only during the decode phase, after prefill is complete. Chunked prefill affects how the prefill phase proceeds but does not interfere with speculative decode behavior once decoding begins.

### With LoRA

When LoRA adapters are in use, the draft model must also support (or match) the LoRA configuration. For EAGLE-based methods, the EAGLE draft model typically needs its own corresponding LoRA-aware version, which limits LoRA + EAGLE combinations to cases where fine-tuned EAGLE models exist for the specific LoRA.
