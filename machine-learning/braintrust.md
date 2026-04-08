---
title: "Braintrust"
category: machine-learning
summary: "Braintrust is an experiment tracking platform for LLMs that focuses on comparing metric distributions across prompt versions and model changes, with built-in caching proxy for cost optimization."
sources:
  - raw/articles/eval-frameworks-ragas-deepeval-braintrust.md
updated: 2026-04-08T19:21:34.947Z
---

# Braintrust

> Braintrust is an experiment tracking platform for LLMs that focuses on comparing metric distributions across prompt versions and model changes, with built-in caching proxy for cost optimization.

# Braintrust

Braintrust approaches [[LLM Evaluation Frameworks]] from a data science perspective, focusing on experiment tracking and metric comparison across prompt versions, models, and datasets rather than pass/fail testing.

## Core Concepts

- **Project**: Logical grouping for AI features (e.g., "support-bot")
- **Experiment**: Single eval run capturing (prompt version, model, dataset) → scores
- **Dataset**: Curated input/expected output pairs persisting across experiments
- **Scores**: Numeric 0-1 metrics attached to each trace

## Implementation

```python
import braintrust
from braintrust import Eval
from autoevals import Factuality, LLMClassifier

Eval(
    "customer-support-bot",
    data=lambda: [
        {"input": "How do I reset password?", "expected": "Visit account settings"}
    ],
    task=lambda input: run_support_bot(input["input"]),
    scores=[
        Factuality,
        LLMClassifier(
            name="Helpful",
            prompt_template="Is this helpful? {{output}}",
            choice_scores={"Yes": 1, "No": 0}
        )
    ]
)
```

## Experiment Comparison

Braintrust's killer feature is experiment diffing - showing which test cases improved or regressed between runs and distribution shifts in each score. This enables confident answers to questions like "did switching from GPT-4o to Claude improve quality?" with statistical evidence.

## Caching Proxy

Braintrust ships a caching proxy that sits in front of LLM providers, caching responses by (model, prompt) hash. During eval runs with multiple metrics, this reduces costs by 60-80% by avoiding redundant API calls for identical prompts.

---
*Related: [[LLM Evaluation Frameworks]], [[RAGAS]], [[DeepEval]], [[Prompt Engineering]], [[AI Gateway]]*
