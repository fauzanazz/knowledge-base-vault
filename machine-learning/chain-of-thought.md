---
title: "Chain-of-Thought"
category: machine-learning
summary: "Chain-of-Thought (CoT) prompting asks LLMs to show their reasoning process before producing final answers, dramatically improving accuracy on multi-step reasoning tasks. It can be implemented as zero-shot or few-shot approaches."
sources:
  - raw/articles/prompt-engineering-patterns-applied.md
updated: 2026-04-08T19:20:06.459Z
---

# Chain-of-Thought

> Chain-of-Thought (CoT) prompting asks LLMs to show their reasoning process before producing final answers, dramatically improving accuracy on multi-step reasoning tasks. It can be implemented as zero-shot or few-shot approaches.

# Chain-of-Thought

Chain-of-Thought (CoT) prompting asks LLMs to show their reasoning process before producing final answers. This technique dramatically improves accuracy on multi-step reasoning tasks by making the model's thinking explicit.

## Implementation Approaches

**Zero-shot CoT**:
```
{question}

Let's think step by step.
```

**Few-shot CoT**:
```
Q: Roger has 5 tennis balls. He buys 2 more cans, each with 3 balls. How many balls?

A: Roger starts with 5 balls.
2 cans × 3 balls = 6 new balls.
5 + 6 = 11 total.
Answer: 11

Q: {question}

A:
```

## When CoT Helps Most

- Multi-step arithmetic or logical reasoning
- Code debugging (reasoning about execution flow)
- Complex classification with nuanced criteria
- Planning and decision-making tasks

## When CoT Hurts

- Simple lookups or factual recall (adds latency/tokens with no benefit)
- Creative generation tasks (overthinking constrains creativity)
- Latency-critical paths where speed matters more than accuracy

## Self-Consistency Enhancement

Run the same CoT prompt multiple times with temperature > 0, then take the majority vote on final answers. This improves accuracy by 10-20% on reasoning tasks at the cost of N× token spend.

```python
def self_consistent_answer(question: str, n: int = 5) -> str:
    answers = []
    for _ in range(n):
        response = llm.complete(cot_prompt(question), temperature=0.7)
        answer = extract_final_answer(response)
        answers.append(answer)
    return Counter(answers).most_common(1)[0][0]  # Majority vote
```

Use self-consistency sparingly—5× token cost is only justified for high-stakes decisions.

## Integration

CoT works well with [[Few-Shot Learning]] (show reasoning in examples), [[Tree of Thought]] (extend to multiple reasoning branches), and [[ReAct Pattern]] (combine reasoning with tool usage).

---
*Related: [[Few-Shot Learning]], [[Tree of Thought]], [[ReAct Pattern]], [[Prompt Engineering]]*
