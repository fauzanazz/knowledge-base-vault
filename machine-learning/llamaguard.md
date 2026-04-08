---
title: "LlamaGuard"
category: machine-learning
summary: "LlamaGuard is Meta's fine-tuned Llama model specialized for content safety classification, covering 14 hazard categories and enabling self-hosted safety validation without third-party data exposure."
sources:
  - raw/articles/guardrails-safety-input-output-filtering.md
updated: 2026-04-08T19:22:50.927Z
---

# LlamaGuard

> LlamaGuard is Meta's fine-tuned Llama model specialized for content safety classification, covering 14 hazard categories and enabling self-hosted safety validation without third-party data exposure.

# LlamaGuard

LlamaGuard is Meta's fine-tuned Llama model specialized for content safety classification. Unlike rule-based systems, it's a proper ML classifier that handles nuanced safety cases across 14 hazard categories.

## Model Capabilities

LlamaGuard covers comprehensive safety categories:
- Violent crimes and illegal activities
- Hate speech and discrimination
- Sexual content and exploitation
- Privacy violations and PII exposure
- Self-harm and dangerous activities
- Regulated substances and weapons
- Misinformation and fraud

## Self-Hosted Safety

Running LlamaGuard locally eliminates third-party data exposure - critical for enterprise and regulated deployments:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

model_id = "meta-llama/Meta-Llama-Guard-2-8B"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype=torch.bfloat16)

def check_safety(conversation: list[dict]) -> str:
    input_ids = tokenizer.apply_chat_template(conversation, return_tensors="pt")
    output = model.generate(input_ids=input_ids, max_new_tokens=100)
    result = tokenizer.decode(output[0][input_ids.shape[-1]:], skip_special_tokens=True)
    return result  # "safe" or "unsafe\n<category>"
```

## Model Versions

**LlamaGuard 2** (8B parameters) provides the core safety classification capabilities with improved accuracy over the original version.

**LlamaGuard 3** (released late 2024) adds:
- Multilingual support for global deployments
- Expanded hazard taxonomy
- Better handling of edge cases and context-dependent safety

## Production Integration

LlamaGuard integrates into [[AI Guardrails]] pipelines as both input and output validation. It requires GPU resources (8B model needs ~16GB VRAM) but provides deterministic, auditable safety decisions without API dependencies.

## Advantages Over Rule-Based Systems

- Handles nuanced cases that regex patterns miss
- Adapts to context and conversational flow
- Provides confidence scores for threshold tuning
- No external API calls or data sharing
- Consistent performance across languages and domains

LlamaGuard is particularly valuable for organizations requiring explainable, self-hosted content safety with enterprise-grade privacy controls.

---
*Related: [[AI Guardrails]], [[LLM Fine-tuning]], [[Prompt Engineering]]*
