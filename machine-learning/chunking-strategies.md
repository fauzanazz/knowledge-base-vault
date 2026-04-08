---
title: "Chunking Strategies"
category: machine-learning
summary: "Chunking strategies determine retrieval granularity in RAG systems and are the most common failure point. Strategies range from fixed-size splitting to semantic chunking and parent-child hierarchies."
sources:
  - raw/articles/rag-architecture-naive-advanced-modular.md
updated: 2026-04-08T19:14:16.238Z
---

# Chunking Strategies

> Chunking strategies determine retrieval granularity in RAG systems and are the most common failure point. Strategies range from fixed-size splitting to semantic chunking and parent-child hierarchies.

# Chunking Strategies

Chunking strategies determine retrieval granularity in [[RAG Architecture]] systems and represent the most common failure point. The choice affects both retrieval precision and context quality.

## Fixed-Size Chunking

Splits text every N tokens (e.g., 512) with overlap windows (e.g., 64 tokens). Fast to implement but breaks semantic units mid-sentence.

```python
def fixed_chunk(text: str, size: int = 512, overlap: int = 64):
    tokens = tokenize(text)
    return [detokenize(tokens[i:i+size]) 
            for i in range(0, len(tokens), size - overlap)]
```

**Use when:** Corpus is well-structured (FAQ pairs, product descriptions).

## Semantic Chunking

Embeds sentences and splits on embedding distance jumps, preserving semantic coherence at the cost of variable chunk sizes. Splits where cosine distance between adjacent sentence embeddings exceeds a threshold.

## Recursive Character Chunking

Splits on a priority list of separators (`\n\n`, `\n`, ` `, ``) until chunks are below target size. Respects document structure better than fixed-size splitting.

## Parent-Child Chunking

The current production standard. Stores parent chunks (e.g., 1024 tokens) for context but embeds child chunks (e.g., 128 tokens) for precise retrieval. When a child chunk is retrieved, its parent is returned to the LLM.

```
Index:    [child_1][child_2][child_3]  ← retrieved by similarity
Storage:  [        parent_1          ]  ← returned as context
```

This pattern dramatically improves context quality while maintaining retrieval precision. It addresses the fundamental tension between precise retrieval and comprehensive context in RAG systems.

---
*Related: [[RAG Architecture]], [[Vector Embeddings]], [[Information Retrieval]]*
