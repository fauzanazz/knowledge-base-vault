---
title: "Vector Embeddings"
category: machine-learning
summary: "Vector embeddings are dense, fixed-length numerical representations that encode semantic meaning of inputs like text, images, or audio. Semantically similar inputs produce geometrically close vectors in high-dimensional space."
sources:
  - raw/articles/embedding-vector-db-pinecone-qdrant-chroma.md
updated: 2026-04-08T18:46:13.763Z
---

# Vector Embeddings

> Vector embeddings are dense, fixed-length numerical representations that encode semantic meaning of inputs like text, images, or audio. Semantically similar inputs produce geometrically close vectors in high-dimensional space.

# Vector Embeddings

A vector embedding is a dense, fixed-length vector of floating-point numbers that encodes the semantic meaning of an input (text, image, audio, structured data). Inputs that are semantically similar land geometrically close in the high-dimensional embedding space; dissimilar inputs land far apart.

```
"I love dogs"    → [0.21, -0.54, 0.88, ..., 0.03]  (1536 dimensions)
"I enjoy puppies"→ [0.19, -0.51, 0.86, ..., 0.01]  ← very close
"Quantum physics"→ [-0.72, 0.38, -0.12, ..., 0.91] ← far away
```

This geometric property enables semantic search, RAG systems, recommendation engines, deduplication, and clustering applications.

## Embedding Models

**Proprietary models** include OpenAI's `text-embedding-3-small` (1536 dimensions) and `text-embedding-3-large` (3072 dimensions), plus Cohere's `embed-v4.0` with 128K context length.

**Open source models** like `BAAI/bge-large-en-v1.5` and `intfloat/e5-large-v2` offer strong performance with 1024 dimensions, while `Alibaba-NLP/gte-Qwen2-7B-instruct` provides state-of-the-art quality with 3584 dimensions.

## Distance Metrics

[[Vector Databases]] use different distance metrics:
- **Cosine similarity** for most NLP embeddings where magnitude doesn't matter
- **Dot product** for embeddings explicitly trained for it
- **Euclidean distance** when absolute magnitude matters

## Production Considerations

Higher dimensions provide more expressiveness but increase storage and compute costs. **Matryoshka Representation Learning (MRL)** embeddings allow truncating dimensions with graceful quality degradation, enabling 6x storage reduction with only ~5% quality loss.

Embeddings form the foundation of modern [[Vector Databases]] and semantic search systems.

---
*Related: [[Vector Databases]], [[Semantic Search]], [[Machine Learning]], [[Natural Language Processing]]*
