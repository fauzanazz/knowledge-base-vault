---
title: "Cross-Encoder Reranking"
category: machine-learning
summary: "Cross-encoder reranking is a post-retrieval optimization that jointly scores query-document pairs for improved precision. It's more accurate than bi-encoder similarity but has O(N) inference cost."
sources:
  - raw/articles/rag-architecture-naive-advanced-modular.md
updated: 2026-04-08T19:14:16.239Z
---

# Cross-Encoder Reranking

> Cross-encoder reranking is a post-retrieval optimization that jointly scores query-document pairs for improved precision. It's more accurate than bi-encoder similarity but has O(N) inference cost.

# Cross-Encoder Reranking

Cross-encoder reranking is a post-retrieval optimization technique in [[RAG Architecture]] that jointly scores query-document pairs to improve precision after initial retrieval.

## How It Works

While initial retrieval optimizes for recall, reranking optimizes for precision:

1. Initial retrieval returns top-K candidates (e.g., 20 documents)
2. Cross-encoder model scores each (query, document) pair jointly
3. Results are reranked by relevance scores
4. Top-N results (e.g., 3-5) are passed to the LLM

```python
from sentence_transformers import CrossEncoder
reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
scores = reranker.predict([(query, chunk.text) for chunk in candidates])
reranked = [c for _, c in sorted(zip(scores, candidates), reverse=True)]
```

## Bi-Encoder vs Cross-Encoder

**Bi-Encoder (Dense Retrieval):**
- Encodes query and documents separately
- Fast similarity computation via dot product
- Used for initial retrieval from large corpora

**Cross-Encoder (Reranking):**
- Processes query and document together
- More accurate relevance scoring through attention
- Higher computational cost (O(N) inference)

## Production Usage

Cross-encoder reranking is considered "cheap insurance" in production RAG systems. Managed services like Cohere Rerank API provide drop-in reranking that typically reduces top-20 results to top-3 with significantly improved relevance.

The pattern of dense retrieval followed by cross-encoder reranking has become standard in production systems, balancing the speed of approximate search with the accuracy of joint query-document modeling.

---
*Related: [[RAG Architecture]], [[Vector Embeddings]], [[Information Retrieval]], [[Hybrid Search]]*
