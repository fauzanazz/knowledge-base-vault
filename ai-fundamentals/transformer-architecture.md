# The Transformer Architecture: A Comprehensive Technical Guide

## Table of Contents

1. [Introduction & Historical Context](#introduction)
2. [The Original "Attention is All You Need" Architecture](#original-architecture)
3. [Scaled Dot-Product & Multi-Head Attention](#attention-mechanisms)
4. [Encoder-Decoder Structure](#encoder-decoder)
5. [Positional Encoding: Sinusoidal, RoPE, and ALiBi](#positional-encoding)
6. [Layer Normalization: Pre-Norm vs Post-Norm](#layer-normalization)
7. [Feed-Forward Networks](#ffn)
8. [Residual Connections](#residuals)
9. [Embedding Layers](#embeddings)
10. [Tokenization: BPE, SentencePiece, and tiktoken](#tokenization)
11. [Architectural Evolution: Decoder-Only (GPT) and Encoder-Only (BERT)](#evolution)
12. [Modern Architectures: LLaMA, Mistral, and Gemma](#modern)
13. [Summary Comparison Table](#comparison)

---

## 1. Introduction & Historical Context <a name="introduction"></a>

Before 2017, the dominant paradigm for sequence-to-sequence tasks — machine translation, summarization, speech recognition — relied on **Recurrent Neural Networks (RNNs)**, particularly Long Short-Term Memory (LSTM) networks and Gated Recurrent Units (GRUs). These models processed input sequentially, one token at a time, which created a fundamental bottleneck: they couldn't be parallelized during training, and long-range dependencies were hard to maintain as the gradient signal had to flow through every time step.

Attention mechanisms were introduced as an enhancement to RNN encoder-decoder architectures (Bahdanau et al., 2015), allowing the decoder to "look back" at all encoder hidden states rather than relying solely on a fixed-length context vector. This was a breakthrough, but attention was still bolted onto a recurrent backbone.

In June 2017, Vaswani et al. published **"Attention is All You Need"** at NeurIPS, proposing a radical simplification: discard recurrence entirely and build a model *solely* from attention mechanisms. The result — the **Transformer** — could be fully parallelized during training, scaled efficiently to large datasets, and achieved state-of-the-art performance on machine translation benchmarks. It became the architectural foundation for every major language model that followed: BERT, GPT, T5, LLaMA, Gemini, Claude, and beyond.

---

## 2. The Original "Attention is All You Need" Architecture <a name="original-architecture"></a>

### Overview

The original Transformer was designed for **sequence transduction** (specifically English-to-German and English-to-French translation). It follows the classic **encoder-decoder** paradigm, but replaces all recurrent and convolutional layers with stacked self-attention and feed-forward layers.

**Key hyperparameters (base model):**
- Model dimension: `d_model = 512`
- Number of attention heads: `h = 8`
- Head dimension: `d_k = d_v = d_model / h = 64`
- Number of encoder/decoder layers: `N = 6`
- Feed-forward inner dimension: `d_ff = 2048`
- Dropout: `p = 0.1`
- Total parameters: ~65 million (base), ~213 million (large)

### High-Level Architecture

```
Input Tokens
    │
    ▼
[Token Embedding + Positional Encoding]
    │
    ▼
┌─────────────────────────────────┐
│         ENCODER STACK           │
│  ╔═══════════════════════════╗  │
│  ║  Multi-Head Self-Attention║  │
│  ║  + Add & LayerNorm        ║  │
│  ║  Feed-Forward Network     ║  │
│  ║  + Add & LayerNorm        ║  │
│  ╚═══════════════════════════╝  │
│       × N = 6 layers            │
└─────────────────────────────────┘
    │ (encoder output)
    │
    ▼
[Target Token Embedding + Positional Encoding]
    │
    ▼
┌─────────────────────────────────┐
│         DECODER STACK           │
│  ╔═══════════════════════════╗  │
│  ║  Masked Multi-Head Attn   ║  │
│  ║  + Add & LayerNorm        ║  │
│  ║  Cross-Attention (enc→dec)║  │
│  ║  + Add & LayerNorm        ║  │
│  ║  Feed-Forward Network     ║  │
│  ║  + Add & LayerNorm        ║  │
│  ╚═══════════════════════════╝  │
│       × N = 6 layers            │
└─────────────────────────────────┘
    │
    ▼
[Linear + Softmax → Vocabulary Distribution]
```

---

## 3. Scaled Dot-Product & Multi-Head Attention <a name="attention-mechanisms"></a>

### 3.1 Scaled Dot-Product Attention

The core computational primitive of the Transformer is **Scaled Dot-Product Attention**. Given a matrix of queries **Q**, keys **K**, and values **V**:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Where:
- **Q** ∈ ℝ^(n × d_k) — query matrix (n = sequence length)
- **K** ∈ ℝ^(m × d_k) — key matrix (m = source length)
- **V** ∈ ℝ^(m × d_v) — value matrix
- `√d_k` — scaling factor to prevent dot products from growing large in magnitude and pushing softmax into saturation regions

**Why the scaling factor?** The dot product Q·K^T has variance proportional to `d_k`. Without scaling, for large `d_k`, softmax outputs become extremely peaked (nearly one-hot), producing vanishingly small gradients. Dividing by `√d_k` keeps the variance of the pre-softmax logits at unit scale.

**Masking in the Decoder:** During autoregressive decoding, the decoder's self-attention applies a **causal mask** — a triangular matrix that sets future positions to `-∞` before the softmax, ensuring token `i` can only attend to tokens `0...i`:

```
Mask[i, j] = { 0      if j <= i
             { -∞     if j > i
```

This means `softmax(masked_logits)` assigns zero weight to future tokens, preserving the autoregressive property.

### 3.2 Multi-Head Attention

Rather than computing a single attention function, the Transformer uses **multi-head attention** — `h` parallel attention heads that each project Q, K, V into lower-dimensional subspaces via learned linear projections:

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) \cdot W^O$$

where each head is:

$$\text{head}_i = \text{Attention}(Q W_i^Q, K W_i^K, V W_i^V)$$

with projection matrices:
- $W_i^Q \in \mathbb{R}^{d_{\text{model}} \times d_k}$
- $W_i^K \in \mathbb{R}^{d_{\text{model}} \times d_k}$
- $W_i^V \in \mathbb{R}^{d_{\text{model}} \times d_v}$
- $W^O \in \mathbb{R}^{hd_v \times d_{\text{model}}}$

**Why multiple heads?** Each head can attend to different aspects of the input — syntactic relationships, semantic roles, coreference, local vs. long-range dependencies — simultaneously. The original paper uses `h = 8` heads with `d_k = d_v = 64`, keeping total computation equivalent to a single-head attention with full dimensionality.

### 3.3 Three Types of Attention in the Original Transformer

| Location | Query Source | Key/Value Source | Type |
|---|---|---|---|
| Encoder self-attention | Encoder layer `l` | Encoder layer `l` | Bidirectional self-attention |
| Decoder masked self-attention | Decoder layer `l` | Decoder layer `l` (past only) | Causal self-attention |
| Decoder cross-attention | Decoder layer `l` | Final encoder output | Cross-attention |

**Cross-attention** is the mechanism by which the decoder "reads" the encoded source — the decoder's queries attend over all encoder key-value pairs at each decoder layer.

---

## 4. Encoder-Decoder Structure <a name="encoder-decoder"></a>

### 4.1 The Encoder

The encoder is a stack of **N = 6** identical layers. Each layer contains:

1. **Multi-Head Self-Attention** sublayer (bidirectional — every token can attend to every other)
2. **Position-wise Feed-Forward Network** sublayer
3. **Residual connections** around each sublayer followed by **Layer Normalization**

Formally, for encoder layer `l`:

$$\mathbf{h}^{(l)}_{\text{attn}} = \text{LayerNorm}\left(\mathbf{h}^{(l-1)} + \text{MultiHead}\left(\mathbf{h}^{(l-1)}, \mathbf{h}^{(l-1)}, \mathbf{h}^{(l-1)}\right)\right)$$

$$\mathbf{h}^{(l)} = \text{LayerNorm}\left(\mathbf{h}^{(l)}_{\text{attn}} + \text{FFN}\left(\mathbf{h}^{(l)}_{\text{attn}}\right)\right)$$

The encoder's final output is a continuous representation **z** ∈ ℝ^(n × d_model) encoding the full context of the input sequence.

### 4.2 The Decoder

The decoder is also a stack of **N = 6** identical layers, but each layer has *three* sublayers:

1. **Masked Multi-Head Self-Attention** — attends to previously generated tokens only (causal mask)
2. **Multi-Head Cross-Attention** — attends to the encoder output (queries from decoder, keys/values from encoder)
3. **Position-wise Feed-Forward Network**

Each sublayer is wrapped with a residual connection and LayerNorm.

### 4.3 The Output Head

After the final decoder layer, a **linear projection** maps from `d_model` → vocabulary size, followed by **softmax** to produce a probability distribution over the next token:

$$P(\text{next token}) = \text{softmax}(W_{\text{lm}} \cdot \mathbf{h}_{\text{decoder}})$$

where $W_{\text{lm}} \in \mathbb{R}^{d_{\text{model}} \times |V|}$. In practice, these weights are often **tied** to the input embedding matrix (weight tying), reducing parameters and improving performance.

---

## 5. Positional Encoding <a name="positional-encoding"></a>

Since the Transformer has no built-in notion of sequence order (unlike RNNs), positional information must be explicitly injected. Three major approaches have emerged over the years.

### 5.1 Sinusoidal Positional Encoding (Vaswani et al., 2017)

The original Transformer uses fixed (non-learned) sinusoidal functions to encode absolute position. For position `pos` and dimension `i`:

$$PE_{(pos,\, 2i)} = \sin\left(\frac{pos}{10000^{2i / d_{\text{model}}}}\right)$$

$$PE_{(pos,\, 2i+1)} = \cos\left(\frac{pos}{10000^{2i / d_{\text{model}}}}\right)$$

Each dimension `i` corresponds to a sinusoid with a different **wavelength**:
- Dimension 0: wavelength ≈ 2π (very short)
- Dimension `d_model/2`: wavelength ≈ 10000 · 2π (very long)

**Key properties:**
- Every position gets a unique encoding vector
- Relative positions can be expressed as linear combinations: PE(pos + k) = f(PE(pos)) for some fixed linear function
- Works for any sequence length at inference (no out-of-vocabulary position problem)
- No learned parameters

**Limitation:** Positional information is "added" to the semantic embedding, polluting the semantic space with absolute position information. This creates entanglement between content and position that doesn't generalize cleanly to different sequence lengths.

```python
def sinusoidal_encoding(seq_len: int, d_model: int) -> np.ndarray:
    positions = np.arange(seq_len)[:, np.newaxis]        # (seq_len, 1)
    dims      = np.arange(d_model)[np.newaxis, :]        # (1, d_model)
    angles    = positions / 10000 ** (2 * (dims // 2) / d_model)
    pe = np.where(dims % 2 == 0, np.sin(angles), np.cos(angles))
    return pe  # (seq_len, d_model)
```

### 5.2 Rotary Position Embedding (RoPE) — Su et al., 2021

RoPE is the de facto standard in modern LLMs (LLaMA, Mistral, Gemma, Qwen, DeepSeek). It starts from a precise mathematical requirement: we want the query-key dot product to depend only on **content** and **relative position** — never on absolute positions individually.

**Mathematical formulation.** For a 2D query/key vector (x₁, x₂) at position m, RoPE applies a rotation by angle mθ:

$$R(\theta, m) = \begin{pmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{pmatrix}$$

The rotated query and key are:
$$\mathbf{q}_m = R(\theta, m) \cdot W_Q \mathbf{x}_m$$
$$\mathbf{k}_n = R(\theta, n) \cdot W_K \mathbf{x}_n$$

The dot product then satisfies:
$$\mathbf{q}_m^T \mathbf{k}_n = (W_Q \mathbf{x}_m)^T R(\theta, n - m)^T (W_K \mathbf{x}_n)$$

**Key insight:** $R(\theta, m)^T R(\theta, n) = R(\theta, n - m)$, so the inner product depends only on the **relative distance** `n - m`. Absolute position information cancels out.

**Full-dimensional extension.** For a `d`-dimensional vector, RoPE divides dimensions into `d/2` pairs, each with its own frequency:

$$\theta_i = 10000^{-2i/d}, \quad i = 0, 1, \ldots, \frac{d}{2} - 1$$

This creates a geometric sequence of frequencies from ≈1 (short-range) to ≈0.0001 (very long-range), providing multi-scale positional information.

**Efficient implementation** avoids full matrix multiplication using an elementwise trick:

$$\text{RoPE}(\mathbf{x}, m) = \mathbf{x} \odot \cos(m\boldsymbol{\theta}) + \text{rotate\_half}(\mathbf{x}) \odot \sin(m\boldsymbol{\theta})$$

where `rotate_half` shuffles pairs of dimensions.

**Advantages over sinusoidal:**
- Decouples position from semantic content (no direct addition to embedding)
- Naturally captures relative positions
- Enables context length extension via **YaRN**, **rope_scaling**, etc.
- No extra parameters

### 5.3 Attention with Linear Biases (ALiBi) — Press et al., 2022

ALiBi takes a radical minimalist approach: **don't add positional encoding to the embeddings at all**. Instead, directly bias the attention scores before softmax with a penalty proportional to distance:

$$\text{Attention score}_{i,j} = \mathbf{q}_i \cdot \mathbf{k}_j - m \cdot |i - j|$$

where `m` is a head-specific slope. For `n` heads, slopes form a geometric sequence:

$$m_k = 2^{-8k/n}, \quad k = 1, 2, \ldots, n$$

For 8 heads: m ∈ {1/2, 1/4, 1/8, 1/16, 1/32, 1/64, 1/128, 1/256}

Different heads thus have different "locality biases" — some strongly prefer nearby tokens, others are more global.

**Key properties:**
- Zero learned parameters, zero modification to Q and K
- Strong extrapolation: a model trained on 1024 tokens extrapolates to 2048 tokens with no quality degradation
- Simple and computationally cheap
- Used in BLOOM and MPT

**Comparison of positional encoding methods:**

| Feature | Sinusoidal | RoPE | ALiBi |
|---|---|---|---|
| Position type | Absolute | Relative (via rotation) | Relative (via linear bias) |
| Parameters | Zero | Zero | Zero |
| Where injected | Input embeddings | Q and K vectors | Attention logits |
| Length extrapolation | Weak | Moderate (with scaling) | Strong |
| Used in | Original Transformer, BERT | LLaMA, Mistral, Gemma | BLOOM, MPT |

---

## 6. Layer Normalization: Pre-Norm vs. Post-Norm <a name="layer-normalization"></a>

### 6.1 Layer Normalization (Ba et al., 2016)

LayerNorm normalizes activations across the **feature dimension** of each individual sample (unlike BatchNorm, which normalizes across the batch dimension). For a vector **x** ∈ ℝ^d:

$$\text{LayerNorm}(\mathbf{x}) = \gamma \odot \frac{\mathbf{x} - \mu(\mathbf{x})}{\sqrt{\sigma^2(\mathbf{x}) + \epsilon}} + \beta$$

where:
- $\mu(\mathbf{x}) = \frac{1}{d}\sum_{i=1}^d x_i$ — mean
- $\sigma^2(\mathbf{x}) = \frac{1}{d}\sum_{i=1}^d (x_i - \mu)^2$ — variance
- $\gamma, \beta \in \mathbb{R}^d$ — learnable scale and shift parameters
- $\epsilon$ — small constant for numerical stability (typically 1e-5 or 1e-6)

LayerNorm is preferred over BatchNorm in Transformers because it operates independently per sample (no batch dependency) and does not require cross-device synchronization in distributed training.

### 6.2 RMSNorm — Root Mean Square Normalization (Zhang & Sennrich, 2019)

RMSNorm asks: *do we actually need the mean-centering step?* It simplifies LayerNorm by removing the mean subtraction:

$$\text{RMSNorm}(\mathbf{x}) = \gamma \odot \frac{\mathbf{x}}{\sqrt{\frac{1}{d}\sum_{i=1}^d x_i^2 + \epsilon}}$$

The denominator is the **root mean square** of the input vector. This:
- Removes the bias parameter β (half the learnable parameters of LayerNorm)
- Is ~7–64% faster in practice due to fewer computations
- Works equally well when hidden states have near-zero mean (which is common after many layers)

RMSNorm is used by **LLaMA, Mistral, Gemma, PaLM, Gopher**, and most modern open-source LLMs.

### 6.3 Post-Norm (Original Transformer)

The original "Attention is All You Need" paper used **Post-LN** — normalization applied *after* the sublayer computation and residual addition:

$$\mathbf{h}' = \text{LayerNorm}(\mathbf{h} + \text{Sublayer}(\mathbf{h}))$$

```
x ──────────────────┐
  └──► Sublayer ──► + ──► LayerNorm ──► output
```

**Problem with Post-Norm:** Applying LayerNorm after the residual connection breaks the identity path. The gradient flowing back through the LayerNorm gets rescaled, and in very deep networks, early layers can receive vanishingly small or exploding gradients. In practice, Post-LN requires very careful learning rate warmup schedules to train stably.

### 6.4 Pre-Norm (Modern Standard)

**Pre-LN** applies normalization *before* the sublayer — leaving the residual connection as a clean identity path:

$$\mathbf{h}' = \mathbf{h} + \text{Sublayer}(\text{LayerNorm}(\mathbf{h}))$$

```
x ──────────────────────────────────────┐
  └──► LayerNorm ──► Sublayer ──► + ──► output
```

**Why Pre-Norm trains better:**
- The residual connection is an **unobstructed identity highway** — gradients flow directly back through the main branch
- Gradient norms are more stable across depth — no warmup required
- Enables training much deeper models reliably

**Pre-Norm in practice:** GPT-2 was the first major model to switch to Pre-LN, adding an additional LayerNorm after the final self-attention block. Since then, virtually all modern LLMs (GPT-3, LLaMA, Falcon, Mistral, Gemma) use Pre-Norm.

**Tradeoff:** Pre-Norm models can suffer from **representation degeneration** — because the residual path is always preserved, some layers may learn to "do nothing," reducing effective depth. Post-Norm often achieves slightly higher peak performance when it converges, but convergence is far less reliable.

```python
# Post-Norm (original)
def post_norm_block(x, attn, ffn, norm1, norm2):
    x = norm1(x + attn(x))
    x = norm2(x + ffn(x))
    return x

# Pre-Norm (modern)
def pre_norm_block(x, attn, ffn, norm1, norm2):
    x = x + attn(norm1(x))
    x = x + ffn(norm2(x))
    return x
```

---

## 7. Feed-Forward Networks <a name="ffn"></a>

### 7.1 Original FFN (ReLU)

Each Transformer layer includes a **Position-wise Feed-Forward Network** applied identically to each position. In the original paper:

$$\text{FFN}(\mathbf{x}) = \text{ReLU}(\mathbf{x} W_1 + \mathbf{b}_1) W_2 + \mathbf{b}_2$$

Where:
- $W_1 \in \mathbb{R}^{d_{\text{model}} \times d_{\text{ff}}}$ — first projection (expand)
- $W_2 \in \mathbb{R}^{d_{\text{ff}} \times d_{\text{model}}}$ — second projection (contract)
- $d_{\text{ff}} = 4 \times d_{\text{model}}$ (typically) — the 4× expansion ratio

This is often called a "two-layer MLP with a bottleneck." The FFN accounts for roughly two-thirds of the total parameters in a Transformer model.

### 7.2 GELU Activation

GPT-2 replaced ReLU with **GELU** (Gaussian Error Linear Unit):

$$\text{GELU}(x) = x \cdot \Phi(x) \approx 0.5x\left(1 + \tanh\left(\sqrt{\frac{2}{\pi}}(x + 0.044715 x^3)\right)\right)$$

GELU is smoother than ReLU (no hard zero gradient for negative values) and empirically performs better for language modeling.

### 7.3 SwiGLU (LLaMA, Mistral, and Most Modern LLMs)

**SwiGLU** (Shazeer, 2020) is a **Gated Linear Unit** variant using the Swish activation. It replaces the standard FFN with a three-matrix gated design:

$$\text{SwiGLU}(\mathbf{x}) = \left(\text{Swish}(\mathbf{x} W_{\text{gate}}) \odot (\mathbf{x} W_{\text{up}})\right) W_{\text{down}}$$

where $\text{Swish}(x) = x \cdot \sigma(x)$ (also called SiLU).

The gate mechanism $\text{Swish}(\mathbf{x} W_{\text{gate}})$ acts as a **learnable filter** on the input features, allowing the network to selectively route information. This improves expressivity relative to a standard two-matrix FFN.

**Parameter budget:** Since SwiGLU uses 3 weight matrices instead of 2, LLaMA sets $d_{\text{ff}} = \frac{2}{3} \times 4 \times d_{\text{model}}$ to keep total parameters constant.

### 7.4 GeGLU (Gemma)

**GeGLU** replaces Swish with GELU in the gate:

$$\text{GeGLU}(\mathbf{x}) = \text{GELU}(\mathbf{x} W_{\text{gate}}) \odot (\mathbf{x} W_{\text{up}})$$

Used by Gemma models.

**FFN activation function timeline:**

| Model | Activation | Notes |
|---|---|---|
| Original Transformer (2017) | ReLU | Simple, fast, non-smooth |
| BERT (2018) | GELU | Smoother, better gradient flow |
| GPT-2 (2019) | GELU | |
| T5 (2020) | ReLU | |
| PaLM (2022) | SwiGLU | |
| LLaMA 1/2/3 (2023–2024) | SwiGLU | |
| Mistral (2023) | SwiGLU | |
| Gemma (2024) | GeGLU | GELU-gated variant |

---

## 8. Residual Connections <a name="residuals"></a>

Residual connections (He et al., 2016 — from ResNet) are a core component of the Transformer. Every sublayer (attention or FFN) is wrapped with:

$$\mathbf{h}^{(\text{out})} = \mathbf{h}^{(\text{in})} + \text{Sublayer}(\cdots)$$

This forms an **identity skip connection** that bypasses the sublayer's computation.

**Why they are essential:**

1. **Gradient flow:** Gradients can backpropagate through the identity path directly from output to input, bypassing the sublayer entirely. This prevents gradient vanishing in deep networks.

2. **Learning incremental refinements:** Rather than learning a complete transformation, each sublayer learns a *residual* — a small correction to the existing representation. This is significantly easier to optimize.

3. **Representation geometry:** The residual stream can be interpreted as a "main information highway" that persists throughout the model, with each layer contributing small updates. This view (the **residual stream** interpretation in mechanistic interpretability) has been productive for understanding how Transformers work internally.

4. **Depth scaling:** Residual connections are what allow stacking 6, 12, 96, or even hundreds of layers without training collapse.

**Mathematical intuition:** In the limit of many infinitesimally small residual steps, the stack of residual layers approximates a continuous-time dynamical system (Neural ODE interpretation), smoothly evolving the representation from input to output.

---

## 9. Embedding Layers <a name="embeddings"></a>

### 9.1 Token Embeddings

The embedding layer is a learned lookup table $E \in \mathbb{R}^{|V| \times d_{\text{model}}}$ that maps integer token IDs to dense vectors:

$$\mathbf{e}_t = E[t] \in \mathbb{R}^{d_{\text{model}}}$$

In the original Transformer, these embeddings are scaled by $\sqrt{d_{\text{model}}}$ before adding positional encodings, so that the positional signal (which has unit scale) doesn't dominate:

$$\mathbf{x}_t = \sqrt{d_{\text{model}}} \cdot E[t] + PE(t)$$

### 9.2 Weight Tying

In many models (including the original Transformer), the **input embedding matrix** is tied (shared) with the **output linear projection** before softmax. This means the same weights `E` are used to map tokens to embeddings *and* to map decoder states back to token probabilities:

$$P(\text{token} = t) = \text{softmax}(E \cdot \mathbf{h}_{\text{final}})$$

This reduces parameters significantly (for a vocab of 32,000 and `d_model = 512`, weight tying saves 32,000 × 512 = 16.4 million parameters) and improves learning because the model must learn consistent token representations in both directions.

### 9.3 Embedding Initialization

Embeddings are typically initialized from $\mathcal{N}(0, 1)$ or $\mathcal{N}(0, d_{\text{model}}^{-0.5})$ and trained end-to-end. The embedding space learns to encode semantic and syntactic properties of tokens, with semantically similar tokens mapping to nearby vectors.

---

## 10. Tokenization: BPE, SentencePiece, and tiktoken <a name="tokenization"></a>

Before any of the above can function, raw text must be converted into integer token IDs. This is the job of the **tokenizer**.

### 10.1 Byte Pair Encoding (BPE)

**BPE** (Sennrich et al., 2016) was originally a data compression algorithm adapted for NLP. It operates on characters (or bytes) and iteratively merges the most frequent adjacent pairs:

**Training algorithm:**
1. Initialize vocabulary with all individual characters (or bytes)
2. Count frequencies of all adjacent character pairs in the corpus
3. Merge the most frequent pair into a new "token" unit
4. Add the merged token to the vocabulary
5. Repeat steps 2–4 until vocabulary size reaches target (e.g., 50,000 or 100,000)

**Example:**
```
Initial:  l o w e r
After step 1 (merge 'l','o' → 'lo'):  lo w e r
After step 2 (merge 'lo','w' → 'low'):  low e r
...
Final token:  lower
```

**BPE at inference (encoding):**
Apply the learned merge rules greedily in order of their training priority. Unknown characters are handled by byte fallback (representing any byte as one of 256 base tokens).

**Advantages:**
- Compact representation of common words (single token)
- Graceful handling of rare words via subword decomposition
- Explicit control over vocabulary size
- Deterministic output

**Used in:** GPT-2, GPT-3, GPT-4 (via tiktoken), RoBERTa

### 10.2 WordPiece (BERT)

BERT uses **WordPiece**, a variant of BPE that maximizes the **likelihood** of the training data under a language model rather than merging purely based on frequency. Subwords from the middle/end of words are prefixed with `##` to indicate continuation (e.g., `"tokenization"` → `["token", "##ization"]`). WordPiece vocabulary for BERT-Base is 30,522 tokens.

### 10.3 SentencePiece (LLaMA, T5, Mistral 7B)

**SentencePiece** (Kudo & Richardson, 2018) is a tokenization framework with two key differences from standard BPE:

1. **Raw text input:** Trains directly on raw Unicode text, treating whitespace as a regular character (represented as `▁`). No pre-tokenization or language-specific preprocessing required.
2. **Language-agnostic:** Works equally well for English, Chinese, Japanese, Arabic — any language.

SentencePiece supports two algorithms:
- **BPE** (same merge-based approach, but on raw text)
- **Unigram Language Model** — starts with a large vocabulary and prunes tokens using an EM algorithm to maximize the unigram probability of the corpus via the Viterbi algorithm

**Unigram model:** assigns a probability to every possible segmentation and selects the maximum-probability segmentation:

$$\text{seg}^* = \arg\max_{\text{seg}} \prod_{t \in \text{seg}} P(t)$$

This allows multiple valid segmentations (useful for regularization during training).

**Used in:** LLaMA 1/2, Mistral 7B, T5, Gemma 1, many multilingual models

### 10.4 tiktoken (OpenAI)

**tiktoken** is OpenAI's high-performance BPE tokenizer library, implemented in Rust with a Python interface. It is ~2–3x faster than pure-Python BPE implementations.

**Key encodings:**
| Encoding | Vocabulary Size | Used by |
|---|---|---|
| `gpt2` | 50,257 | GPT-2 |
| `cl100k_base` | 100,421 | GPT-3.5, GPT-4, Claude (via custom variant) |
| `o200k_base` | ~200,000 | GPT-4o, GPT-4.1, o-series models |

The `cl100k_base` encoding uses a **regex pre-tokenization** step that splits on whitespace, punctuation, and common patterns before BPE merges, producing more consistent token boundaries.

**tiktoken vs SentencePiece:**
| Feature | tiktoken (OpenAI BPE) | SentencePiece |
|---|---|---|
| Training input | Pre-tokenized text | Raw text directly |
| Algorithm | BPE (merge-based) | BPE or Unigram LM |
| Implementation | Rust core, very fast | C++ |
| Whitespace | Preserved via regex | Encoded as special char `▁` |
| Custom training | Not supported | Full pipeline |
| Multilingual | Less flexible | Excellent |

### 10.5 Token Efficiency

| Text type | GPT-4 (cl100k) | GPT-4o (o200k) | LLaMA 3 |
|---|---|---|---|
| English prose (1k chars) | ~250 tokens | ~210 tokens | ~220 tokens |
| Python code (1k chars) | ~200 tokens | ~180 tokens | ~190 tokens |
| Chinese text (1k chars) | ~1000 tokens | ~350 tokens | ~320 tokens |

Larger, better-trained vocabularies (o200k_base, LLaMA 3's 128K vocab) yield more efficient representations, particularly for non-Latin scripts.

---

## 11. Architectural Evolution: Decoder-Only (GPT) and Encoder-Only (BERT) <a name="evolution"></a>

The original Transformer was an encoder-decoder for translation. Researchers quickly recognized that the encoder and decoder components could be used independently for different types of tasks.

### 11.1 Encoder-Only: BERT (Devlin et al., 2018)

**BERT** (Bidirectional Encoder Representations from Transformers) uses only the encoder stack with **bidirectional self-attention** — every token attends to every other token simultaneously. There is no causal mask.

**Architecture:**
- BERT-Base: 12 layers, 12 heads, d_model=768, d_ff=3072 (~110M params)
- BERT-Large: 24 layers, 16 heads, d_model=1024, d_ff=4096 (~340M params)
- Tokenizer: WordPiece (vocab size 30,522)
- Positional encoding: Learned absolute embeddings
- Normalization: Post-LN (original Transformer style)

**Pre-training tasks:**

1. **Masked Language Modeling (MLM):** 15% of tokens are randomly masked; the model must predict the original token from context. Unlike causal LM, the model sees *both left and right context*, learning bidirectional representations:
   ```
   Input:  "The [MASK] sat on the mat"
   Target: "cat" (for the masked position)
   ```

2. **Next Sentence Prediction (NSP):** Given two sentences A and B, predict whether B follows A in the original text. (Later research showed NSP adds little value — RoBERTa removed it.)

**Fine-tuning:** Add task-specific heads on top of the `[CLS]` token representation (classification) or individual token representations (NER, QA). BERT dominated NLP benchmarks from 2018–2022.

**BERT's strength:** Bidirectional context is ideal for **understanding** tasks — classification, NER, question answering, semantic similarity. The model sees the complete context when representing each token.

**BERT's limitation:** Cannot generate text natively. It's a representation model, not a generative model.

**Notable BERT variants:**
- **RoBERTa** (Liu et al., 2019): Removed NSP, used dynamic masking and more data — significantly better performance
- **ALBERT** (Lan et al., 2019): Parameter-sharing across layers + factorized embedding, greatly reduced parameters
- **DistilBERT**: Knowledge distillation to 6 layers, 60% of size, 97% of performance
- **DeBERTa** (He et al., 2020): Disentangled attention with separate position embeddings, significantly better performance

### 11.2 Decoder-Only: GPT Series (Radford et al., 2018–2023)

**GPT** (Generative Pre-trained Transformer) uses only the decoder stack with **causal (autoregressive) self-attention** — each token can only attend to itself and previous tokens.

**GPT-1 (2018):**
- 117M parameters
- 12 layers, 12 heads, d_model=768
- Context window: 512 tokens
- Pre-training: Causal language modeling on BookCorpus (~1B words)
- Tokenizer: BPE (40,000 vocabulary)
- Introduced the **pre-train then fine-tune** paradigm

**GPT-2 (2019):**
- 1.5B parameters (largest variant)
- 48 layers, 25 attention heads
- Context window: 1,024 tokens
- **Key architectural change:** Switched to **Pre-LN** (LayerNorm before each sublayer), added final LayerNorm after last attention block
- Demonstrated surprisingly capable **zero-shot** generation
- Tokenizer: BPE (50,257 vocabulary)

**GPT-3 (2020):**
- 175B parameters
- 96 layers, 96 heads, d_model=12288
- Context window: 2,048 tokens
- **Key discovery:** **In-context learning** — few-shot prompting enables new tasks without gradient updates
- Used **alternating dense and sparse attention** (local + global) in some layers

**The GPT decoder block (Pre-LN):**
```
x = x + MultiHeadCausalAttention(LayerNorm(x))
x = x + FFN(LayerNorm(x))
return x
```

**Why decoder-only became dominant:**
1. **Unified task framing:** Any task can be expressed as text generation (next token prediction)
2. **Scaling laws:** Decoder-only models exhibit smoother scaling behavior
3. **Simplicity:** No cross-attention mechanism reduces implementation complexity
4. **Emergent capabilities:** At sufficient scale, in-context learning, chain-of-thought reasoning, and instruction following emerge
5. **Training efficiency:** Single training objective (next token prediction) vs. multiple objectives for encoders

As of 2025, ~78% of open-source LLMs on Hugging Face are decoder-only.

### 11.3 Encoder-Decoder: T5 and BART

**T5** (Raffel et al., 2020) reframes all NLP tasks as text-to-text and uses the full encoder-decoder architecture. Key innovations:
- Replaced absolute positional encodings with relative position biases (scalar biases added to attention logits)
- Used RMSNorm-like normalization
- "span corruption" pre-training objective

**BART** (Lewis et al., 2019) pre-trains the full encoder-decoder by corrupting documents (masking, shuffling, deletion) and reconstructing them, excelling at generation tasks like summarization and translation.

---

## 12. Modern Architectures: LLaMA, Mistral, and Gemma <a name="modern"></a>

Modern open-source LLMs share a **decoder-only backbone** but incorporate several architectural improvements over the original GPT design. The "LLaMA recipe" (Pre-RMSNorm + SwiGLU + RoPE + GQA) has become the new default.

### 12.1 LLaMA (Meta AI, 2023–2024)

**LLaMA 1** (Touvron et al., 2023) introduced an efficient, high-quality open-source decoder-only model series. Key deviations from GPT:

| Component | GPT-3 | LLaMA 1 |
|---|---|---|
| Normalization | Post-LN, LayerNorm | Pre-RMSNorm |
| Positional encoding | Learned absolute | RoPE |
| FFN activation | GeLU | SwiGLU |
| Attention | Multi-Head Attention | Multi-Head Attention |
| Bias terms | Yes | No (except in QKV) |

**LLaMA 2** (2023) introduced **Grouped Query Attention (GQA)** for 34B and 70B models, and extended the context window from 2K to 4K tokens.

**LLaMA 3** (2024) significantly scaled up:
- Vocabulary: 128K tokens (tiktoken-based BPE)
- Context window: 8K (base), extended via RoPE scaling
- GQA for all sizes (8B: 8 KV heads for 32 query heads; 70B: 8 KV heads for 64 query heads)
- Training: ~15 trillion tokens

**LLaMA 3 70B Architecture:**
```
Layers:      80
Hidden dim:  8192
Query heads: 64
KV heads:    8  (8:1 ratio via GQA)
d_head:      128
FFN hidden:  28672
Vocab:       128K
```

#### Grouped Query Attention (GQA)

Standard **Multi-Head Attention (MHA)** has independent Q, K, V for every head. During inference, all past K and V must be cached (**KV cache**), consuming O(n × h × d_head) memory per layer.

**Multi-Query Attention (MQA)** — extreme variant: share a single K/V across all query heads. Reduces KV cache h× but hurts quality.

**GQA** is the middle ground: group query heads and share one K/V per group:

$$\text{GQA}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{Concat}_{g=1}^{G}\left[\text{Attention}(\mathbf{Q}_g, \mathbf{K}_g, \mathbf{V}_g)\right] W^O$$

For LLaMA 3 70B with 64 query heads and 8 KV heads, each KV head serves 8 query heads — reducing the KV cache by 8× with minimal quality loss.

### 12.2 Mistral (Mistral AI, 2023–2024)

**Mistral 7B** (Jiang et al., 2023) achieved remarkable performance for its size, outperforming LLaMA 2 13B on most benchmarks.

**Key innovations:**

**Sliding Window Attention (SWA):** Instead of full attention over the entire sequence, each token in SWA only attends to the `w` most recent tokens (window size `w = 4096` for Mistral 7B). This reduces attention complexity from O(n²) to O(n·w).

Crucially, through layer stacking, information can still propagate over long distances via the "receptive field" of each token, which grows with depth: after `k` layers, a token's effective receptive field is `k × w` tokens.

**Architecture (Mistral 7B):**
```
Layers:      32
Hidden dim:  4096
Query heads: 32
KV heads:    8   (GQA)
FFN hidden:  14336
Sliding win: 4096
Vocab:       32000 (SentencePiece)
```

**Mistral 7B also uses:**
- Pre-RMSNorm
- SwiGLU FFN
- RoPE positional encoding
- No bias terms

**Mixtral 8x7B** (Mistral AI, 2023) extended this with **Mixture-of-Experts (MoE)**, where the FFN is replaced by 8 expert networks, with 2 experts selected per token via a learned router:

$$\text{MoE}(\mathbf{x}) = \sum_{i=1}^{8} \text{Softmax}(\text{Top2}(\mathbf{x} W_g))_i \cdot \text{SwiGLU}_i(\mathbf{x})$$

This gives 46.7B total parameters but only 12.9B active per forward pass — combining high capacity with efficient computation.

**Mistral Small 3.1 (2025):** Abandoned sliding window attention in favor of full attention with a custom tokenizer optimized for efficiency.

### 12.3 Gemma (Google DeepMind, 2024)

**Gemma** is Google's family of open-weight decoder-only models, sharing architectural lineage with Gemini.

**Gemma 1 (2B, 7B) key features:**
- Multi-Query Attention (MQA) — extreme KV sharing
- **GeGLU activation** (GELU-based gated linear unit)
- RoPE positional encoding
- Large vocabulary: 256K tokens (supports multilingual)
- Pre-RMSNorm

**Gemma 2 (9B, 27B) innovations:**
- Switched from MQA to **GQA** (balanced tradeoff)
- Introduced **hybrid attention**: alternating local (sliding window, 4096 tokens) and global (full attention) layers in a 1:1 ratio
- **Logit soft-capping**: clamp attention logits to [-50, 50] via `tanh` before softmax — prevents extreme attention spikes
- Knowledge distillation from larger to smaller models during training

**Gemma 3 (2025) further innovations:**
- Adjusted hybrid ratio: 5 local layers per 1 global layer (5:1)
- Uses GQA with sliding window
- Pre-RMSNorm *and* Post-RMSNorm (both positions used simultaneously for better training dynamics)
- Context: 8K training, extends to 1M+ tokens via RoPE scaling at inference

**Gemma 2 27B Architecture:**
```
Layers:     46
Hidden dim: 4608
Query heads: 32
KV heads:    16 (GQA, 2:1 ratio)
FFN hidden:  36864
Sliding win: 4096 (local layers)
Vocab:       256K
```

### 12.4 Architecture Comparison: Modern LLMs

| Feature | Original Transformer | GPT-3 | LLaMA 3 8B | Mistral 7B | Gemma 2 9B |
|---|---|---|---|---|---|
| **Architecture** | Enc-Dec | Dec-only | Dec-only | Dec-only | Dec-only |
| **Normalization** | Post-LayerNorm | Pre-LayerNorm | Pre-RMSNorm | Pre-RMSNorm | Pre-RMSNorm |
| **Positional Enc.** | Sinusoidal | Learned abs. | RoPE | RoPE | RoPE |
| **FFN Activation** | ReLU | GeLU | SwiGLU | SwiGLU | GeGLU |
| **Attention Type** | MHA | MHA | GQA | GQA | GQA |
| **Attention Pattern** | Full | Full | Full | Sliding Window | Hybrid (local+global) |
| **Vocabulary Size** | ~37K | 50K | 128K | 32K | 256K |
| **Bias Terms** | Yes | Yes | No | No | Yes (selective) |

---

## 13. Summary Comparison Table <a name="comparison"></a>

### Attention Mechanism Evolution

```
MHA (2017) → MQA (2019) → GQA (2023) → MLA (2024, DeepSeek)
  ↑               ↑           ↑              ↑
 Full KV       Single KV   Grouped KV    Compressed KV
 per head     per model    per group      via low-rank
```

### Normalization Evolution

```
BatchNorm → LayerNorm (Post) → LayerNorm (Pre) → RMSNorm (Pre)
 (CNN era)    (Original Transformer)   (GPT-2+)       (LLaMA+)
```

### Positional Encoding Evolution

```
Sinusoidal → Learned Absolute → Relative → RoPE / ALiBi
  (2017)         (GPT)          (T5, XL)   (2021-2022)
```

### FFN Activation Evolution

```
ReLU → GeLU → SwiGLU/GeGLU (Gated Linear Units)
(2017)  (2019)     (2022+)
```

---

## Conclusion

The Transformer architecture, introduced in 2017 as a sequence transduction model for machine translation, has proven to be one of the most consequential architectural innovations in the history of AI. Its core insight — that attention mechanisms alone are sufficient for learning long-range dependencies in sequences — enabled unprecedented parallelism during training and led to the discovery of powerful **scaling laws**: performance improves predictably with model size, data, and compute.

The architecture has evolved significantly:

- **Positional encoding** moved from absolute sinusoidal patterns to relative rotation (RoPE) or linear attention biases (ALiBi), enabling better length generalization
- **Normalization** shifted from Post-LN to Pre-RMSNorm for training stability at scale
- **FFN activations** evolved from ReLU to gated variants (SwiGLU/GeGLU) for better representational capacity
- **Attention heads** evolved from full MHA to GQA for memory-efficient inference
- **Architecture variants** emerged for different use cases: encoder-only (BERT) for understanding, decoder-only (GPT) for generation, encoder-decoder (T5) for transduction

Modern LLMs like LLaMA 3, Mistral, and Gemma represent a refined consensus design: **Pre-RMSNorm + SwiGLU + RoPE + GQA in a decoder-only stack**, trained with causal language modeling at scale. This stack combines training stability, inference efficiency, and strong generalization — the foundation upon which GPT-4, Claude, Gemini, and every frontier model of today is built.

---

*Sources: Vaswani et al. (2017) "Attention is All You Need"; Devlin et al. (2018) "BERT"; Radford et al. (2018, 2019) GPT-1/2; Brown et al. (2020) GPT-3; Su et al. (2021) RoPE; Press et al. (2022) ALiBi; Touvron et al. (2023) LLaMA 1/2; Jiang et al. (2023) Mistral; Team et al. (2024) Gemma; Zhang & Sennrich (2019) RMSNorm; Xiong et al. (2020) Pre-LN Analysis.*

---

## ✅ Summary

**What I did:**
- Conducted 6 targeted web searches across all required topic areas simultaneously to gather technical depth
- Synthesized research into a ~6,500-word comprehensive markdown article

**What I found/accomplished:**
Produced a comprehensive markdown article covering all requested topics with mathematical rigor:

1. **Original Transformer** — full architecture with scaled dot-product attention formula, encoder-decoder stacks, hyperparameters
2. **Encoder-Decoder structure** — detailed breakdown of all three attention types, the output head, weight tying
3. **Positional Encoding** — complete mathematical formulations for Sinusoidal (PE equations), RoPE (rotation matrix derivation, relative position property proof), and ALiBi (slope formula, bias mechanism), with a comparison table
4. **Layer Normalization** — LayerNorm vs RMSNorm equations, Pre-Norm vs Post-Norm analysis with gradient flow explanation and code examples
5. **Feed-Forward Networks** — ReLU → GELU → SwiGLU → GeGLU progression with full equations
6. **Residual Connections** — mathematical role, gradient highway interpretation
7. **Tokenization** — BPE algorithm walkthrough, WordPiece (BERT), SentencePiece (Unigram LM), tiktoken with vocabulary/speed comparisons
8. **Embedding layers** — weight tying, scaling factor, initialization
9. **Architectural evolution** — BERT (MLM/NSP pre-training, bidirectional), GPT series (GPT-1/2/3 timeline with key changes), T5/BART
10. **Modern architectures** — LLaMA 3, Mistral 7B (sliding window attention), Gemma 2 (hybrid local/global attention, logit capping), GQA math, with full spec tables

**Files created/modified:** None (output provided directly as markdown)

**Issues encountered:** None — all searches returned high-quality technical sources including the original arXiv papers, academic write-ups, and engineering documentation.