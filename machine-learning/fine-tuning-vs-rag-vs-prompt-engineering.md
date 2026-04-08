---
title: "Fine-tuning vs RAG vs Prompt Engineering"
category: machine-learning
summary: "A decision framework for choosing between prompt engineering, RAG, and fine-tuning when customizing LLM behavior based on cost, effort, and performance trade-offs."
sources:
  - raw/articles/fine-tuning-vs-rag-vs-prompt-engineering-decision-framework.md
updated: 2026-04-08T19:20:41.510Z
---

# Fine-tuning vs RAG vs Prompt Engineering

> A decision framework for choosing between prompt engineering, RAG, and fine-tuning when customizing LLM behavior based on cost, effort, and performance trade-offs.

# Fine-tuning vs RAG vs Prompt Engineering

When a base LLM doesn't behave as needed, you have three fundamental approaches: [[Prompt Engineering]], [[RAG Architecture]], and fine-tuning. Each occupies a different point in the cost/effort/performance space.

## The Three Approaches

**[[Prompt Engineering]]** shapes behavior through carefully crafted instructions at inference time. It requires no external infrastructure or training, making it the ideal starting point for any customization effort.

**[[RAG Architecture]]** injects relevant external knowledge into the context window dynamically through retrieval from vector databases or search engines. The model's weights remain unchanged.

**Fine-tuning** continues training a pre-trained model on domain-specific data, permanently altering its weights to internalize patterns, style, facts, and behaviors.

## Decision Matrix

| Dimension | Prompt Engineering | RAG | Fine-tuning |
|---|---|---|---|
| **Setup cost** | Very low (hours) | Medium (days–weeks) | High (weeks–months) |
| **Inference cost** | Base model cost | Base + retrieval overhead | Base model cost |
| **Latency** | ~0ms added | +50–500ms | ~0ms (or lower) |
| **Knowledge freshness** | Training cutoff only | Real-time updates | Stale after training |
| **Data requirement** | None | Documents/corpus | 500–50K+ labeled examples |
| **Debuggability** | High | High | Low |

## When to Use Each Approach

**Use [[Prompt Engineering]] for:**
- Output format control (JSON, tables)
- Persona/tone definition
- Simple reasoning tasks
- Rapid prototyping
- Tasks within the model's training distribution

**Use [[RAG Architecture]] for:**
- Dynamic knowledge that changes frequently
- Large knowledge corpora
- Citation requirements
- Multiple knowledge domains
- Privacy-sensitive source data

**Use Fine-tuning for:**
- Style and tone internalization
- Domain-specific language patterns
- Consistent behavior at scale
- Latency-critical applications (smaller fine-tuned models)
- Safety/alignment requirements

## Hybrid Approaches

Production systems often combine approaches. The most common pattern is fine-tuning for style consistency while using [[RAG Architecture]] for knowledge injection. This separates "how to respond" (fine-tuned behavior) from "what to say" (retrieved knowledge).

## Decision Framework

Start with [[Prompt Engineering]] always—it's free and instant. If knowledge gaps exist, add [[RAG Architecture]]. Only fine-tune when you need consistent style/behavior that prompts cannot achieve reliably.

---
*Related: *
