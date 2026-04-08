---
title: "Few-Shot Learning"
category: machine-learning
summary: "Few-shot learning uses a small number of examples (typically 3-5) to teach LLMs patterns for structured tasks, providing higher ROI than exhaustive instructions. It's the most effective prompt enhancement for classification and structured generation."
sources:
  - raw/articles/prompt-engineering-patterns-applied.md
updated: 2026-04-08T19:20:06.457Z
---

# Few-Shot Learning

> Few-shot learning uses a small number of examples (typically 3-5) to teach LLMs patterns for structured tasks, providing higher ROI than exhaustive instructions. It's the most effective prompt enhancement for classification and structured generation.

# Few-Shot Learning

Few-shot learning uses a small number of examples to teach LLMs patterns for structured tasks. The model infers the desired behavior from examples rather than requiring exhaustive specification, making it the highest-ROI addition for structured tasks.

## Basic Pattern

Instead of zero-shot instructions:
```
Classify the sentiment of this review: '{text}'
```

Use few-shot examples:
```
Classify sentiment as POSITIVE, NEGATIVE, or NEUTRAL.

Review: "Absolutely love this product, works perfectly!"
Sentiment: POSITIVE

Review: "Broke after two days. Complete waste of money."
Sentiment: NEGATIVE

Review: "Does what it says. Nothing special."
Sentiment: NEUTRAL

Review: "{text}"
Sentiment:
```

## Best Practices

**Example Selection**:
- Use 3-5 examples covering edge cases, not just happy paths
- Include at least one example for each output class
- Order examples from simple to complex
- Balance examples across classes to avoid mode bias

**Formatting Consistency**: Maintain identical formatting between examples and the target query to ensure pattern recognition.

**Dynamic Selection**: For large example libraries, retrieve the most semantically similar examples to the current query using [[Vector Embeddings]] search. This outperforms fixed examples for diverse input distributions.

## Integration with Other Patterns

Few-shot learning combines effectively with:
- **[[Chain-of-Thought]]**: Show reasoning steps in examples
- **[[Prompt Engineering]]**: Include in system prompt structure
- **[[Tool Calling]]**: Demonstrate proper function usage

## When to Use

Few-shot learning excels for:
- Classification tasks with multiple categories
- Structured data extraction
- Format conversion tasks
- Style transfer and tone matching

It's less effective for open-ended creative generation where examples might constrain creativity.

---
*Related: [[Prompt Engineering]], [[Chain-of-Thought]], [[Tool Calling]], [[Vector Embeddings]]*
