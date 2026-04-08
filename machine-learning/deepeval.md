---
title: "DeepEval"
category: machine-learning
summary: "DeepEval is a general-purpose LLM evaluation framework with 14+ metrics and pytest integration, covering quality dimensions beyond RAG including bias, toxicity, and custom criteria evaluation."
sources:
  - raw/articles/eval-frameworks-ragas-deepeval-braintrust.md
updated: 2026-04-08T19:21:34.929Z
---

# DeepEval

> DeepEval is a general-purpose LLM evaluation framework with 14+ metrics and pytest integration, covering quality dimensions beyond RAG including bias, toxicity, and custom criteria evaluation.

# DeepEval

DeepEval is a comprehensive [[LLM Evaluation Frameworks]] tool covering the full spectrum of LLM quality dimensions with native pytest integration for seamless CI pipeline adoption.

## Key Metrics

| Metric | Purpose |
|--------|---------|
| `GEval` | Custom criteria via LLM-as-judge with chain-of-thought |
| `HallucinationMetric` | Factual correctness against ground truth |
| `AnswerRelevancyMetric` | Response relevance to input |
| `BiasMetric` | Gender, racial, political bias detection |
| `ToxicityMetric` | Harmful or offensive content identification |
| `JsonCorrectnessMetric` | Structural and semantic JSON validation |
| `SummarizationMetric` | Coverage and conciseness for summaries |

## pytest Integration

```python
import pytest
from deepeval import assert_test
from deepeval.metrics import GEval, HallucinationMetric
from deepeval.test_case import LLMTestCase

@pytest.mark.parametrize("test_case", load_test_cases())
def test_support_bot(test_case):
    output = run_bot(test_case.input)
    case = LLMTestCase(
        input=test_case.input,
        actual_output=output,
        expected_output=test_case.expected
    )
    assert_test(case, [HallucinationMetric(threshold=0.3)])
```

## G-Eval: Custom Criteria

G-Eval is DeepEval's most flexible metric, allowing natural language rubric definition. An LLM evaluates outputs against custom criteria with chain-of-thought reasoning, returning 0-1 scores. This handles domain-specific requirements like tone, brand voice, or legal compliance that don't fit pre-built metrics.

DeepEval slots naturally into existing testing workflows, making it ideal for teams already using pytest for traditional software testing.

---
*Related: [[LLM Evaluation Frameworks]], [[RAGAS]], [[Prompt Engineering]], [[Tool Calling]]*
