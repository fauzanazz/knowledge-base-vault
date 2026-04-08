# vLLM Advanced Features

vLLM has evolved from a pure inference engine into a comprehensive serving platform with advanced features spanning structured output generation, multi-adapter serving, multimodal inference, tool calling, kernel-level compute optimizations, and a fully OpenAI-compatible API layer. This article covers the advanced capability set introduced across vLLM versions from 0.6 through V1 (2024–2026), focusing on production-relevant features for sophisticated LLM deployment scenarios.

---

## Table of Contents

1. [Structured Output / Guided Decoding](#structured-output--guided-decoding)
2. [LoRA Multi-Adapter Serving](#lora-multi-adapter-serving)
3. [Multimodal Support](#multimodal-support)
4. [Tool Calling and Function Calling](#tool-calling-and-function-calling)
5. [FlashAttention Integration](#flashattention-integration)
6. [CUDA Graph Optimization](#cuda-graph-optimization)
7. [OpenAI-Compatible API Features](#openai-compatible-api-features)
8. [vLLM V1 Architecture Overview](#vllm-v1-architecture-overview)
9. [Reasoning Model Support](#reasoning-model-support)
10. [MoE-Specific Optimizations](#moe-specific-optimizations)
11. [Disaggregated Prefill and KV Transfer](#disaggregated-prefill-and-kv-transfer)
12. [Observability and Deployment Features](#observability-and-deployment-features)

---

## Structured Output / Guided Decoding

Structured decoding ensures that LLM outputs conform to a specified schema by constraining the token sampling distribution at every generation step. Rather than filtering outputs after the fact, vLLM integrates constraint enforcement directly into the sampling pipeline.

### How Guided Decoding Works

At each decoding step, after the model computes logits over the vocabulary:

```
logits (size: vocab_size)
  → apply constraint mask (zero out invalid tokens)
  → softmax
  → sample
```

The mask at each step is computed by a constraint automaton that tracks the current generation state. The valid tokens are exactly those that can extend the current output while remaining consistent with the target schema.

### Backend 1: XGrammar (Default)

XGrammar (default since vLLM ~0.6+) uses **Pushdown Automata (PDA)** for grammar-based decoding, providing substantially better performance than the previous Outlines backend.

#### Technical Approach

- **PDA over FSM**: XGrammar's PDA can represent context-free grammars (CFGs), which are strictly more expressive than the Finite State Machines (FSMs) used by Outlines. A PDA is conceptually "a collection of FSMs, where each FSM represents a context-free grammar rule."
- **Multi-step transitions**: PDA allows multiple state transitions per token, more efficiently handling complex grammar constructs
- **C with pthread**: Grammar compilation was moved from Python to native C code with POSIX threading, dramatically reducing the CPU overhead of grammar setup
- **Tokenizer caching**: Tokenizer data (token-to-character mappings) is cached across requests sharing the same schema, eliminating repeated compilation

#### Performance Impact

In benchmarks by Red Hat (January 2025), XGrammar provides **5× improvement in Time-Per-Output-Token (TPOT)** under high concurrency compared to Outlines, primarily because XGrammar's C-compiled constraint enforcement adds less latency per token than Outlines' Python-based FSM traversal.

#### Supported Constraint Types

| Type | Description | Example |
|------|-------------|---------|
| `json` | JSON schema validation | `{"type": "object", "properties": {...}}` |
| `regex` | Regular expression matching | `r"\w+@\w+\.com"` |
| `grammar` | EBNF/GBNF context-free grammar | SQL grammar, custom DSL |
| `choice` | Enumeration of valid strings | `["yes", "no", "maybe"]` |
| `structural_tag` | Tag-based structure | Custom structured output formats |

#### XGrammar Limitations (as of 2025)

- Complex JSON schemas with regex patterns or numeric range constraints not fully supported (upstream PRs in progress)
- Only GBNF grammar format (EBNF support planned)
- Recursive grammars can have high compilation overhead on first use

### Backend 2: Outlines (FSM-Based)

Outlines (Willard & Louf, 2023) was vLLM's original guided decoding backend and remains available as a fallback for patterns not yet supported by XGrammar.

**Mechanism:**
1. Compile the schema/regex into a Finite State Machine (FSM)
2. At each decode step: `valid_tokens = fsm.allowed_tokens(current_state)`
3. Apply mask and sample

**Limitations:**
- Per-request FSM compilation is expensive and single-threaded
- One state transition per token (less efficient than PDA)
- Mask computation blocks the sampling critical path

### Backend 3: guidance

The `guidance` library is supported as an additional structured output backend in recent vLLM versions, providing another option for schema-constrained generation.

### API Usage

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1")

# JSON schema enforcement
from pydantic import BaseModel
class UserProfile(BaseModel):
    name: str
    age: int
    email: str

response = client.chat.completions.create(
    model="meta-llama/Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "Generate a user profile"}],
    extra_body={
        "guided_json": UserProfile.schema(),
    }
)

# Regex pattern enforcement
response = client.chat.completions.create(
    model=model,
    messages=[{"role": "user", "content": "Generate an email address"}],
    extra_body={
        "guided_regex": r"\w+@\w+\.(com|org|net)",
    }
)

# Context-free grammar (SQL)
sql_grammar = """
start: select_stmt
select_stmt: "SELECT" columns "FROM" table
...
"""
response = client.chat.completions.create(
    model=model,
    messages=[{"role": "user", "content": "Query for all users"}],
    extra_body={"guided_grammar": sql_grammar}
)

# Constrained choice
response = client.chat.completions.create(
    model=model,
    messages=[{"role": "user", "content": "Is this positive or negative?"}],
    extra_body={"guided_choice": ["positive", "negative", "neutral"]}
)
```

### V1 Roadmap for Structured Decoding

vLLM V1 includes planned improvements to reduce structured output overhead further:

- **Scheduler-level mask computation**: Pre-compute token masks before the GPU forward pass completes (using CPU during GPU computation), removing mask computation from the sampling critical path entirely
- **Centralized mask broadcasting**: For tensor-parallel deployments, compute masks once per batch and broadcast to all GPU workers, eliminating redundant computation across TP ranks
- **Spec decode + structured output**: Tree scoring in speculative decode shares the same state-machine API as guided decoding, enabling correct integration
- **Tool-use schema support**: XGrammar developing native tool-calling schema support to replace Python-level tool argument parsing

---

## LoRA Multi-Adapter Serving

vLLM supports serving **multiple LoRA adapters simultaneously** from a single base model, enabling cost-efficient multi-tenant deployment where different users receive different fine-tuned behavior without separate model instances.

### Architecture

```
Base Model Weights (loaded once, shared)
     ↓
Linear Layer Forward Pass:
  y = x × W_base + x × (A × B) × scale
                   └─ LoRA delta ─┘
```

Where A (down-projection) and B (up-projection) are the LoRA-specific matrices. Only A and B (typically 0.1–1% of base model parameters) differ between adapters.

### Punica SGMV Kernels

The core kernel enabling efficient multi-adapter batching is **Punica's Segmented-Gather Matrix-Vector (SGMV)** multiplication:

```
Standard batching (naive): separate kernel call per LoRA adapter
SGMV: single kernel call handles all adapters with different ranks
```

The SGMV kernel segments the input batch by adapter assignment and fuses the heterogeneous LoRA delta computations into a single GPU kernel launch. This eliminates per-adapter kernel launch overhead, which would otherwise scale O(num_adapters) per step.

### Horizontal Kernel Fusion (PR #11234, January 2025)

For `MergedColumnParallelLinear` layers (which merge multiple weight matrices, e.g., QKV projection), vLLM fuses the shrink (down-projection by A) and expand (up-projection by B) kernels for 1–3 sub-LoRAs in a single Triton kernel call, reducing total kernel launches for LoRA computation.

### Memory Management

```
GPU Memory:   active adapters (up to --max-loras)
CPU Memory:   inactive adapters (up to --max-cpu-loras)
```

The system hot-swaps adapters between GPU and CPU based on per-request routing, with the GPU cache managed as an LRU cache of adapter weights.

### Configuration

```bash
# Server startup
vllm serve meta-llama/Llama-2-7b-hf \
    --enable-lora \
    --max-loras 4 \           # Max adapters on GPU simultaneously
    --max-lora-rank 64 \      # Maximum supported LoRA rank
    --max-cpu-loras 16 \      # CPU cache size for inactive adapters
    --lora-modules \
        adapter1=/path/to/adapter1 \
        adapter2=/path/to/adapter2
```

```python
# Python API per-request routing
from vllm import LLM
from vllm.lora.request import LoRARequest

llm = LLM(
    model="meta-llama/Llama-2-7b-hf",
    enable_lora=True,
)

# Route to specific adapter
outputs = llm.generate(
    "Write a poem about clouds",
    lora_request=LoRARequest(
        lora_name="creative_writer",
        lora_int_id=1,
        lora_local_path="/path/to/creative_writer_adapter"
    )
)
```

### LoRA + Prefix Caching

LoRA adapters modify the model's effective computation, so KV cache blocks computed with adapter A cannot be reused for requests using adapter B. vLLM handles this by incorporating the LoRA adapter ID into the block hash:

```
block_hash = hash(lora_id, prefix_hash, block_tokens)
```

This ensures that each adapter maintains its own independent prefix cache, enabling APC to work correctly in multi-adapter deployments.

### Multi-Modal LoRA (Experimental)

LoRA adapters can target not just the language model backbone but also:
- **Vision tower**: fine-tune the image encoder for domain-specific visual understanding
- **Vision-language connector**: fine-tune the adapter between vision and language representations

This enables complete multimodal fine-tuning via LoRA, with per-request routing to different VLM fine-tunes served from a single base model.

---

## Multimodal Support

vLLM V1 extends beyond text to native support for vision, video, and audio inputs across a broad range of model architectures.

### Supported Modalities and Models

| Modality | Supported Model Families |
|----------|--------------------------|
| **Images** | LLaVA, Qwen2-VL, InternVL, Pixtral, Llama 3.2 Vision, Phi-3.5-Vision, Aria, Gemma3 |
| **Video** | Qwen2-VL, InternVL, LLaVA-Video |
| **Audio** | Qwen2-Audio, Whisper (transcription) |

### V1 Multimodal Architecture

#### 1. Encoder Cache

Vision encoder outputs (image embedding tensors) are stored in a dedicated on-GPU **encoder cache**, separate from the KV cache:

```
Request with image → Vision Encoder → Embeddings
                                           ↓
                                    Encoder Cache (GPU)
                                           ↓
                     Future request with same image → reuse embeddings
```

This avoids re-running the vision encoder (which can be computationally expensive, involving thousands of embedding vectors per image) for repeated images across requests or across chunks in chunked prefill.

#### 2. Encoder-Aware Scheduler

The V1 scheduler tracks multimodal embedding positions within each request, coordinating:
- When to invoke the vision encoder for new images
- How to interleave vision encoding with chunked prefill
- Which chunks contain embedded image tokens vs. text tokens

For a prompt containing both text and images, the scheduler knows exactly which token positions correspond to image embeddings and correctly handles KV block allocation.

#### 3. Asynchronous Preprocessing

Raw image data processing (decode, resize, normalize, convert to tensor) is offloaded to a **separate preprocessing process** (via ZeroMQ IPC), running asynchronously while the main engine processes other requests:

```
API Server → Image bytes → Preprocessing Process (async)
                                    ↓
                          Processed tensors → Engine Core
```

This prevents image preprocessing from blocking inference for other requests.

#### 4. Preprocessing Cache

Preprocessed image tensors are cached for identical inputs (byte-for-byte identical images), enabling deduplication across concurrent requests that include the same image:

```
Request A with image X → preprocess X → cache result
Request B with image X → cache hit → skip preprocessing
```

#### 5. Multimodal Prefix Caching

Image hashes are incorporated into the APC block hash:
```
block_hash = hash(token_ids_hash, image_content_hash)
```

For multi-turn VQA conversations with the same image, KV blocks encoding the image embeddings are cached and reused, eliminating both vision encoding and attention computation for repeated image tokens.

### Language-Model-Only Mode

For VLM-capable model architectures (Llama-4, Mistral-3, Qwen-3.5) where multimodal capability is not needed, the `--language-model-only` flag disables multimodal modules:

```bash
vllm serve meta-llama/Llama-4 --language-model-only
```

This frees GPU memory that would otherwise be allocated to the vision encoder and connector, increasing available KV cache capacity.

---

## Tool Calling and Function Calling

vLLM's OpenAI-compatible server supports tool/function calling, enabling agentic applications that route model outputs to external functions.

### Architecture

```
Client request with tools: [{"name": "search", "parameters": {...}}, ...]
         ↓
vLLM formats tools into model's chat template (model-specific format)
         ↓
Model generates response (possibly with tool call invocation)
         ↓
Tool parser watches streaming tokens for tool call patterns
         ↓
When complete tool call detected: emit tool_calls in response
         ↓
Client executes function, returns result
         ↓
Follow-up request with tool result
```

### Available Tool Parsers

| Parser Name | Target Models |
|-------------|---------------|
| `llama3_json` | Llama-3.x Instruct |
| `hermes` | Hermes-format models |
| `mistral` | Mistral Instruct |
| `qwen2.5` | Qwen 2.5 Instruct |
| `internlm` | InternLM models |
| `granite20` | IBM Granite 20B |
| `xlam` | xLAM function-calling models |
| `pythonic` | Models using Python-style function syntax |

### Server Configuration

```bash
vllm serve meta-llama/Meta-Llama-3-8B-Instruct \
    --enable-auto-tool-choice \
    --tool-call-parser llama3_json \
    --chat-template /path/to/tool_chat_template.jinja
```

### Client Usage

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="meta-llama/Meta-Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)

# Check if model wants to call a tool
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    # Execute: call get_weather(location="Tokyo")
```

### Guided Decoding Integration

When `tool_choice` specifies a particular function (not `auto`), vLLM can constrain the tool arguments to valid JSON matching the function's parameter schema using XGrammar:

```python
response = client.chat.completions.create(
    model=model,
    messages=[...],
    tools=tools,
    tool_choice={"type": "function", "function": {"name": "get_weather"}},
    # JSON schema for get_weather arguments is automatically enforced
)
```

This eliminates the possibility of malformed JSON in tool call arguments, improving reliability in automated pipelines.

---

## FlashAttention Integration

vLLM relies on FlashAttention for high-performance attention computation, supporting multiple FA versions with different capabilities and performance characteristics.

### Attention Backend Options

| Backend | Status | Use Case |
|---------|--------|----------|
| **FlashAttention 2 (FA2)** | Default | Production; most GPU types |
| **FlashAttention 3 (FA3)** | V1 + H100 | Full CUDA graph; maximum performance |
| **FlashInfer** | Alternative | Different performance characteristics |
| **Triton** | Pure Python | Flexibility; non-NVIDIA hardware |
| **FlashMLA** | DeepSeek only | MLA attention kernel |

### FlashAttention 2

FA2 is the production-tested default providing:

- **Online softmax (Flash Attention algorithm)**: Computes attention without materializing the full N×N attention matrix; O(N) memory instead of O(N²)
- **IO-aware tiling**: Reads Q, K, V from HBM exactly twice; writes output once — minimizing the dominant memory bandwidth cost
- **GQA/MQA optimization**: Handles Grouped-Query Attention and Multi-Query Attention with dynamic dispatch based on a `num_splits` heuristic

**CUDA graph limitation**: FA2's GQA dynamic dispatch (branching on `num_splits`) prevents full CUDA graph capture. FA2 supports CUDA graphs only for **uniform decode batches** (all queries length 1) — mixed prefill/decode batches must run in eager mode with FA2.

### FlashAttention 3

FA3 addresses FA2's CUDA graph limitation and provides additional capabilities for H100 hardware:

- **Unified attention routine**: No dynamic dispatch; deterministic execution graph enables full CUDA graph capture for all batch types
- **Warpspecialization**: Dedicates warp groups to different pipeline stages (data loading vs. GEMM), overlapping memory access with computation on H100's specialized hardware
- **FP8 support**: Directly computes attention with FP8 KV cache, eliminating a dequantization step
- **Required for full CUDA graph in V1**: The `FULL` CUDA graph mode requires FA3

Enabling FA3:
```bash
VLLM_FLASH_ATTN_VERSION=3 VLLM_USE_V1=1 vllm serve <model> \
    --compilation-config '{"cudagraph_mode": "FULL"}'
```

### FlashMLA (DeepSeek MLA)

DeepSeek-V2/V3 use **Multi-head Latent Attention (MLA)**, which compresses the KV cache into low-rank latent representations rather than storing full K and V tensors:

```
Standard attention: KV cache = K + V (large)
MLA: KV cache = compressed latent c_KV (small)
     At attention time: K, V are up-projected from c_KV
```

**FlashMLA** is a specialized attention kernel for MLA decode, providing:
- Efficient BF16 queries + compressed latent KV computation
- Full CUDA graph support (added in PR #18581)
- FP8 KV cache support for latent tensors (via Triton MLA backend, PR #34597)

### PagedAttention

vLLM's core innovation remains the **PagedAttention** kernel — a custom attention implementation that handles non-contiguous KV cache storage:

```
Block table: logical block → physical memory address
Attention:   gather K, V blocks from scattered physical locations
             → run attention with gathered K, V
```

PagedAttention enables OS-style virtual memory management for the KV cache, allowing different sequences to share physical memory pages and enabling fine-grained memory reclamation. Block sizes (typically 16 tokens) balance gather overhead against internal memory fragmentation.

---

## CUDA Graph Optimization

CUDA graphs eliminate CPU-side overhead by pre-capturing the entire sequence of GPU operations and replaying them with minimal CPU involvement.

### Why CUDA Graphs Matter

At small batch sizes (e.g., batch size 1 with a Llama-8B on H100), the GPU forward pass completes in ~5ms. However, without CUDA graphs:

```
Python interpreter overhead: 1–3ms
CUDA API kernel launch overhead: 1–2ms
CPU–GPU synchronization: 0.5–1ms
Total overhead: 2.5–6ms ≈ 50–120% of actual GPU compute time
```

CUDA graphs eliminate all of this, making per-step overhead essentially zero.

### Piecewise CUDA Graphs (V1 Default)

Standard CUDA graphs require all tensor shapes to be fixed. Since attention is dynamic (variable-length KV cache), vLLM V1 uses **piecewise CUDA graphs** via `torch.compile`:

```
Forward pass = [Static GEMM] + [Dynamic Attention] + [Static GEMM] + ...
                  ↑captured         ↑eager mode         ↑captured
```

Static segments (linear layers, non-attention ops) are captured as graph segments. Dynamic segments (attention) run in eager mode. The overall pass alternates between graph replay and eager execution.

**Padding for fixed shapes**: Since graph capture requires fixed shapes, vLLM captures graphs at discrete batch sizes (1, 2, 4, 8, 16, ..., max). The actual batch is padded to the nearest captured size.

### Full CUDA Graph (V1 + FA3)

With FlashAttention 3, the attention computation becomes graphable (no dynamic dispatch). PR #16072 (April 2025) introduced:

```bash
VLLM_FLASH_ATTN_VERSION=3 VLLM_USE_V1=1 vllm serve <model> \
    --compilation-config '{"cudagraph_mode": "FULL"}'
```

In `FULL` mode, the **entire forward pass** (including attention) is captured in a single CUDA graph. This provides the maximum CPU overhead reduction but requires:
- FA3 (SM 9.0 / H100)
- Persistent buffers for attention metadata
- Incompatible with cascade attention (currently)

### Graph Capture at Startup

During vLLM server initialization ("warming up"), CUDA graphs are captured at all supported batch sizes:

```
Startup: capture graph for batch_size ∈ {1, 2, 4, 8, 16, 32, 64, ...}
Runtime: select nearest captured size; pad input if needed
```

The number of graphs captured and the GPU memory consumed by graphs depends on `--max-num-seqs`. This explains why vLLM's startup takes longer than a naive model load.

### torch.compile Integration

V1 uses `torch.compile` as the foundation for CUDA graph management:

- **Automatic subgraph detection**: Identifies static vs. dynamic operations without manual annotation
- **Inductor optimization**: Generates optimized CUDA kernels, including operator fusion (e.g., fusing element-wise operations into preceding GEMMs)
- **Optimization levels**: `-O0` (disabled) through `-O4` (aggressive); `-O3` is V1 default
- **Piecewise capture**: `FULL_AND_PIECEWISE` mode (V1 default) intelligently captures where possible

```python
# V1 compilation config (passed as server argument)
# --compilation-config '{"optimization_level": 3, "cudagraph_mode": "FULL_AND_PIECEWISE"}'
```

---

## OpenAI-Compatible API Features

vLLM implements the OpenAI API specification, enabling drop-in replacement for applications using OpenAI's Python/REST client libraries.

### Chat Completions

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="dummy")

# Standard chat
response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum entanglement."},
    ],
    max_tokens=500,
    temperature=0.7,
    top_p=0.9,
)

# Streaming
for chunk in client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[...],
    stream=True,
):
    print(chunk.choices[0].delta.content, end="", flush=True)
```

### Completions (Legacy)

```python
response = client.completions.create(
    model="meta-llama/Llama-3.1-8B",
    prompt="The capital of France is",
    max_tokens=50,
)
```

### Embeddings API

```python
response = client.embeddings.create(
    model="intfloat/e5-mistral-7b-instruct",
    input=["Text to embed", "Another text"],
)
embeddings = [item.embedding for item in response.data]
```

### vLLM-Specific API Extensions

vLLM extends the OpenAI API with additional parameters via `extra_body`:

```python
response = client.chat.completions.create(
    model=model,
    messages=[...],
    extra_body={
        # Prefix caching hint
        "use_beam_search": False,

        # Guided decoding
        "guided_json": schema_dict,
        "guided_regex": r"pattern",
        "guided_grammar": grammar_string,
        "guided_choice": ["option1", "option2"],
        "guided_decoding_backend": "xgrammar",  # or "outlines"

        # Speculative decoding (per-request, experimental)
        "num_speculative_tokens": 5,

        # Request priority
        "priority": 1,  # higher = more important

        # Skip special tokens in response
        "skip_special_tokens": True,
    }
)
```

### Multi-Modal Input (Chat API)

vLLM's OpenAI-compatible API supports image inputs following the OpenAI vision format:

```python
response = client.chat.completions.create(
    model="llava-hf/llava-1.5-7b-hf",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": "https://..."}},
                {"type": "text", "text": "Describe this image."}
            ]
        }
    ],
)
```

### Model List and Info

```python
# List available models
models = client.models.list()
for model in models.data:
    print(model.id)
```

### Batch API (Offline Processing)

For batch workloads, the Python API provides direct access without HTTP:

```python
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")
params = SamplingParams(temperature=0.8, top_p=0.95, max_tokens=256)

prompts = ["Prompt 1", "Prompt 2", "Prompt 3"]
outputs = llm.generate(prompts, params)

for output in outputs:
    print(output.outputs[0].text)
```

---

## vLLM V1 Architecture Overview

vLLM V1 (default since mid-2025) is a complete re-architecture of the core engine that provides a foundation for all the advanced features described in this article.

### Key Architectural Changes

#### Isolated EngineCore Process

The scheduler and model executor run in a dedicated **EngineCore process**:

```
API Server Process ──── ZeroMQ IPC ──── EngineCore Process
   (HTTP, tokenization,                    (scheduler, GPU execution)
    streaming, de-tokenization)
```

CPU-intensive tasks (tokenization, de-tokenization, multimodal preprocessing, streaming output) run in the API Server process and overlap with GPU computation in EngineCore, improving overall efficiency.

#### Unified Scheduler

V1's scheduler eliminates the prefill/decode distinction:
- All scheduling expressed as `{request_id: num_tokens}`
- Naturally handles chunked prefill, speculative decode, multimodal, and mixed batches
- Persistent batch with incremental diff updates (reduces CPU-side overhead)

#### Symmetric Tensor Parallelism

In V1, all tensor-parallel workers are fully identical — the scheduler transmits incremental diffs rather than full batch state. This reduces per-step communication overhead in large TP deployments.

#### Performance: Up to 1.7× Throughput vs V0

The V1 architecture improvements (pipeline parallelism between CPU and GPU, persistent batch, O(1) caching, unified scheduler) provide up to 1.7× higher throughput compared to V0 on equivalent hardware, with even larger gains for multimodal workloads (>1.7× due to async preprocessing).

---

## Reasoning Model Support

vLLM added dedicated support for models that output explicit reasoning traces before their final answer (DeepSeek-R1, QwQ, etc.):

```bash
vllm serve deepseek-ai/DeepSeek-R1 \
    --reasoning-parser deepseek_r1
```

The reasoning parser:
- Detects and extracts content within `` / `` tags
- Separates reasoning content from the final answer in the response structure
- Excludes reasoning content from structured output schema enforcement (reasoning trace is unconstrained; only the final answer is constrained)

```python
response = client.chat.completions.create(
    model="deepseek-ai/DeepSeek-R1",
    messages=[{"role": "user", "content": "Solve: 2x + 5 = 13"}],
)

# Response includes separate reasoning_content field
reasoning = response.choices[0].message.reasoning_content
answer = response.choices[0].message.content
```

---

## MoE-Specific Optimizations

For Mixture-of-Experts models (Mixtral, DeepSeek-V2/V3, Qwen-MoE):

### Fused MoE Kernel

Expert routing and expert GEMM execute in a single GPU kernel pass, avoiding separate kernel launches for routing and computation:

```
Input → [Expert Router] → [Expert GEMMs] → Output
                    ↑ single fused kernel ↑
```

### Expert Parallel Load Balancing (EPLB)

In expert-parallel deployments (experts distributed across GPUs), load imbalance occurs when some experts receive many more tokens than others. EPLB (Expert Parallel Load Balancing) dynamically balances work across expert-parallel ranks during inference.

### FP8 MoE

Expert weight matrices are quantized to FP8, with online quantization applied during the fused kernel. The unified `MoeOnlineWeightLoader` handles both INT8 and FP8 MoE quantization through the same serving path.

---

## Disaggregated Prefill and KV Transfer

For large-scale deployments, vLLM supports separating prefill and decode onto different instance pools:

### Architecture

```
Client → Load Balancer
              ↓
         P Instances (Prefill-Specialized)
              │ KV cache transfer (NCCL / RDMA)
              ↓
         D Instances (Decode-Specialized)
              ↓
         Response → Client
```

- **P Instances**: Large chunk sizes, optimized for compute-intensive prefill of long contexts
- **D Instances**: Large batch sizes, optimized for memory-bandwidth-bound decode

### KV Transfer Mechanisms

- **NCCL**: In-cluster GPU-to-GPU transfer via NVLink or InfiniBand
- **RDMA**: Cross-node transfer via high-bandwidth interconnects
- **LMCache**: External KV store enabling cross-instance cache sharing
- **Mooncake integration**: vLLM's `kv_transfer` module supports Mooncake's distributed KV cache protocol

---

## Observability and Deployment Features

### Prometheus Metrics

vLLM exposes comprehensive metrics for monitoring:

| Metric | Description |
|--------|-------------|
| `vllm:request_success_total` | Total successful requests |
| `vllm:request_prompt_tokens_total` | Total prompt tokens processed |
| `vllm:e2e_request_latency_seconds` | End-to-end request latency histogram |
| `vllm:time_to_first_token_seconds` | TTFT histogram |
| `vllm:time_per_output_token_seconds` | TPOT histogram |
| `vllm:gpu_cache_usage_perc` | GPU KV cache utilization |
| `vllm:cache_config_info` | Cache configuration (hit rates, etc.) |
| `vllm:num_requests_running` | Current active requests |
| `vllm:num_requests_waiting` | Queued requests |

### Sleep Mode

For cost optimization with bursty workloads:

```python
# Offload model to CPU when idle
llm.sleep(level=1)  # level 1: keep weights on CPU
# ... idle period ...
llm.wake_up()       # reload to GPU on demand
```

### MCP Server Integration

vLLM provides a Model Context Protocol (MCP) server endpoint, enabling agentic frameworks (LangChain, AutoGPT, etc.) to interact with vLLM-served models through the standardized MCP interface.

### Multi-Node Tensor Parallelism

For models too large for a single node:

```bash
# Node 1 (rank 0 - primary)
vllm serve meta-llama/Llama-3.1-405B \
    --tensor-parallel-size 16 \
    --pipeline-parallel-size 2 \
    --distributed-executor-backend ray

# Node 2 (rank 1 - worker)
# Started automatically via Ray cluster
```

Ray-based distributed execution handles cross-node communication, worker process management, and failure recovery.
