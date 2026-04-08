---
title: "Guardrails AI Framework"
category: machine-learning
summary: "Guardrails AI is a Python framework that provides structured validation for LLM inputs and outputs through a hub of 50+ pre-built validators and automatic retry-with-correction capabilities."
sources:
  - raw/articles/guardrails-safety-input-output-filtering.md
updated: 2026-04-08T19:22:50.918Z
---

# Guardrails AI Framework

> Guardrails AI is a Python framework that provides structured validation for LLM inputs and outputs through a hub of 50+ pre-built validators and automatic retry-with-correction capabilities.

# Guardrails AI Framework

Guardrails AI is a Python framework that provides structured validation for LLM inputs and outputs through a `Guard` abstraction and hub of 50+ pre-built validators. It excels at output format validation with automatic retry-with-correction loops.

## Core Architecture

The framework centers around `Guard` objects that chain multiple validators:

```python
from guardrails import Guard
from guardrails.hub import ValidJson, ToxicLanguage, DetectPII

guard = Guard().use_many(
    ValidJson(on_fail="fix"),
    ToxicLanguage(threshold=0.5, on_fail="exception"),
    DetectPII(pii_entities=["EMAIL_ADDRESS"], on_fail="fix")
)
```

## Key Features

**Retry-with-Correction** is Guardrails' signature capability. When validation fails, it automatically constructs a correction prompt and re-runs the LLM up to `max_retries`. This makes output validation resilient without cascading application errors.

**Structured Output Validation** enforces Pydantic schemas and JSON formats, ensuring LLM responses conform to application contracts. The framework can automatically fix minor formatting issues or retry with specific correction instructions.

**Validator Hub** provides 50+ pre-built validators covering toxicity detection, PII scanning, format validation, length constraints, and domain-specific rules. Custom validators can be easily added.

## Production Integration

Guardrails integrates with major LLM providers and can wrap existing API calls:

```python
validated_output, metadata = guard(
    llm_api=openai_client.chat.completions.create,
    prompt="Generate a product recommendation...",
    model="gpt-4o",
    max_retries=3
)
```

## Use Cases

Guardrails AI is particularly effective for applications requiring structured outputs, content safety validation, and reliable format compliance. It's commonly used in customer service bots, content generation pipelines, and data extraction workflows where output consistency is critical.

The framework forms a key component of comprehensive [[AI Guardrails]] strategies, especially for output validation layers.

---
*Related: [[AI Guardrails]], [[LLM Evaluation Frameworks]], [[Prompt Engineering]]*
