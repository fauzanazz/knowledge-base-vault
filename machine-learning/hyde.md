---
title: "HyDE"
category: machine-learning
summary: "HyDE (Hypothetical Document Embedding) is a retrieval technique that generates fake answers to queries and uses their embeddings for search. It works well when query language differs from document language."
sources:
  - raw/articles/rag-architecture-naive-advanced-modular.md
updated: 2026-04-08T19:14:16.238Z
---

# HyDE

> HyDE (Hypothetical Document Embedding) is a retrieval technique that generates fake answers to queries and uses their embeddings for search. It works well when query language differs from document language.

# HyDE

HyDE (Hypothetical Document Embedding) is a retrieval technique used in [[RAG Architecture]] that generates fake answers to queries and uses their embeddings for search instead of the original query embedding.

## How HyDE Works

1. Generate a hypothetical answer to the user query using an LLM
2. Embed the generated answer instead of the original query
3. Use this embedding to search the vector database
4. Retrieve documents based on similarity to the hypothetical answer

```python
def hyde_retrieve(query: str, k: int = 5):
    hypothetical_doc = llm.complete(f"Write a detailed answer to: {query}")
    hyp_embedding = embed(hypothetical_doc)
    return vector_store.search(hyp_embedding, top_k=k)
```

## When HyDE Helps

HyDE is particularly effective when:
- Query language differs from document language ("how do I..." vs documentation prose)
- Users ask questions in natural language but documents are technical
- The corpus contains answers rather than questions

## The Hypothesis

The core hypothesis is that a generated answer occupies a similar embedding space to real answers in the knowledge base. Since [[Vector Embeddings]] cluster semantically similar content, a well-generated hypothetical answer should retrieve actual answers more effectively than the original question.

## Limitations

HyDE adds LLM inference cost and latency to the retrieval process. It works best for technical corpora where there's a clear style gap between user queries and document content. For FAQ-style corpora where queries and documents are already similar, traditional embedding similarity often performs as well with lower cost.

---
*Related: [[RAG Architecture]], [[Vector Embeddings]], [[Information Retrieval]], [[Query Rewriting]]*
