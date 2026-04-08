---
title: "LLM Fine-tuning"
category: machine-learning
summary: "Fine-tuning continues training pre-trained language models on domain-specific data to permanently alter weights and internalize specialized patterns, style, and behaviors."
sources:
  - raw/articles/fine-tuning-vs-rag-vs-prompt-engineering-decision-framework.md
updated: 2026-04-08T19:20:41.510Z
---

# LLM Fine-tuning

> Fine-tuning continues training pre-trained language models on domain-specific data to permanently alter weights and internalize specialized patterns, style, and behaviors.

# LLM Fine-tuning

Fine-tuning continues training a pre-trained language model on domain-specific data, updating the model's weights to internalize patterns, style, facts, and behaviors from your dataset. The result is a specialized model checkpoint deployed separately from the base model.

## When to Use Fine-tuning

Fine-tuning solves behavior problems, not knowledge problems:

- **Style internalization**: Legal writing, medical documentation, brand voice too subtle for [[Prompt Engineering]]
- **Domain-specific language**: Heavy jargon, specialized output formats, field-specific reasoning
- **Consistent behavior at scale**: Eliminate prompt-sensitivity and reduce token costs
- **Latency optimization**: Fine-tuned smaller models can match larger base models
- **Safety/alignment**: RLHF and DPO to steer away from specific outputs

## Data Requirements

| Scenario | Min Examples | Recommended |
|---|---|---|
| Style/format adjustment | 50–200 | 500+ |
| Domain-specific reasoning | 500–1000 | 2000+ |
| Complex task learning | 1000–5000 | 10K+ |
| Replacing system prompts | 50–100 | 300+ |

Quality significantly outweighs quantity—200 expert-curated examples outperform 2000 noisy examples.

## Training Process

Fine-tuning uses supervised learning on input-output pairs formatted as conversations:

```json
{"messages": [
  {"role": "system", "content": "You are a legal assistant..."},
  {"role": "user", "content": "Draft a confidentiality clause"},
  {"role": "assistant", "content": "CONFIDENTIALITY. The parties agree..."}
]}
```

The model learns to predict the assistant's response given the context, gradually adapting its weights to match the training distribution.

## Limitations

- **Knowledge staleness**: Facts frozen at training time; combine with [[RAG Architecture]] for dynamic knowledge
- **High cost**: GPT-4o fine-tuning costs ~$25/1M training tokens plus deployment
- **Data collection bottleneck**: Generating quality training data is slow and expensive
- **Opaque evaluation**: Unlike [[RAG Architecture]], weight changes are difficult to inspect
- **Catastrophic forgetting**: Narrow domain training can degrade general capabilities

## Hybrid Patterns

The most effective production systems combine fine-tuning with other approaches:

- **Fine-tuned model + [[RAG Architecture]]**: Fine-tune for style, retrieve for knowledge
- **Fine-tuned retriever + base generator**: Improve [[Vector Embeddings]] quality without generator costs
- **Fine-tuning + [[Prompt Engineering]]**: Reduce prompt length while maintaining flexibility

Fine-tuning works best when you have clear behavioral requirements and sufficient high-quality training data.

---
*Related: *
