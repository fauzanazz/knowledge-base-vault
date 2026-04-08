# Scaling Laws and Training Pipelines for Large Language Models
## A Comprehensive Deep Dive from Data to Deployment

---

## Table of Contents

1. [Introduction](#introduction)
2. [Scaling Laws: The Science of Growth](#scaling-laws)
   - Kaplan et al. (2020): The OpenAI Power Laws
   - Chinchilla (2022): The Compute-Optimal Revolution
   - Beyond Chinchilla: Modern Deviations
3. [Pre-Training Data Curation](#data-curation)
   - Common Crawl: The Raw Ore
   - The Pile: The Composite Dataset Era
   - RedPajama: Open Reproduction
   - FineWeb: Decanting the Web
   - Data Mixing & Quality Signals
4. [Training Infrastructure](#infrastructure)
   - Data Parallelism
   - Tensor Parallelism (Megatron-LM)
   - Pipeline Parallelism
   - 3D Parallelism
   - ZeRO and DeepSpeed
   - FSDP (PyTorch Native)
5. [Mixed Precision Training](#mixed-precision)
   - FP16 and the Need for Loss Scaling
   - BF16: The Modern Default
   - FP8: The Emerging Frontier
   - Gradient Checkpointing
6. [Learning Rate Schedules](#lr-schedules)
   - Linear Warmup
   - Cosine Decay
   - Warmup–Stable–Decay (WSD)
7. [Mixture of Experts (MoE)](#moe)
   - Architecture Fundamentals
   - Switch Transformer: Top-1 Routing
   - Mixtral 8×7B: Top-2 Routing
   - DeepSeek-V3: Auxiliary-Loss-Free Routing
   - Load Balancing
8. [Continual Pre-Training](#continual)
9. [The Full Pipeline: Data to Deployment](#full-pipeline)
10. [Key Takeaways](#takeaways)

---

## 1. Introduction <a name="introduction"></a>

Large language models (LLMs) have transformed the AI landscape through a cascade of engineering and scientific breakthroughs, from the mathematical formalisms governing model size and data scale to the distributed systems enabling trillion-parameter training runs. Understanding this landscape requires navigating multiple interacting disciplines: empirical scaling science, large-scale data engineering, distributed systems architecture, numerical methods, and optimization theory.

This article provides a rigorous, unified treatment of the entire LLM training lifecycle — from the question of *how much compute should I spend on parameters vs. tokens?* to the question of *how do I serve a 671-billion parameter model in production?*

---

## 2. Scaling Laws: The Science of Growth <a name="scaling-laws"></a>

### 2.1 Kaplan et al. (2020): The OpenAI Power Laws

The modern era of principled LLM scaling began with the 2020 paper *Scaling Laws for Neural Language Models* by Kaplan et al. at OpenAI. Their key finding was that transformer language model loss follows clean power-law relationships with three quantities: the number of model parameters **N**, the size of the training dataset **D** (in tokens), and the computational budget **C** (in FLOPs).

The loss function was modeled as:

```
L(N, D) ≈ (N_c/N)^α_N + (D_c/D)^α_D + L_∞
```

Where `L_∞` is the irreducible loss attributable to the true entropy of natural language, and `N_c`, `D_c`, `α_N`, `α_D` are empirically fitted constants.

The Kaplan analysis suggested that **model size should grow much faster than data** for a fixed compute budget — specifically, parameters should scale as `N ∝ C^0.73` while training tokens scale only as `D ∝ C^0.27`. This led to a generation of significantly *undertrained* large models: GPT-3 (175B parameters, ~300B tokens), Gopher (280B parameters), and LaMDA operated under the implicit assumption that more parameters were worth more than more data.

### 2.2 Chinchilla (2022): The Compute-Optimal Revolution

In March 2022, Jordan Hoffmann, Sebastian Borgeaud, and colleagues at DeepMind published *Training Compute-Optimal Large Language Models* (arXiv:2203.15556), colloquially known as the **Chinchilla paper**. It became the most influential result in applied LLM science in years.

By training **over 400 language models** ranging from 70M to 16B parameters across 5B to 500B tokens, the authors discovered that the Kaplan recommendation was systematically wrong: prior large models were massively undertrained relative to their compute budget.

**The Chinchilla Scaling Equations:**

For a compute budget C (in FLOPs), the optimal model size N\* and optimal training token count D\* follow:

```
N_opt ∝ C^a    where a ≈ 0.49–0.50
D_opt ∝ C^b    where b ≈ 0.49–0.51
```

The exponents are approximately equal, leading to the headline finding: **model size and training tokens should scale in equal proportions as compute increases.** The practical rule of thumb derived from this:

```
D_opt / N_opt ≈ 20
```

**For every model parameter, you need approximately 20 training tokens for compute-optimal training.**

The paper validated this by training Chinchilla itself: a 70B parameter model trained on 1.4 trillion tokens (20× its parameter count), using the same compute as Gopher (280B, ~300B tokens). Chinchilla uniformly outperformed Gopher, GPT-3, Jurassic-1, and Megatron-Turing NLG (530B) — despite being 4× smaller than the largest competitor. On MMLU, Chinchilla achieved 67.5% accuracy, a 7%+ improvement over Gopher.

**Three methods of estimation were used**, all converging on the same result:
1. **Fixed compute, varying model sizes** — Fit a curve through the loss minimum for each FLOP budget
2. **IsoFLOP profiles** — For each compute level, find the model size that minimizes final loss
3. **Parametric fitting** — Directly fit a joint loss function L(N, D) over all experiments

The implications were immediate and profound:
- A 10B parameter model needs ~200B tokens
- A 70B parameter model needs ~1.4T tokens
- A 500B parameter model needs ~10T tokens

**This reframed LLM development** from a pure parameter-count race into a careful resource allocation problem between model capacity and data volume.

### 2.3 Beyond Chinchilla: Modern Deviations

In practice, post-Chinchilla models increasingly train *beyond* the compute-optimal point. Why? Because Chinchilla optimal is optimal for a **single training run at fixed compute**, but it ignores **inference costs**. A smaller, more-trained model:
- Costs less to serve at inference time
- Has lower memory requirements
- Enables larger deployment batches

This led to a systematic trend of training smaller models on dramatically more tokens:
- **LLaMA 1** (2023): 7B–65B parameters trained on 1T–1.4T tokens
- **LLaMA 2** (2023): 7B–70B on 2T tokens
- **LLaMA 3** (2024): 8B and 70B on 15T+ tokens
- **Mistral 7B**: ~7B on ~1T+ tokens, vastly exceeding Chinchilla ratio

Additionally, more recent work (e.g., MosaicML, 2023) suggested the optimal ratio may be closer to 40–70 tokens per parameter when **data quality is improved** — showing that dataset quality can substitute for raw token count.

**The "Chinchilla" rule of 20 tokens/parameter remains the primary benchmark**, but is best understood as a lower bound for compute-optimality, not an upper bound for performance.

---

## 3. Pre-Training Data Curation <a name="data-curation"></a>

The quality and composition of pre-training data is arguably the single most impactful factor in final model performance. Building multi-trillion-token datasets requires a sophisticated engineering and data science pipeline.

### 3.1 Common Crawl: The Raw Ore

[Common Crawl](https://commoncrawl.org/) is the bedrock of nearly all large-scale pre-training datasets. It is a nonprofit that has been crawling the public web since 2011 and releases monthly snapshots of petabytes of raw HTML. A single monthly crawl contains tens of billions of documents. As of 2024, there are 90+ total crawl snapshots, representing hundreds of terabytes of compressed raw text.

**Why Common Crawl is necessary but insufficient:**
- Covers the breadth of human internet text
- Free and publicly accessible
- Updated monthly with fresh content

**Why it requires heavy processing:**
- Massive noise: spam, boilerplate, pornography, malware, near-duplicate content
- Low average text quality: ~80%+ of raw extracted text is unusable
- Language imbalance: predominantly English
- Deduplication is critical — without it, the model memorizes repeated content

### 3.2 The Pile: The Composite Dataset Era

[The Pile](https://arxiv.org/abs/2101.00027) (EleutherAI, 2021) was a landmark open dataset comprising **825 GB** of diverse text from **22 curated sources**, including:

| Source | Type |
|---|---|
| Pile-CC | Web text (Common Crawl) |
| PubMed Central | Scientific articles |
| Books3 | Books |
| GitHub | Code |
| ArXiv | Research papers |
| Wikipedia | Encyclopedia |
| OpenWebText2 | Reddit-filtered web |
| FreeLaw | Legal opinions |
| StackExchange | Q&A |
| DM Mathematics | Math |

The Pile's core innovation was demonstrating that **mixing diverse, high-quality domain-specific data** substantially improved model performance on specialized tasks compared to web-only corpora. It showed that a "diverse yet curated" approach beat naive data scaling.

The Pile was used to train EleutherAI's GPT-Neo and GPT-NeoX-20B series, establishing the template for open-source LLM development.

### 3.3 RedPajama: Open Reproduction and Scale

[RedPajama](https://github.com/togethercomputer/redpajama-data) (Together AI, 2023) was designed as an open reproduction of the LLaMA training data, which Meta had not publicly released. It spawned two major versions:

**RedPajama-V1** reproduced the ~1.2 trillion token LLaMA dataset composition:
- CommonCrawl (processed via CCNet pipeline)
- C4 (Colossal Clean Crawled Corpus)
- GitHub code
- Wikipedia
- Books (Gutenberg, Books3)
- ArXiv
- StackExchange

**RedPajama-V2** dramatically scaled up to **100+ trillion tokens** from 84 Common Crawl snapshots (5 European languages). Its key innovation was releasing the data in *two layers*:
1. **Raw, unfiltered text** — available for custom filtering
2. **Quality signals** — pre-computed labels from CCNet, C4, MassiveText, RefinedWeb heuristics, plus Bloom filter exact deduplication and MinHash fuzzy deduplication labels

This "signals-as-metadata" design allowed researchers to filter the dataset *without* reprocessing the raw data, enabling rapid experimentation with different data mixing strategies.

### 3.4 FineWeb: Decanting the Web

[FineWeb](https://arxiv.org/abs/2406.17557) (HuggingFace, 2024) represents the current state of the art in English web pre-training data, comprising **15 trillion tokens** derived from 96 Common Crawl snapshots. The FineWeb paper is one of the most carefully documented data curation studies ever published.

**FineWeb's Processing Pipeline:**

```
Common Crawl Dumps (WARC files)
        ↓
URL Filtering (block-listed domains)
        ↓
Text Extraction (trafilatura)
        ↓
Language Detection (fastText)
        ↓
Quality Heuristics (line/word/character level)
        ↓
MinHash Deduplication (per-dump + cross-dump)
        ↓
FineWeb (15T tokens)
        ↓
Educational Content Classifier (LLM judge)
        ↓
FineWeb-Edu (1.3T tokens)
```

Key design choices from the FineWeb ablation study:
- **Per-dump deduplication** before cross-dump deduplication preserves diversity better than global deduplication
- **Line-level filtering** (remove lines with excessive punctuation, numbers, short lines) outperforms paragraph-level
- **FineWeb-Edu** — filtering for educational quality using a LLaMA-3-8B-based classifier — produced models that significantly outperformed all baseline datasets on MMLU and ARC benchmarks despite being only 1.3T tokens

Models trained on FineWeb outperformed those trained on C4, Dolma-v1.6, The Pile, SlimPajama, and RedPajama on standard benchmarks, establishing FineWeb as the reference open web dataset as of 2024.

### 3.5 Data Mixing and Quality Signals

Modern pre-training datasets are carefully **mixed** from multiple sources with specific weights:

| Source Type | Role | Typical Weight |
|---|---|---|
| Web crawl (CC/FineWeb) | Broad knowledge, scale | 60–80% |
| Code (GitHub) | Reasoning, structured text | 5–15% |
| Books | Long-form coherence | 5–10% |
| Wikipedia | Factual accuracy anchoring | 1–5% |
| Scientific papers | Domain depth | 1–5% |
| Math corpora | Mathematical reasoning | 1–5% |

**Key data quality signals** computed during filtering:
- **Perplexity score** (CCNet KenLM): Text with high perplexity from a Wikipedia-trained LM is likely garbage
- **Deduplication**: MinHash LSH for fuzzy deduplication; Bloom filters for exact match
- **Language identification**: fastText lid.176.bin
- **Heuristic filters**: character/word ratio, punctuation density, URL density, stop word ratio, alphanumeric fraction
- **Classifier-based quality**: GPT-4/LLaMA-based educational quality scores

---

## 4. Training Infrastructure <a name="infrastructure"></a>

Training a 70B+ parameter model requires distributing computation across hundreds or thousands of GPUs simultaneously. This is not merely a software engineering challenge — it requires careful co-design of algorithms, communication patterns, and hardware topology.

### 4.1 Data Parallelism

The simplest form of distributed training. Each GPU holds a **full copy of the model** but processes a different shard of the training data. Gradients are averaged across all GPUs after each forward/backward pass via `AllReduce`.

```
GPU 0: batch[0:B]  → gradients_0
GPU 1: batch[B:2B] → gradients_1
...
GPU N: batch[...] → gradients_N

AllReduce: avg_grad = (grad_0 + grad_1 + ... + grad_N) / N
All GPUs update weights with avg_grad
```

**PyTorch DDP (DistributedDataParallel)** is the standard implementation. It overlaps gradient communication with backward computation for efficiency.

**Limitation**: Requires the entire model to fit in one GPU's memory — impossible for 70B+ models on standard hardware.

### 4.2 Tensor Parallelism (Megatron-LM)

[Megatron-LM](https://arxiv.org/abs/1909.08053) (NVIDIA, 2019) introduced **intra-layer (tensor) model parallelism**: splitting individual weight matrices across multiple GPUs.

For a transformer's feed-forward layer `Y = XW`, with W being a `[d_model × d_ff]` matrix:
- **Column parallelism**: Split W into `[d_model × d_ff/N]` chunks — each GPU computes part of the output
- **Row parallelism**: Split along the input dimension, requiring an AllReduce to sum outputs

For self-attention layers, the Q, K, V projection matrices are split across attention heads. Each GPU handles a subset of heads independently.

```
GPU 0: heads 0–8   (W_Q0, W_K0, W_V0)
GPU 1: heads 8–16  (W_Q1, W_K1, W_V1)
...
GPU 7: heads 56–64 (W_Q7, W_K7, W_V7)
→ AllReduce after output projection
```

Tensor parallelism requires 2 `AllReduce` operations per transformer layer (one in forward, one in backward), making it **bandwidth-intensive**. It is therefore typically confined to **within a single NVLink-connected node** (where bandwidth is ~600 GB/s on NVSwitch), not across nodes.

Megatron-LM further introduced **sequence parallelism**: distributing LayerNorm and Dropout across the sequence dimension, reducing peak activation memory.

### 4.3 Pipeline Parallelism

Pipeline parallelism splits the **layers** of a model across GPUs (or nodes), with each stage processing a *micro-batch* and passing activations to the next stage.

```
Stage 0 (GPU 0–1): Layers 0–15
Stage 1 (GPU 2–3): Layers 16–31
Stage 2 (GPU 4–5): Layers 32–47
Stage 3 (GPU 6–7): Layers 48–63
```

The naive approach suffers from the **pipeline bubble** — GPUs in early stages sit idle while later stages finish the backward pass. Megatron-LM's **PipeDream-Flush / 1F1B schedule** interleaves micro-batches to reduce the bubble fraction from `O(p-1)/p` (where p = pipeline stages) toward a smaller constant. The **interleaved pipeline schedule** further reduces the bubble by assigning non-contiguous layer "chunks" to each stage.

Pipeline parallelism communicates only **activations** between stages (not full weights), making it suitable for **cross-node communication** over InfiniBand.

### 4.4 3D Parallelism

The combination of data, tensor, and pipeline parallelism forms **3D parallelism**, pioneered by Megatron-LM and Megatron-DeepSpeed. This allows scaling to trillion-parameter models:

```
Total GPUs = DP_degree × TP_degree × PP_degree

Example (1024 GPUs, GPT-3 175B):
  TP = 8  (within each DGX node, using NVLink)
  PP = 16 (across nodes, over InfiniBand)
  DP = 8  (across DP groups)
  → 8 × 16 × 8 = 1024 GPUs
```

NVIDIA achieved **163 TFLOPs/GPU** (end-to-end including communication) training a 1T parameter model on 3072 A100 GPUs using this approach, representing 51.4% of the theoretical peak.

### 4.5 ZeRO and DeepSpeed

Microsoft's [DeepSpeed](https://www.deepspeed.ai/) library introduced the **Zero Redundancy Optimizer (ZeRO)**, which addresses the memory redundancy of standard data parallelism.

In standard DDP, each GPU holds:
- **Parameters**: 2 bytes/param (fp16) × N
- **Gradients**: 2 bytes/param × N
- **Optimizer states**: 12 bytes/param (fp32 params + Adam m/v) × N

Total: ~16 bytes/param → 112 GB for a 7B model, far exceeding a single GPU.

**ZeRO partitions these states across data-parallel ranks:**

| ZeRO Stage | What is partitioned | Memory reduction |
|---|---|---|
| **ZeRO-1** | Optimizer states | 4× |
| **ZeRO-2** | Optimizer states + gradients | 8× |
| **ZeRO-3** | Optimizer states + gradients + parameters | 64× (with 64 GPUs) |

ZeRO-3 is the most aggressive: no GPU holds the full model. During the forward and backward passes, parameters are gathered from all ranks via `AllGather`, used locally, then freed. This enables training models whose total size exceeds any individual GPU.

**ZeRO-Infinity** extends this to NVMe SSD offloading, enabling models with trillions of parameters to be trained on commodity hardware (at the cost of I/O throughput).

DeepSpeed also provides:
- **Activation checkpointing** integration
- **Kernel fusion** for attention and layer norm
- **Communication optimization** (bucket overlap, compression)
- **FP16/BF16 training** with automatic loss scaling

### 4.6 FSDP (Fully Sharded Data Parallel)

PyTorch's native **FSDP** (introduced in PyTorch 1.11, made production-ready in 1.12) is the open-source alternative to ZeRO-3, implemented directly in the PyTorch codebase.

FSDP shards parameters, gradients, and optimizer states across all GPUs (like ZeRO-3), but does so at the module level using `FlatParameter` groups. Key modes:

| FSDP Mode | Behavior |
|---|---|
| `FULL_SHARD` | Full ZeRO-3: params + grads + optimizer sharded |
| `SHARD_GRAD_OP` | Like ZeRO-2: only grads + optimizer sharded |
| `HYBRID_SHARD` | Full shard within a node, replicate across nodes |
| `NO_SHARD` | DDP-equivalent (no sharding) |

`HYBRID_SHARD` is particularly useful for multi-node training with limited inter-node bandwidth — it minimizes expensive cross-node AllGather by replicating within nodes.

FSDP integrates seamlessly with HuggingFace Accelerate and Transformers, making it the preferred choice for researchers training models outside the Megatron-specific code path.

---

## 5. Mixed Precision Training <a name="mixed-precision"></a>

### 5.1 The Case for Lower Precision

FP32 (32-bit float) uses 4 bytes per value. For a 7B parameter model:
- Parameters alone: 28 GB
- Gradients: 28 GB
- Adam optimizer states (m, v in FP32): 56 GB
- Activations: variable, but often 30–100+ GB

This is prohibitive on a single GPU. Mixed precision training reduces memory by representing forward/backward computation in 16-bit formats while retaining critical state in FP32.

Modern GPUs provide dedicated hardware (Tensor Cores) for 16-bit matrix multiplication — A100s can deliver 312 TFLOPS in BF16 vs. 77.6 TFLOPS in FP32, a 4× compute throughput advantage.

### 5.2 FP16 Training

**FP16 (IEEE 754 half-precision):**
- 1 sign bit, 5 exponent bits, 10 mantissa bits
- Range: ~6.1×10⁻⁵ to 65,504
- Precision: ~3 decimal digits

The narrow dynamic range of FP16 creates two critical problems:
1. **Gradient underflow**: Small gradients (common in early training and deep layers) can round to zero, causing training stagnation
2. **Gradient overflow**: Large gradients exceed 65,504 and become `inf` or `NaN`, corrupting the training step

**Loss scaling** addresses these issues by multiplying the loss by a large scale factor (e.g., 2¹⁶) before the backward pass, shifting gradient values into the representable FP16 range, then unscaling before the optimizer step.

**Dynamic loss scaling** (used by NVIDIA Apex and PyTorch `GradScaler`) automatically adjusts the scale factor:
- If any gradient is `inf`/`NaN`, skip the optimizer step and halve the scale factor
- After `N` consecutive clean steps, double the scale factor

**Master weights in FP32**: The authoritative copy of parameters is always kept in FP32. FP16 "working copies" are used only for forward/backward passes. The FP32 weights are updated by the optimizer, then re-cast to FP16 for the next forward pass.

### 5.3 BF16: The Modern Default

**BF16 (Brain Float 16, developed by Google Brain):**
- 1 sign bit, 8 exponent bits, 7 mantissa bits
- **Same dynamic range as FP32** (~1.18×10⁻³⁸ to 3.39×10³⁸)
- Lower precision than FP16 (7 vs. 10 mantissa bits)

The key advantage: because BF16 has the same exponent width as FP32, **loss scaling is unnecessary**. Gradients that would overflow FP16 are perfectly representable in BF16.

**BF16 has become the default for large model training** on modern hardware:
- NVIDIA A100/H100 provide native BF16 Tensor Core support
- Google TPUs are optimized for BF16
- PaLM, Gemini, LLaMA 2/3, Mistral, and virtually all frontier models use BF16 training

**Trade-off**: BF16 has fewer mantissa bits, so weight updates with very small magnitude relative to weight values may round to zero. In practice, this is rarely a limiting factor for transformer language model training.

```
Format Comparison:
FP32:  [1 sign][8 exponent][23 mantissa] = 32 bits
FP16:  [1 sign][5 exponent][10 mantissa] = 16 bits
BF16:  [1 sign][8 exponent][7 mantissa]  = 16 bits
FP8 (E4M3): [1 sign][4 exponent][3 mantissa] = 8 bits
```

### 5.4 FP8: The Emerging Frontier

FP8 training (used in DeepSeek-V3, Transformer Engine) halves memory again vs. BF16. Two variants:
- **E4M3** (4 exponent, 3 mantissa): Better precision, used for activations
- **E5M2** (5 exponent, 2 mantissa): Wider range, used for gradients

FP8 training requires **per-tensor scaling factors** to prevent underflow/overflow, and not all operations are FP8-safe. The current best practice keeps attention and layer-norm in BF16 while using FP8 for linear layer GEMMs.

### 5.5 Gradient Checkpointing (Activation Checkpointing)

During a standard forward pass, all intermediate activations are saved in GPU memory for use in the backward pass. For a transformer with L layers and sequence length T, this requires O(L × T) activation memory — often the dominant memory cost.

**Gradient checkpointing** (Chen et al., 2016; also called *activation checkpointing*) trades compute for memory by **discarding activations during the forward pass** and **recomputing them on demand** during the backward pass.

```
Standard training:
Forward:   [Act₀] → [Act₁] → [Act₂] → ... → [Actₗ]  (all stored)
Backward:  Use stored activations to compute gradients

Gradient checkpointing:
Forward:   [Act₀] → ✗ → [Act₂] → ✗ → [Act₄] → ...  (checkpoints only)
Backward:  Recompute Act₁ from Act₀, compute grad, then recompute Act₃ from Act₂...
```

**Memory vs. compute trade-off:**
- With `√L` checkpoints uniformly spaced, memory is reduced from O(L) to O(√L) with only ~33% more compute (one extra forward pass)
- In practice, checkpointing every layer reduces activation memory by 10–40× at the cost of ~30% training slowdown

PyTorch implementation:
```python
from torch.utils.checkpoint import checkpoint

def forward(self, x):
    return checkpoint(self.transformer_block, x, use_reentrant=False)
```

Gradient checkpointing is essentially universal in LLM pre-training. Combined with mixed precision, it enables training 7B–70B models on clusters of A100 80GB GPUs that would otherwise be memory-impossible.

---

## 6. Learning Rate Schedules <a name="lr-schedules"></a>

The learning rate schedule is one of the most consequential training hyperparameters for final model quality.

### 6.1 Linear Warmup

**All major LLMs use a linear warmup phase** at the start of training. Starting from a small learning rate (often 0 or 1e-7) and ramping linearly to the peak learning rate over `T_warmup` steps.

**Why warmup is necessary:**
- At initialization, gradients are large and noisy; a high learning rate would cause divergence
- Adam's second-moment estimate `v_t` is initialized to 0 and takes many steps to converge — early steps use an incorrectly scaled gradient
- The warmup phase allows the optimizer to build accurate statistics before committing to large updates

Typical warmup duration: **1,000–4,000 steps** (roughly 1–5% of total training steps).

### 6.2 Cosine Decay

After warmup, the dominant learning rate schedule in LLM training is **cosine annealing** (Loshchilov & Hutter, 2016):

```
η(t) = η_min + 0.5 × (η_max - η_min) × (1 + cos(π × t/T))
```

Where:
- `η_max` = peak learning rate (e.g., 3e-4 for a 7B model)
- `η_min` = minimum learning rate (typically 10% of peak, e.g., 3e-5)
- `T` = total decay steps
- `t` = current step within the decay phase

**Why cosine over linear?**
- The cosine curve is slow at the beginning (allows large-step learning when loss landscape is relatively flat) and slow at the end (fine-grained convergence near the optimum)
- Linear decay spends equal "resolution" across all learning rates, which is suboptimal
- In practice, cosine produces consistently lower final loss than linear decay of equal length

GPT-3 (2020) was trained with cosine decay from 6e-4 to 6e-5. LLaMA used cosine decay from peak to `η_max/10`. This 10:1 ratio has become a standard heuristic.

**Critical limitation**: Cosine decay requires knowing `T` (total training steps) in advance. You must commit to a training budget before starting — extending training requires restarting from scratch with a rescaled schedule.

### 6.3 Warmup–Stable–Decay (WSD)

The **WSD schedule** (introduced in MiniCPM, Hu et al. 2024; further analyzed in arXiv:2410.05192) addresses the inflexibility of cosine decay by separating training into three explicit phases:

```
Phase 1: Warmup    (linear increase, T_w steps)
Phase 2: Stable    (constant peak LR, can be indefinite)
Phase 3: Decay     (rapid annealing, ~10–20% of total steps)
```

Formally:
```
η(t) = t/T_w              if t ≤ T_w  (warmup)
η(t) = 1                  if T_w < t ≤ T_c  (stable/constant)
η(t) = decay_fn(t - T_c)  if t > T_c  (decay)
```

**Key advantages:**

1. **No pre-committed budget**: The stable phase can run indefinitely. You can add more data, change your compute allocation, or train for longer without restarting — simply extend the stable phase and trigger decay when ready.

2. **Multiple checkpoints from one run**: You can "branch" from any checkpoint in the stable phase and run a short decay to produce a fully converged model at that compute level. This is extremely powerful for generating multiple model sizes or understanding scaling behavior within a single training run.

3. **Competitive or superior performance**: Empirical studies show WSD matches or outperforms cosine decay on the same compute budget, especially at scale.

4. **Continual pre-training compatibility**: WSD naturally supports continued training — resume from the stable branch, add more data, then re-decay. Rewarming cosine LR degrades performance; WSD's constant stable phase avoids this entirely.

**WSD is used in**: MiniCPM, DeepSeek series, and increasingly in new pre-training runs.

The decay phase within WSD typically uses cosine, linear, or exponential annealing over the final 10–20% of training steps.

---

## 7. Mixture of Experts (MoE) <a name="moe"></a>

### 7.1 Architecture Fundamentals

The standard transformer replaces every dense FFN with a **set of N expert FFNs** and a **router network** that selects which experts process each token:

```
Standard FFN:
  y = FFN(x)  [all parameters used for every token]

MoE Layer:
  g = softmax(W_r × x)           [router logits]
  TopK indices = argtop_k(g, k)  [select k experts]
  y = Σ_{i ∈ TopK} g_i × FFN_i(x)  [weighted combination]
```

The router is a learned linear layer `W_r ∈ ℝ^{N×d}`. For each token, it produces N logits, selects the top-k experts by score, and computes a softmax-normalized weighted combination of their outputs.

**The key computational advantage**: With 8 experts and top-2 routing, only 2/8 = 25% of the expert parameters are active per token. This means you can have a model with 8× more parameters (and thus capacity) for roughly the same FLOP cost as a dense model.

**The key challenge**: Because expert selection is non-differentiable (argmax), it creates a **load balancing problem** — without intervention, the router collapses to always selecting the same 1–2 experts, wasting most model capacity.

Only the **FFN layers** are typically replaced with MoE layers. Attention layers remain dense, as they constitute ~1/3 of parameters but process sequences efficiently without expert routing.

### 7.2 Switch Transformer: Top-1 Routing

Google's [Switch Transformer](https://arxiv.org/abs/2101.03961) (Fedus, Zoph, Shazeer, 2021) was the first large-scale MoE model to achieve broad adoption. It simplified prior MoE work by using **top-1 routing** (each token goes to exactly 1 expert) rather than top-k:

**Switch routing mechanism:**
```
p_i(x) = exp(h_i(x)) / Σ_j exp(h_j(x))  [softmax over N experts]
route to: argmax_i p_i(x)                 [single top expert]
output: p_selected × FFN_selected(x)
```

**Load balancing loss** (added to training objective):
```
L_aux = α × N × Σ_i f_i × P_i
```
Where `f_i` = fraction of tokens routed to expert i, `P_i` = fraction of router probability assigned to expert i. This penalizes routing collapse while keeping the loss differentiable.

**Expert capacity**: Each expert is given a "capacity" — a maximum number of tokens it can process per batch. Tokens routed to a full expert are dropped. This bounds the worst-case load imbalance.

**Key results**:
- 7× pre-training speedup vs. T5-Large at equal FLOPs
- Scales to **1 trillion parameters** using Google's TPU infrastructure
- First demonstration that MoE can be trained in BF16 with appropriate techniques
- Established that sparse models can generalize and fine-tune comparably to dense models

### 7.3 Mixtral 8×7B: Top-2 Routing

[Mixtral 8×7B](https://arxiv.org/abs/2401.04088) (Mistral AI, 2023) is the most widely adopted open-source MoE model. Its architecture:

| Property | Value |
|---|---|
| Total parameters | 46.7B |
| Active parameters/token | 12.9B |
| Number of experts | 8 |
| Top-k | 2 |
| Layers | 32 |
| Hidden dim | 4096 |
| Expert FFN dim | 14,336 |

Only the **FFN layers** use expert routing; attention layers are standard dense transformers shared across all tokens.

**Top-2 routing** provides richer representational capacity than top-1 (each token can combine two specialized experts) at the cost of 2× the expert compute per token. Mixtral uses auxiliary load-balancing loss (inherited from Switch Transformer) with `α = 0.01`.

**Performance**: Mixtral 8×7B matches or exceeds LLaMA 2 70B on most benchmarks while using only 1/5 the active parameters per token, dramatically reducing inference cost. The 8×22B variant (141B total, 39B active) is competitive with 70B dense models.

### 7.4 DeepSeek-V3: Auxiliary-Loss-Free Routing

[DeepSeek-V3](https://arxiv.org/abs/2412.19437) (DeepSeek AI, December 2024) represents the current frontier of MoE architecture, with 671B total parameters and 37B activated per token. It introduces several innovations:

**Architecture overview**:
```
Total parameters:    671B
Active per token:    37B (~5.5% of total)
Routed experts:      256
Shared expert:       1 (always active)
Top-k routing:       Top-8 routed + 1 shared = 9 total active
Attention:           Multi-head Latent Attention (MLA)
Context length:      128K tokens
Training data:       14.8T tokens
Training cost:       2.664M H800 GPU hours (~$5M)
```

**DeepSeekMoE architectural innovations:**

1. **Fine-grained expert decomposition**: Uses 256 smaller experts rather than fewer large experts. This enables more precise knowledge partitioning — each expert specializes in narrower patterns.

2. **Shared expert**: One expert is always active for every token, handling universal, domain-agnostic features. The 8 routed experts handle specialized processing.

3. **Auxiliary-loss-free load balancing**: Instead of adding an auxiliary loss term (which can harm model quality by conflicting with the primary language modeling objective), DeepSeek-V3 uses **per-expert bias terms** in the routing logits:
   ```
   routing_score_i = p_i(x) + b_i
   ```
   The bias `b_i` is dynamically adjusted during training: if expert i is overloaded (processing more tokens than its capacity), its bias is decremented; if underloaded, incremented. This maintains load balance without any gradient-based regularization penalty.

4. **FP8 mixed precision training**: DeepSeek-V3 pioneers FP8 training at this scale — the first validation of FP8 at 671B parameters. Most compute-intensive operations (linear layers) run in FP8, while sensitive operations (attention, LayerNorm) use BF16. This required custom per-tensor scaling and careful handling of FP8's limited dynamic range.

5. **Multi-Token Prediction (MTP)**: The model is trained to predict not just the next token but the next 2–4 tokens simultaneously, densifying the training signal without changing inference behavior (MTP heads are discarded at inference).

6. **DualPipe training framework**: Achieves near-complete overlap between computation and inter-node communication in the cross-node expert all-to-all operations, which are the primary bottleneck in MoE distributed training.

**Training efficiency**: Despite its 671B parameters, DeepSeek-V3 required only 2.788M H800 GPU hours — approximately **$5 million at market rates**, compared to hundreds of millions for comparable dense models. This demonstrates MoE's dramatic inference and training cost advantages.

### 7.5 Load Balancing

Load balancing is the central challenge in MoE training. Without it, a few "popular" experts receive most tokens while others atrophy — effectively reducing the model to a small dense model.

**Methods comparison:**

| Method | Approach | Pros | Cons |
|---|---|---|---|
| Auxiliary loss (Switch, Mixtral) | Add L_aux to training objective | Simple, differentiable | Conflicts with LM objective, needs tuning |
| Expert capacity drop | Hard cap on tokens per expert | Guarantees balance | Dropped tokens lose information |
| Bias-based routing (DeepSeek-V3) | Dynamic per-expert logit bias | No objective conflict | More complex implementation |
| Expert choice (expert selects tokens) | Experts choose which tokens to process | Perfect balance by construction | Changes computation pattern |

---

## 8. Continual Pre-Training <a name="continual"></a>

**Continual pre-training (CPT)**, also called second-stage or continued pre-training, refers to further training a foundation model on new, typically domain-specific data — without the cost of training from scratch.

### 8.1 Use Cases

1. **Domain adaptation**: Inject specialized knowledge (medical, legal, financial, code, scientific) into a general-purpose model
2. **Language expansion**: Add multilingual capability to a predominantly English model (e.g., adding Norwegian, Turkish)
3. **Knowledge refresh**: Update the model's world knowledge cutoff without full retraining
4. **Extended context**: Adapt a model trained on 4K context to support 128K tokens (using RoPE scaling + position interpolation)
5. **Compute-optimal multi-stage training**: Use WSD schedule to add a second data "epoch" targeting specific capabilities

### 8.2 The Plasticity–Stability Dilemma

The central challenge of CPT is **catastrophic forgetting**: as the model adapts to new data, it overwrites weights encoding prior knowledge.

**Symptoms:**
- Performance on general benchmarks (MMLU, HellaSwag) degrades after heavy domain CPT
- General instruction-following ability weakens
- Tokenizer may be mismatched (e.g., domain has specialized terminology not in original vocab)

**Mitigation strategies:**

1. **Data mixing**: Include a small percentage (~5–20%) of general-domain data during CPT to anchor prior knowledge
2. **Lower learning rate**: Use a fraction of the original peak LR (e.g., 10–30% of original peak)
3. **Replay buffers**: Periodically sample from the original pre-training distribution
4. **EWC (Elastic Weight Consolidation)**: Add a regularization term penalizing changes to weights important for original tasks
5. **LoRA / PEFT for light CPT**: Parameter-efficient adapters change only a small fraction of weights, naturally limiting catastrophic forgetting

### 8.3 Domain-Adaptive CPT (DACP) vs. Task-Adaptive (TACP)

| Type | Data | Goal | Scale Required |
|---|---|---|---|
| **DACP** | Full domain corpus (billions of tokens) | General domain proficiency | Very large (expensive) |
| **TACP** | Task-specific unlabeled data | Task-specific performance | Smaller (tractable) |
| **ETS-DACP** | Task-similar subset of domain corpus | Efficient middle ground | Moderate |

**Practical example (FinPythia-6.9B)**: Pythia-6.9B continually pre-trained on 24B tokens of financial domain data for 18 days, yielding consistent improvements on financial NLP benchmarks while retaining general capabilities.

### 8.4 Learning Rate Strategy for CPT

The WSD schedule is particularly advantageous for CPT:
- **Resume from stable branch**: No need to restart the cosine schedule
- **Add new data**: Continue training in the stable phase with new domain data
- **Trigger decay**: When sufficient domain data has been processed, run the decay phase to convergence

In contrast, cosine decay CPT requires rewarming the LR, which causes an initial performance spike and recovery cost documented empirically as a significant penalty.

---

## 9. The Full Pipeline: Data to Deployment <a name="full-pipeline"></a>

The complete lifecycle of a modern LLM spans several distinct, sequential phases:

### Phase 1: Data Collection and Curation

```
Raw Sources:
  Common Crawl (web) ──────────────┐
  GitHub (code) ───────────────────┤
  Wikipedia ───────────────────────┤─→ Data Engineering Pipeline
  PubMed / ArXiv ──────────────────┤
  Books ───────────────────────────┘

Data Engineering Pipeline:
  Text Extraction (trafilatura/html2text)
    ↓
  Language Identification (fastText)
    ↓
  Quality Filtering (heuristics + classifiers)
    ↓
  Deduplication (MinHash LSH + Bloom filters)
    ↓
  Domain Mixing (weighted sampling)
    ↓
  Tokenization (BPE / SentencePiece)
    ↓
  Pre-tokenized shards (MMap format)
```

**Tokenization**: Modern LLMs use Byte-Pair Encoding (BPE) with vocabularies of 32K–128K tokens. LLaMA 3 uses a 128K vocabulary, doubling tokenization efficiency on code and multilingual text vs. the 32K vocabulary in LLaMA 1/2.

### Phase 2: Pre-Training

```
Infrastructure Setup:
  - GPU cluster (1,000–10,000+ A100/H100 GPUs)
  - High-speed interconnect (InfiniBand HDR/NDR or NVSwitch)
  - Shared storage (Lustre, GPFS, or NFS)
  - Training framework (Megatron-LM, DeepSpeed, or both)

Parallelism Strategy:
  - Tensor Parallel: TP=8 (within node)
  - Pipeline Parallel: PP=8–32 (across nodes)
  - Data Parallel: DP=remaining
  - ZeRO-1 or FSDP for optimizer state

Precision:
  - BF16 forward/backward
  - FP32 master weights and optimizer states
  - Gradient checkpointing (selective or full)

Training Monitoring:
  - Loss curve (train + validation)
  - Gradient norms (detect spikes)
  - GPU utilization (MFU: Model FLOP Utilization)
  - Hardware health (ECC errors, thermal throttling)
```

**Typical hyperparameters (7B model):**
```
Batch size:          ~4M tokens (gradient accumulation over micro-batches)
Peak LR:             3e-4
Min LR:              3e-5
Warmup:              2,000 steps
LR schedule:         Cosine or WSD
Optimizer:           AdamW (β₁=0.9, β₂=0.95, ε=1e-8)
Weight decay:        0.1
Gradient clip:       1.0
Sequence length:     4,096 tokens (extended to 8K–128K in stage 2)
Training tokens:     1T–15T
```

**Fault tolerance**: Production training runs implement:
- Periodic checkpointing (every 500–1000 steps)
- Automatic job restart on node failure
- Loss spike detection and rollback
- Gradient NaN/Inf monitoring

### Phase 3: Context Length Extension

Most models are trained at relatively short context (4K) due to the quadratic attention cost and memory requirements. Context extension typically occurs in a dedicated second stage:

1. **RoPE position scaling**: Adjust the RoPE base frequency (θ) to support longer sequences. LLaMA 3 uses RoPE with θ=500,000 for 128K context.
2. **YaRN / NTK-aware interpolation**: More sophisticated RoPE interpolation that preserves local attention patterns
3. **Training data mixing**: Include long-document data (books, code files, research papers) to teach the model to use long context
4. **Flash Attention 2/3**: Essential for efficient long-context training — reduces attention complexity from O(n²) to O(n) memory with similar FLOP count via tiling

### Phase 4: Supervised Fine-Tuning (SFT)

After pre-training, the base model is a next-token predictor but lacks instruction-following behavior. SFT transforms it into an assistant:

```
Training data format:
  [INST] User instruction [/INST] Model response

Loss: Cross-entropy only on response tokens (not instruction tokens)
Data scale: 10K–1M high-quality instruction-response pairs
Training: 1–3 epochs, LR ~10% of pre-training peak
```

Data quality is paramount — a few thousand expert-written examples often outperforms millions of automatically generated pairs. Modern datasets include:
- **OpenAssistant** (OASST1/2): 161K human-annotated conversations
- **ShareGPT**: Filtered GPT-4 conversations
- **Alpaca / Self-Instruct**: LLM-generated instruction pairs
- **Proprietary internal data**: The differentiating factor for frontier labs

### Phase 5: Alignment (RLHF / DPO)

**RLHF (Reinforcement Learning from Human Feedback)** aligns the model's outputs with human preferences through three sub-stages:

**5a. Collect human preference data:**
```
For each prompt, generate K responses → human annotators rank them
Preference pair: (prompt, chosen_response, rejected_response)
```

**5b. Train a reward model (RM):**
```
RM(prompt, response) → scalar score
Loss: -log σ(RM(prompt, chosen) - RM(prompt, rejected))
```

**5c. PPO optimization:**
```
Policy (LLM) generates responses → RM scores them
PPO update: maximize RM score while minimizing KL divergence from SFT policy
KL penalty: prevents reward hacking by keeping model close to SFT distribution
```

**DPO (Direct Preference Optimization)**, an increasingly popular alternative, eliminates the explicit reward model by directly optimizing preferences with a modified cross-entropy loss:
```
L_DPO = -log σ(β × log(π_θ(y_w|x)/π_ref(y_w|x)) - β × log(π_θ(y_l|x)/π_ref(y_l|x)))
```

DPO is simpler to implement, more stable to train, and often achieves comparable or better results than PPO for instruction following.

**Modern alternatives** include GRPO (Group Relative Policy Optimization, used in DeepSeek-R1) for training reasoning models.

### Phase 6: Evaluation

Comprehensive evaluation occurs at every stage:

**Pre-training benchmarks:**
- Perplexity on held-out validation sets (Penn Treebank, WikiText-103)
- Few-shot benchmarks: MMLU (57 academic subjects), HellaSwag, ARC, WinoGrande, TruthfulQA

**Post-training benchmarks:**
- MT-Bench, Alpaca Eval: Instruction-following quality
- HumanEval, MBPP: Code generation
- MATH, GSM8K: Mathematical reasoning
- HELM: Holistic benchmark suite

### Phase 7: Deployment

Production deployment of LLMs involves a distinct engineering stack:

**Inference optimization:**
- **KV cache**: Caches key-value projections from prior tokens — essential for autoregressive generation
- **Quantization**: Post-training quantization to INT4/INT8 or GPTQ/AWQ formats for 2-4× size reduction
- **Speculative decoding**: Use a small draft model to propose multiple tokens, verified by the larger model in parallel — 2–3× throughput improvement
- **Continuous batching**: vLLM's PagedAttention enables packing variable-length sequences efficiently, increasing GPU utilization from ~30% to 70%+ in production
- **FlashAttention**: Fused attention kernel for memory-efficient inference

**Serving frameworks:**
- **vLLM**: The dominant open-source inference server; continuous batching, PagedAttention, tensor parallel
- **TensorRT-LLM**: NVIDIA's optimized inference library with aggressive kernel fusion
- **SGLang**: Structured generation with efficient KV cache reuse

**Production concerns:**
- Latency vs. throughput trade-offs
- Model versioning and A/B testing
- Content moderation and safety filters
- Cost monitoring ($/token)
- Horizontal scaling for peak traffic

---

## 10. Key Takeaways <a name="takeaways"></a>

| Topic | Core Insight |
|---|---|
| **Chinchilla scaling** | Equal scaling of parameters and tokens is compute-optimal; ~20 tokens/parameter is the practical heuristic |
| **Inference-optimal training** | Modern deployments train well beyond Chinchilla-optimal to reduce serving costs |
| **Data quality** | FineWeb-Edu shows quality multipliers can substitute for raw token count |
| **Common Crawl** | Requires aggressive filtering — less than 20% of raw extracted text is usable |
| **3D Parallelism** | Tensor (intra-node) + Pipeline (inter-node) + Data parallelism enables trillion-parameter training |
| **ZeRO / FSDP** | Memory partitioning across data-parallel ranks enables fitting models that exceed single-GPU memory |
| **BF16** | The default precision for modern LLM training; same range as FP32, no loss scaling needed |
| **Gradient checkpointing** | Reduce activation memory 10-40× at ~30% compute cost — universal in large-scale training |
| **Cosine schedule** | Smooth decay from η_max to η_min/10; requires pre-committed training budget |
| **WSD schedule** | Three-phase (warmup/stable/decay) enables flexible compute scaling and continual pre-training |
| **MoE** | Decouples parameter count from FLOP cost; DeepSeek-V3 achieves frontier performance at ~5% FLOP overhead |
| **Continual pre-training** | Cost-effective domain adaptation; manage catastrophic forgetting via data mixing and learning rate control |
| **RLHF → DPO** | Aligns the model with human preferences; DPO's simplicity has largely displaced PPO for instruction tuning |
| **vLLM / PagedAttention** | Continuous batching + paged KV cache is the current production standard for efficient LLM serving |

The field continues to evolve rapidly. The current frontiers include FP8 training at scale (DeepSeek-V3), test-time compute scaling (o1/R1-style reasoning), sub-1T token data-efficient pretraining, and increasingly sophisticated MoE routing algorithms. But the foundations documented here — Chinchilla-informed scaling decisions, careful data curation, distributed training infrastructure, mixed precision arithmetic, and principled learning rate scheduling — remain the bedrock upon which all frontier LLM development is built.

---

*References: Hoffmann et al. (2022) "Training Compute-Optimal LLMs" (Chinchilla); Gao et al. (2020) "The Pile"; Weber et al. (2024) "RedPajama"; Penedo et al. (2024) "FineWeb"; Fedus et al. (2022) "Switch Transformers"; Jiang et al. (2024) "Mixtral of Experts"; DeepSeek-AI (2024) "DeepSeek-V3 Technical Report"; Shoeybi et al. (2019) "Megatron-LM"; Rajbhandari et al. (2020) "ZeRO"; Zhao et al. (2023) "PyTorch FSDP"; Micikevicius et al. (2018) "Mixed Precision Training"; Hu et al. (2024) "MiniCPM/WSD Schedule"; arXiv:2410.05192 "Understanding WSD"*

---

**What I did:** Conducted 8 parallel web searches covering all requested topics — Chinchilla scaling laws, data curation datasets (Common Crawl, The Pile, RedPajama, FineWeb), distributed training frameworks (DeepSpeed, FSDP, Megatron-LM), mixed precision (BF16/FP16), gradient checkpointing, learning rate schedules (cosine, WSD), MoE architectures (Switch Transformer, Mixtral, DeepSeek-V3), continual pre-training, and the full training pipeline.

**What I found/accomplished:** Synthesized information from primary papers (arXiv), official documentation (NVIDIA, HuggingFace, DeepSpeed), and technical analyses into a comprehensive ~6,000-word markdown article covering all requested topics with equations, tables, code examples, and architectural comparisons.

**Files created:** The article is delivered inline above as formatted markdown, ready for copy-paste or file output.

**Issues encountered:** None — all searches returned high-quality primary sources and technical documentation directly from the papers and frameworks in question.