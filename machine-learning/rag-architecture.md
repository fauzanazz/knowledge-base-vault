---
title: "RAG Architecture"
category: machine-learning
summary: "Retrieval-Augmented Generation (RAG) grounds LLM responses in external knowledge through a spectrum of architectures from simple retrieve-then-read pipelines to modular orchestration systems. It reduces hallucination and enables domain-specific answers without fine-tuning."
sources:
  - raw/articles/rag-architecture-naive-advanced-modular.md
updated: 2026-04-08T19:14:16.237Z
---

# RAG Architecture

> Retrieval-Augmented Generation (RAG) grounds LLM responses in external knowledge through a spectrum of architectures from simple retrieve-then-read pipelines to modular orchestration systems. It reduces hallucination and enables domain-specific answers without fine-tuning.

# RAG Architecture

Retrieval-Augmented Generation (RAG) grounds LLM responses in external knowledge, reducing hallucination and enabling domain-specific answers without fine-tuning. RAG exists as a spectrum from simple pipelines to complex modular systems.

## Naive RAG

The simplest RAG follows a linear pipeline:
```
User Query → Embedding → Vector Store Lookup → Top-K Chunks → LLM Prompt → Response
```

This approach works for FAQ-style retrieval over tightly scoped corpora but breaks with:
- Irrelevant chunks due to query-chunk semantic mismatch
- Truncated context from fixed top-k limits
- Low faithfulness when LLMs ignore retrieved context
- Poor multi-hop reasoning from single retrieval steps

## Advanced RAG

Advanced RAG introduces optimization stages:

**Pre-Retrieval:**
- Query rewriting for better retrieval
- HyDE (Hypothetical Document Embedding) generates fake answers for embedding
- Query decomposition breaks complex queries into sub-questions

**Retrieval Methods:**
- Dense retrieval uses [[Vector Embeddings]] for semantic similarity
- Sparse retrieval (BM25) excels at exact keyword matching
- [[Hybrid Search]] combines both via Reciprocal Rank Fusion (RRF)

**Post-Retrieval:**
- Cross-encoder reranking improves precision
- Contextual compression extracts only relevant sentences
- Filtering removes duplicates

## Modular RAG

Modular RAG treats retrieval as a composable graph with routing, iterative retrieval, and tree-organized approaches like RAPTOR that cluster documents hierarchically.

## Production Patterns

**Parent-Child Chunking:** Store large parent chunks for context but embed small child chunks for precise retrieval. When a child is retrieved, return its parent to the LLM.

**Evaluation:** Use RAGAS metrics including faithfulness (claims supported by context), answer relevance, context precision, and context recall.

Hybrid retrieval with cross-encoder reranking represents the current production standard for most applications.

---
*Related: [[Vector Embeddings]], [[Vector Databases]], [[Hybrid Search]], [[Machine Learning]]*
