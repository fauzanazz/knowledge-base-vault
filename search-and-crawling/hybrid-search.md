---
title: "Hybrid Search"
category: search
summary: "Hybrid search combines dense vector embeddings with sparse keyword-based vectors to leverage both semantic understanding and exact keyword matching. It consistently outperforms dense-only search through techniques like Reciprocal Rank Fusion."
sources:
  - raw/articles/embedding-vector-db-pinecone-qdrant-chroma.md
updated: 2026-04-08T18:46:13.790Z
---

# Hybrid Search

> Hybrid search combines dense vector embeddings with sparse keyword-based vectors to leverage both semantic understanding and exact keyword matching. It consistently outperforms dense-only search through techniques like Reciprocal Rank Fusion.

# Hybrid Search

Hybrid search combines dense [[Vector Embeddings]] (semantic) with sparse BM25-style vectors (keyword) to leverage both semantic understanding and exact keyword matching. This approach consistently outperforms dense-only search by handling queries that require precise term matching alongside semantic similarity.

## Why Hybrid Search?

Dense embeddings excel at semantic similarity but can struggle with:
- Exact product names, model numbers, or technical terms
- Proper nouns and domain-specific terminology
- Queries requiring precise keyword matching

Sparse vectors (like BM25) excel at keyword matching but miss semantic relationships. Combining both approaches captures the strengths of each method.

## Implementation

**Reciprocal Rank Fusion (RRF)** is the standard technique for combining dense and sparse results:

```python
# Qdrant hybrid search example
results = client.query_points(
    collection_name="hybrid",
    prefetch=[
        Prefetch(query=dense_vec, using="dense", limit=20),
        Prefetch(query=SparseVector(indices=bm25_indices, values=bm25_values), using="sparse", limit=20),
    ],
    query=FusionQuery(fusion=Fusion.RRF),
    limit=5
)
```

## RRF Formula

RRF combines rankings from multiple retrieval systems:

```
RRF_score = Σ(1 / (k + rank_i))
```

Where `k` is a constant (typically 60) and `rank_i` is the rank from each system. This approach is robust and doesn't require score normalization.

## Database Support

[[Qdrant]] provides native sparse vector support for hybrid search. [[Pinecone]] supports hybrid through metadata filtering combined with dense search. [[Chroma]] requires external sparse vector generation.

## Performance Benefits

Hybrid search typically improves retrieval quality by 10-30% over dense-only approaches, particularly for:
- E-commerce product search
- Technical documentation retrieval
- Legal and medical document search
- Multi-language applications

The combination ensures both semantic understanding and precise term matching work together for optimal search results.

---
*Related: [[Vector Embeddings]], [[Vector Databases]], [[Semantic Search]], [[Information Retrieval]], [[Qdrant]], [[Search Algorithms]]*
