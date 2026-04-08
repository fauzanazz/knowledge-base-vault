---
title: "HNSW Algorithm"
category: algorithms
summary: "HNSW (Hierarchical Navigable Small World) is the dominant algorithm for approximate nearest-neighbor search in vector databases. It builds multi-layer graphs enabling fast traversal with 95-99% recall and ~10ms query times on millions of vectors."
sources:
  - raw/articles/embedding-vector-db-pinecone-qdrant-chroma.md
updated: 2026-04-08T18:46:13.768Z
---

# HNSW Algorithm

> HNSW (Hierarchical Navigable Small World) is the dominant algorithm for approximate nearest-neighbor search in vector databases. It builds multi-layer graphs enabling fast traversal with 95-99% recall and ~10ms query times on millions of vectors.

# HNSW Algorithm

HNSW (Hierarchical Navigable Small World) is the dominant algorithm for approximate nearest-neighbor (ANN) search in [[Vector Databases]]. It builds a multi-layer graph structure that enables fast traversal through high-dimensional [[Vector Embeddings]] space.

## Architecture

HNSW creates a hierarchical structure where:
- **Bottom layer** contains all vectors in the dataset
- **Upper layers** contain progressively sparser "skip graphs" for fast traversal
- Each node connects to nearby nodes, forming a navigable small-world network

## Query Process

1. **Enter at top layer** and greedily navigate to the closest node
2. **Descend one layer** and repeat the greedy search
3. **Continue until bottom layer** where the final nearest neighbors are found

This approach typically achieves 95-99% recall with ~10ms query time on millions of vectors.

## Tuning Parameters

**M** - Number of bidirectional links per node (typical: 16-64)
- Higher M = better recall but more memory usage
- Lower M = faster queries but potentially lower recall

**ef_construction** - Size of dynamic candidate list during index building (typical: 100-500)
- Higher values = better index quality but slower build time
- Critical for index quality, affects all future queries

**ef** - Candidate list size at query time
- Runtime parameter that trades recall vs latency
- Can be adjusted per query based on requirements

## Performance Characteristics

HNSW provides excellent performance for datasets up to billions of vectors. Memory usage scales with the number of vectors and the M parameter. The algorithm is particularly effective because it maintains the small-world property where most nodes are reachable through a small number of hops.

HNSW has become the standard for production vector search due to its combination of high recall, low latency, and predictable performance characteristics.

---
*Related: [[Vector Databases]], [[Vector Embeddings]], [[Approximate Nearest Neighbor]], [[Search Algorithms]], [[Performance Optimization]]*
