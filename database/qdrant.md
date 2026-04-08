---
title: "Qdrant"
category: database
summary: "Qdrant is an open-source vector database written in Rust, designed for high performance and reliability. It offers rich filtering capabilities, named vectors, and sparse vector support for hybrid search applications."
sources:
  - raw/articles/embedding-vector-db-pinecone-qdrant-chroma.md
updated: 2026-04-08T18:46:13.766Z
---

# Qdrant

> Qdrant is an open-source vector database written in Rust, designed for high performance and reliability. It offers rich filtering capabilities, named vectors, and sparse vector support for hybrid search applications.

# Qdrant

Qdrant is an open-source [[Vector Databases|vector database]] written in Rust, designed for high performance and reliability. It can be self-hosted or used through Qdrant Cloud, offering advanced features like rich filtering, named vectors, and sparse vector support.

## Key Features

**Named vectors** allow multiple embedding spaces per document, enabling separate embeddings for titles and body content:

```python
client.create_collection(
    collection_name="docs",
    vectors_config={
        "title": VectorParams(size=1536, distance=Distance.COSINE),
        "body": VectorParams(size=1536, distance=Distance.COSINE),
    }
)
```

**Rich filtering** supports complex metadata queries with datetime ranges, nested conditions, and geographic filters. **Sparse vector support** enables hybrid search combining dense [[Vector Embeddings]] with BM25-style keyword matching.

**WASM-based quantization** provides efficient compression while maintaining query performance.

## Hybrid Search

Qdrant excels at combining dense and sparse vectors using Reciprocal Rank Fusion (RRF):

```python
results = client.query_points(
    collection_name="hybrid",
    prefetch=[
        Prefetch(query=dense_vec, using="dense", limit=20),
        Prefetch(query=sparse_vec, using="sparse", limit=20),
    ],
    query=FusionQuery(fusion=Fusion.RRF),
    limit=5
)
```

## Trade-offs

**Advantages** include Rust-based performance, comprehensive filtering capabilities, active development, and flexibility for self-hosting or cloud deployment.

**Disadvantages** include operational overhead for self-hosting and a less mature cloud offering compared to [[Pinecone]].

Qdrant is ideal for teams requiring advanced filtering, hybrid search capabilities, or preferring open-source solutions with deployment flexibility.

---
*Related: [[Vector Databases]], [[Vector Embeddings]], [[Pinecone]], [[Chroma]], [[Hybrid Search]], [[Open Source]]*
