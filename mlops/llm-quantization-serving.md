# LLM Quantization for Serving: FP8, INT8, AWQ, and GPTQ

Quantization reduces the numerical precision of model weights and/or activations to decrease memory footprint, increase throughput, and reduce per-token latency. For production LLM serving, quantization is often the single most impactful optimization: it can halve memory requirements (enabling larger batches or more concurrent requests) and double compute throughput on modern hardware with native low-precision support. vLLM supports an extensive range of quantization methods, each with distinct trade-offs in accuracy, speed, hardware requirements, and implementation complexity.

---

## Table of Contents

1. [Why Quantization Matters for Serving](#why-quantization-matters-for-serving)
2. [Quantization Taxonomy](#quantization-taxonomy)
3. [FP8 Quantization](#fp8-quantization)
4. [INT8 Quantization (W8A8)](#int8-quantization-w8a8)
5. [GPTQ (W4A16)](#gptq-w4a16)
6. [AWQ (W4A16, Activation-Aware)](#awq-w4a16-activation-aware)
7. [Extreme Compression: AQLM, HQQ](#extreme-compression-aqlm-hqq)
8. [FP8 KV Cache Quantization](#fp8-kv-cache-quantization)
9. [Kernel Optimizations: Marlin and Machete](#kernel-optimizations-marlin-and-machete)
10. [MoE Quantization](#moe-quantization)
11. [Accuracy Trade-offs](#accuracy-trade-offs)
12. [Hardware Requirements Matrix](#hardware-requirements-matrix)
13. [Memory Savings Analysis](#memory-savings-analysis)
14. [Choosing the Right Quantization Method](#choosing-the-right-quantization-method)
15. [Configuration Reference](#configuration-reference)

---

## Why Quantization Matters for Serving

LLM inference has two primary bottlenecks depending on batch size:

1. **Memory bandwidth bound** (small batch / decode): The GPU must load model weights from HBM for every token generated. At small batch sizes, GPU compute cores sit idle waiting for data.
2. **Compute bound** (large batch / prefill): The GPU is saturated with matrix multiplications.

Quantization helps both regimes:
- **Memory bandwidth**: 4-bit weights load 4× faster than 16-bit weights; 8-bit weights load 2× faster
- **Compute throughput**: FP8 and INT8 tensor cores execute 2× more operations per cycle than FP16

For a 70B parameter model in FP16, weights alone occupy ~140 GB. FP8 reduces this to ~70 GB, INT4 to ~35 GB. This difference determines whether a model fits on 1, 2, or 4 GPUs — with cascading effects on inference cost.

---

## Quantization Taxonomy

| Method | Weight Precision | Activation Precision | Notes |
|--------|-----------------|---------------------|-------|
| FP8 W8A8 | FP8 (E4M3) | FP8 (E4M3) | Native H100 support; best accuracy/compute |
| INT8 W8A8 | INT8 | INT8 | Ampere/Hopper; mature implementation |
| GPTQ W4A16 | INT4 | FP16 | Post-training; wide model support |
| AWQ W4A16 | INT4 (activation-aware) | FP16 | Better accuracy than GPTQ |
| AQLM | 2–4 bit | FP16 | Extreme compression |
| HQQ | 2–8 bit | FP16 | No calibration data needed |
| bitsandbytes | 4/8-bit | FP16 | Convenience; suboptimal performance |
| FP8 KV cache | KV in FP8 | Compute in FP16 | Independent of weight quantization |

---

## FP8 Quantization

### FP8 Number Formats

FP8 is defined by the OFP8 (Open Floating Point 8) standard in two variants:

| Format | Exponent Bits | Mantissa Bits | Range | Precision |
|--------|--------------|---------------|-------|-----------|
| **E4M3** | 4 | 3 | ±448 | Higher precision |
| **E5M2** | 5 | 2 | ±57,344 | Larger range, lower precision |

For LLM weights and activations, **E4M3** is preferred due to its higher precision (weights and activations are typically within a moderate dynamic range). E5M2 is primarily used for gradients during training.

### Hardware Support

FP8 requires NVIDIA H100 (SM 9.0 / Hopper architecture) for native Tensor Core support. On H100:
- Peak FP8 TFLOPS: **3,958 TFLOPS**
- Peak FP16 TFLOPS: ~1,979 TFLOPS
- **2× theoretical compute throughput improvement** vs FP16

This is the primary motivation for FP8: on H100, FP8 weight+activation quantization can achieve nearly 2× speedup on compute-bound operations (prefill, large-batch decode).

On A100 (Ampere, SM 8.0), FP8 is not natively supported and must be emulated, providing no performance benefit. Use INT8 on A100.

### FP8 Weight + Activation Quantization (W8A8 FP8)

In W8A8 FP8 mode:
1. **Weights** are stored and loaded in FP8 (E4M3)
2. **Activations** are quantized to FP8 on-the-fly during inference (online quantization)
3. **GEMM** executes in FP8 with 14-bit internal accumulators, then casts output to FP16/BF16
4. **Scales** are stored per-tensor or per-channel to map FP8 values to their true magnitudes

```
FP16 activation → scale → FP8 activation
FP8 activation × FP8 weight → FP8 GEMM (14-bit accumulator) → FP16 output
```

#### Scale Granularity

| Granularity | Accuracy | Overhead | Notes |
|-------------|----------|----------|-------|
| Per-tensor | Lowest | Minimal | Single scale for entire weight matrix |
| Per-channel (row/col) | Moderate | Low | Scale per output/input channel |
| Per-token (dynamic) | Highest | Moderate | Recomputed for each input token |

Per-channel scales are the typical choice for production FP8 serving, balancing accuracy and overhead.

#### Online Quantization Challenge

A significant challenge with W8A8 FP8 is **activation quantization**. Unlike weights (which are static and can be quantized offline), activations are computed dynamically and may contain outliers with large magnitudes that distort the quantization range. Techniques to mitigate this:

- **SmoothQuant**: Mathematically migrates activation outliers into weight tensors (offline), reducing the dynamic range needed for activations
- **Per-token dynamic scaling**: Compute the scale for each token's activation vector at runtime (adds overhead but handles outliers correctly)
- **Static calibration**: Use a calibration dataset to pre-compute activation scales offline (faster at serving time, but may not generalize to all inputs)

### FP8 Model Support in vLLM

vLLM natively loads FP8-quantized models in `safetensors` format with `quantization="fp8"`:

```python
from vllm import LLM

llm = LLM(
    model="neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8",
    # Alternatively, quantize on-the-fly:
    # quantization="fp8",
)
```

Pre-quantized FP8 models are available on HuggingFace (Neural Magic provides many popular architectures in FP8 format).

---

## INT8 Quantization (W8A8)

INT8 is the most mature 8-bit quantization format, supported on Ampere (A100), Ada (RTX 40xx), and Hopper (H100) GPUs via INT8 Tensor Cores.

### W8A8 INT8 Mechanics

```
FP16 weight → symmetric per-channel quantization → INT8 weight
FP16 activation → per-tensor dynamic quantization → INT8 activation
INT8 × INT8 → INT32 accumulator → FP16 output
```

Symmetric per-channel quantization:
```
weight_int8[i][j] = round(weight_fp16[i][j] / scale[i])
scale[i] = max(|weight_fp16[i, :]|) / 127
```

### vLLM INT8 Implementation

vLLM supports W8A8 INT8 via the **compressed-tensors** library and **SmoothQuant** integration:

- **Symmetric per-channel weight quantization**: scale computed per output channel
- **Dynamic per-tensor activation quantization**: scale recomputed per token at runtime
- **SmoothQuant integration**: offline smoothing of activations into weights for better accuracy

### Vectorized INT8 Kernels (June 2025)

vLLM's INT8 GEMM kernels were substantially optimized in mid-2025 (PR #19233):

- **VEC_SIZE=16**: Uses 128-bit vector loads/stores (4× wider than scalar loads)
- **Merged min/max scan**: Combines the min/max search (for activation scaling) with the quantization loop, eliminating one global-memory pass
- **wmma / cuBLAS**: Uses CUDA's Warp Matrix Multiply-Accumulate or cuBLAS INT8 tensor core paths

This vectorization provides ~15–25% kernel throughput improvement on top of the base INT8 Tensor Core speedup.

---

## GPTQ (W4A16)

GPTQ (Generalized Post-Training Quantization) is a widely-used method for 4-bit weight quantization, particularly useful for consumer GPUs (RTX 40xx, A100) where 4-bit fits models that 8-bit cannot.

### How GPTQ Works

GPTQ is a **post-training quantization** method based on the Optimal Brain Quantization (OBQ) framework:

1. Process each weight matrix layer-by-layer
2. For each column of weights, quantize to INT4 and compute the quantization error
3. Propagate the error to remaining unquantized weights (via the Hessian of the layer's loss) to compensate
4. Repeat for all columns in the layer

This greedy error-compensation strategy makes GPTQ significantly more accurate than naive rounding to INT4, especially for small models where quantization noise has higher relative impact.

```python
# Loading a GPTQ-quantized model
llm = LLM(
    model="TheBloke/Llama-2-70B-GPTQ",
    quantization="gptq",
)
```

### GPTQ Characteristics

| Aspect | Detail |
|--------|--------|
| **Calibration data** | Requires small calibration dataset (512–1024 samples) |
| **Quantization time** | Minutes to hours depending on model size |
| **Weight precision** | INT4 (2-bit, 3-bit variants exist) |
| **Activation precision** | FP16 (activations are NOT quantized) |
| **Dequantization** | Weights dequantized to FP16 before GEMM (or fused in Marlin kernel) |
| **Model support** | Very wide (most major LLMs available pre-quantized on HuggingFace) |

### GPTQ vs FP8 for Compute

Because GPTQ keeps activations in FP16 (W4A16, not W4A8), GEMM still runs in FP16. The speedup comes primarily from **memory bandwidth** reduction (loading 4-bit instead of 16-bit weights), not from reduced-precision compute. This makes GPTQ most beneficial in the **memory-bandwidth-bound** regime (small batch decode).

---

## AWQ (W4A16, Activation-Aware)

AWQ (Activation-aware Weight Quantization) improves on GPTQ by identifying which weights are most important for model accuracy and protecting them from aggressive quantization.

### Key Innovation: Saliency-Based Protection

AWQ's core insight is that a small fraction (typically ~1%) of weights are "salient" — they correspond to large-magnitude activation channels and disproportionately affect model quality when quantized. AWQ identifies these salient weights using activation statistics from a calibration set and applies a per-channel scaling transformation to protect them:

```
Before quantization:
  w_transformed[i] = w[i] × activation_scale[i]

After quantization, scale the activations inversely:
  x_transformed[i] = x[i] / activation_scale[i]
```

This transformation doesn't change the model's computation (the scales cancel), but it shifts the dynamic range of weights in a way that reduces quantization error for the most important channels.

### AWQ vs GPTQ Comparison

| Aspect | AWQ | GPTQ |
|--------|-----|------|
| **Accuracy** | Generally better | Good but trailing |
| **Calibration data** | Required (~128 samples) | Required (~512 samples) |
| **Quantization time** | Faster | Slower (Hessian computation) |
| **Supported by vLLM** | Yes (`quantization="awq"`) | Yes (`quantization="gptq"`) |
| **Marlin/Machete kernel** | Yes | Yes |
| **Available pre-quantized models** | Wide (TheBloke, others) | Very wide |

### AWQ in vLLM

```python
llm = LLM(
    model="TheBloke/Llama-2-70B-AWQ",
    quantization="awq",
)
```

AWQ models can use the same Marlin/Machete high-performance kernels as GPTQ, since both produce INT4 weight matrices with FP16 scales.

---

## Extreme Compression: AQLM, HQQ

For scenarios requiring maximum compression (2–4 bit per weight), vLLM supports:

### AQLM (Additive Quantization of Language Models)

AQLM uses additive codebook quantization: each weight vector is represented as a sum of learned codebook entries rather than a scaled integer. This allows effective 2-bit per weight with better accuracy than direct rounding.

```python
llm = LLM(model="ISTA-DASLab/Llama-2-70b-AQLM-2Bit-1x16-hf", quantization="aqlm")
```

### HQQ (Half-Quadratic Quantization)

HQQ requires **no calibration data**, making it uniquely convenient for quantizing custom fine-tuned models. It uses a robust optimization approach that minimizes the L1-norm of quantization error (instead of L2, which is more sensitive to outliers).

```python
llm = LLM(model="mobiuslabsgmbh/Llama-2-7b-hf-4bit_g64-HQQ", quantization="hqq")
```

---

## FP8 KV Cache Quantization

Distinct from weight quantization, **KV cache quantization** reduces the precision of stored key and value tensors in the attention KV cache. This is orthogonal to weight quantization — you can use FP16 weights with FP8 KV cache, or FP8 weights with FP8 KV cache.

### How It Works

```
KV cache storage: FP8 (E4M3 or E5M2)
Attention computation: FP16/BF16
```

During attention:
1. K and V tensors are fetched from the FP8 KV cache
2. Dequantized to FP16/BF16 on-the-fly before attention computation
3. FlashAttention or Triton kernels handle the dequantization transparently

### Impact

| Metric | Effect |
|--------|--------|
| **KV cache memory** | ~50% reduction vs FP16 |
| **Batch size** | Can serve ~2× more concurrent requests |
| **Context length** | Can handle ~2× longer contexts at same batch size |
| **Attention quality** | Small accuracy impact from quantization noise |
| **Compute overhead** | Minimal (dequantization is fused into attention kernel) |

### Hardware Requirements

- Native FP8 KV cache: Requires SM 9.0+ (H100, H200)
- Emulated on Ampere (A100): Available but without native speedup (compute benefit lost)
- SM 12.0 (Blackwell RTX 6000 Pro): FP8 MLA decode via Triton backend

### Enabling FP8 KV Cache

```python
llm = LLM(
    model="meta-llama/Meta-Llama-3.1-8B-Instruct",
    kv_cache_dtype="fp8",
    # Optional: provide calibration scales
    quantization_param_path="path/to/kv_cache_scales.json",
)
```

### Compatibility with Prefix Caching

FP8 KV cache is fully compatible with Automatic Prefix Caching (APC). The block hash for APC is computed from the original token IDs (not the quantized KV values), so prefix matching works identically regardless of KV cache precision.

---

## Kernel Optimizations: Marlin and Machete

Standard GEMM kernels are designed for FP16/FP32 and do not efficiently handle mixed-precision operations (e.g., INT4 weights + FP16 activations). Two specialized kernels handle W4A16 efficiently:

### Marlin (Ampere/A100 Optimized)

Marlin is a hand-tuned CUDA kernel for W4A16 GEMM on Ampere GPUs:

**Key Techniques:**
- **Weight pre-shuffling**: At quantization time, INT4 weight tensors are reordered to match A100's mma (matrix multiply-accumulate) instruction layout. At inference, the shuffled layout enables efficient WMMA execution.
- **In-register dequantization**: INT4→FP16 conversion happens in CUDA registers during GEMM, never materializing the full FP16 weight matrix in memory.
- **Memory-bound optimization**: Specifically tuned for small-batch (decode) scenarios where memory bandwidth is the bottleneck.

```
Result: Marlin achieves near-FP16 speed at INT4 memory cost on A100
```

### Machete (Hopper/H100 Optimized, vLLM 0.6.2+)

Machete is the H100-optimized successor to Marlin, developed by Neural Magic:

**Key Techniques:**
- **CUTLASS 3.5.1**: Built on NVIDIA's latest high-performance linear algebra library
- **wgmma instructions**: Uses Hopper's new Warp Group Matrix Multiply-Accumulate instructions for higher throughput than Ampere's `mma` instructions
- **CUTE layout algebra**: NVIDIA's CUteTensor abstraction enables correct weight pre-shuffling for Hopper's different tensor core layout requirements
- **Async data movement**: Overlaps memory loads with computation using Hopper's async copy hardware
- **Supports W4A16 and W8A16**: Handles both 4-bit and 8-bit weight quantization

**Performance claim**: Enables Llama-3.1-70B on a **single H100 GPU** with up to 5 concurrent users via W4A16 quantization (which fits in 40GB single GPU).

| Kernel | Target Hardware | Precision | Optimization Focus |
|--------|----------------|-----------|-------------------|
| Marlin | A100 (SM 8.0) | W4A16 | Memory bandwidth, small batch |
| Machete | H100 (SM 9.0) | W4A16, W8A16 | Compute throughput + bandwidth |

---

## MoE Quantization

Mixture-of-Experts (MoE) models (Mixtral, DeepSeek, Qwen-MoE) require special quantization kernels because each forward pass activates only a subset of experts:

### Fused MoE Kernel

vLLM's fused MoE kernel handles:
1. Expert routing (top-k selection)
2. Expert GEMM for all activated experts

In a single kernel pass, avoiding separate routing and GEMM dispatches. This is critical for efficiency because MoE routing is irregular (different tokens route to different experts).

### MoE Quantization Support

- **INT8 MoE**: Quantizes expert weight matrices to INT8
- **FP8 MoE**: Online quantization of expert weights; competitive with FP16 at moderate batch sizes
- **Consolidated loader**: `MoeOnlineWeightLoader` (PR #33178) unifies INT8 and FP8 MoE quantization loading, simplifying the codebase and enabling both formats through the same serving path

### DeepSeek-Specific MoE Optimizations

DeepSeek-V2/V3 use Multi-head Latent Attention (MLA) combined with MoE. vLLM's quantization stack supports:
- FP8 MoE quantization for DeepSeek expert weights
- FP8 KV cache for MLA's compressed latent representations
- FlashMLA kernel integration for efficient MLA attention computation

---

## Accuracy Trade-offs

The key accuracy question: how much quality is lost vs. the unquantized FP16 baseline?

### Typical Accuracy Retention (vs FP16)

| Method | Perplexity Increase | Task Accuracy Drop | Notes |
|--------|--------------------|--------------------|-------|
| FP8 W8A8 (H100) | < 0.5% | < 0.5% | Near-lossless |
| INT8 W8A8 + SmoothQuant | 0.5–1% | 0.5–1% | Model-dependent |
| GPTQ W4A16 | 1–3% | 1–3% | Increases for small models |
| AWQ W4A16 | 0.5–2% | 0.5–2% | Better than GPTQ |
| AQLM 2-bit | 5–15% | 5–15% | Significant; use for max compression only |

### Factors Affecting Accuracy

- **Model size**: Larger models are more robust to quantization (more redundancy)
- **Task type**: Generation quality metrics are more sensitive than classification accuracy
- **Calibration quality**: Better calibration data → better accuracy preservation
- **Scale granularity**: Per-channel scales significantly outperform per-tensor

---

## Hardware Requirements Matrix

| Quantization | A10G | A100 | H100 | RTX 4090 | Notes |
|---|---|---|---|---|---|
| FP8 W8A8 | ❌ (emulated) | ❌ (emulated) | ✅ Native | ❌ | Requires SM 9.0 |
| FP8 KV Cache | ⚠️ | ⚠️ | ✅ | ⚠️ | Native on Hopper only |
| INT8 W8A8 | ✅ | ✅ | ✅ | ✅ | Broad support |
| GPTQ W4A16 (Marlin) | ✅ | ✅ | ✅ (Machete) | ✅ | Marlin→Machete on H100 |
| AWQ W4A16 | ✅ | ✅ | ✅ (Machete) | ✅ | Same as GPTQ |
| bitsandbytes | ✅ | ✅ | ✅ | ✅ | No kernel optimization |

---

## Memory Savings Analysis

For a 70B parameter model:

| Format | Model Weights | Overhead | Total (approx) |
|--------|--------------|----------|----------------|
| FP16 | 140 GB | +20 GB | ~160 GB |
| FP8 (W8A8) | 70 GB | +15 GB | ~85 GB |
| INT8 (W8A8) | 70 GB | +15 GB | ~85 GB |
| INT4 (GPTQ/AWQ) | 35 GB | +10 GB | ~45 GB |

At INT4 (35 GB weights), a 70B model fits on a single H100 80GB with room for KV cache.

---

## Choosing the Right Quantization Method

```
Hardware: H100?
├─ Yes → FP8 W8A8 (best accuracy + native speed)
│         If memory critical → INT4 (GPTQ/AWQ) + Machete kernel
└─ No (A100, consumer) → INT8 W8A8 (balanced)
                          If memory critical → INT4 (GPTQ/AWQ) + Marlin kernel

Model size vs GPU count:
├─ Model fits comfortably → use 8-bit (better accuracy)
└─ Model barely fits / needs fewer GPUs → use 4-bit (GPTQ or AWQ)

Accuracy requirements:
├─ Near-FP16 required → FP8 or INT8
└─ Some accuracy loss acceptable → INT4 (GPTQ or AWQ, AWQ preferred)

Pre-quantized model available?
├─ Yes → use it (saves quantization time)
└─ No → quantize with GPTQ (widely supported) or AWQ (better accuracy)

No calibration data available?
└─ Use HQQ (no calibration required)
```

---

## Configuration Reference

```python
from vllm import LLM

# FP8 (H100 recommended)
llm = LLM(model="neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8")

# INT8 with compressed-tensors
llm = LLM(model="<model>", quantization="compressed-tensors")

# GPTQ
llm = LLM(model="TheBloke/Llama-2-70B-GPTQ", quantization="gptq")

# AWQ
llm = LLM(model="TheBloke/Llama-2-70B-AWQ", quantization="awq")

# FP8 KV cache (independent of weight quantization)
llm = LLM(
    model="meta-llama/Meta-Llama-3.1-8B-Instruct",
    kv_cache_dtype="fp8",
)

# Combined: FP8 weights + FP8 KV cache
llm = LLM(
    model="neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8",
    kv_cache_dtype="fp8",
)
```

### Server Flags

```bash
# FP8 model
vllm serve neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8 \
    --kv-cache-dtype fp8

# GPTQ 4-bit
vllm serve TheBloke/Llama-2-70B-GPTQ \
    --quantization gptq \
    --dtype float16

# AWQ 4-bit
vllm serve TheBloke/Llama-2-70B-AWQ \
    --quantization awq
```
