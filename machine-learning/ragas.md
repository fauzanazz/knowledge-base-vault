---
title: "RAGAS"
category: machine-learning
summary: "RAGAS is an evaluation framework for RAG systems that measures faithfulness, answer relevance, context precision, and context recall. It provides standardized metrics for assessing retrieval and generation quality."
sources:
  - raw/articles/rag-architecture-naive-advanced-modular.md
  - raw/articles/eval-frameworks-ragas-deepeval-braintrust.md
updated: 2026-04-08T19:21:34.925Z
---

# RAGAS

> RAGAS is an evaluation framework for RAG systems that measures faithfulness, answer relevance, context precision, and context recall. It provides standardized metrics for assessing retrieval and generation quality.

# RAGAS

RAGAS is a specialized evaluation framework for [[RAG Architecture]] systems that provides metrics mapping precisely to the two core failure modes: bad retrieval (irrelevant context) and bad generation (unfaithful answers).

## Core Metrics

**Faithfulness** measures whether factual claims in generated answers are grounded in retrieved context. Uses LLM-as-judge to decompose answers into atomic claims and verify each against context chunks. Scores below 0.8 indicate hallucination beyond provided context.

**Answer Relevancy** measures whether answers actually address the asked question. An answer can be faithful but irrelevant. RAGAS computes this by generating reverse-engineered questions from answers and measuring similarity to originals.

**Context Precision** measures correct ranking of retrieved chunks - whether most relevant chunks appear first. Higher-ranked irrelevant chunks waste context windows and inflate costs.

**Context Recall** measures whether retrieved context contains information needed to answer questions, evaluated against reference answers.

## Implementation

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy
from datasets import Dataset

data = {
    "question": ["What is the capital of France?"],
    "answer": ["Paris is the capital of France."],
    "contexts": [["France's capital city is Paris."]],
    "ground_truth": ["Paris"]
}

result = evaluate(Dataset.from_dict(data), metrics=[faithfulness, answer_relevancy])
```

## Synthetic Test Generation

RAGAS generates question/answer/context triples from document corpora, creating diverse question types including factoid, multi-hop reasoning, and multi-context synthesis for realistic test distributions.

---
*Related: [[RAG Architecture]], [[LLM Evaluation Frameworks]], [[Vector Databases]], [[Chunking Strategies]]*
