---
title: "Tree of Thought"
category: machine-learning
summary: "Tree of Thought (ToT) extends Chain-of-Thought by exploring multiple reasoning branches in parallel, evaluating each path, and selecting the best solution. It's compute-intensive but valuable for complex planning and problem-solving tasks."
sources:
  - raw/articles/prompt-engineering-patterns-applied.md
updated: 2026-04-08T19:20:06.461Z
---

# Tree of Thought

> Tree of Thought (ToT) extends Chain-of-Thought by exploring multiple reasoning branches in parallel, evaluating each path, and selecting the best solution. It's compute-intensive but valuable for complex planning and problem-solving tasks.

# Tree of Thought

Tree of Thought (ToT) extends [[Chain-of-Thought]] by exploring multiple reasoning branches in parallel and selecting the best path. The model generates several "thoughts," evaluates them, and backtracks when a branch fails.

## Algorithm Structure

```
Question
├── Approach A → evaluate(A) → score: 0.8 → expand → ...
├── Approach B → evaluate(B) → score: 0.3 → prune
└── Approach C → evaluate(C) → score: 0.9 → expand → Solution
```

## Implementation Pattern

```python
def tree_of_thought(problem: str, breadth: int = 3, depth: int = 3) -> str:
    thoughts = generate_initial_thoughts(problem, n=breadth)
    
    for level in range(depth):
        evaluated = [(t, evaluate_thought(problem, t)) for t in thoughts]
        # Keep top-breadth thoughts
        thoughts = [t for t, score in sorted(evaluated, key=lambda x: -x[1])[:breadth]]
        # Expand surviving thoughts
        thoughts = [expand_thought(problem, t) for t in thoughts]
        if any(is_solution(t) for t in thoughts):
            break
    
    return best_solution(thoughts)
```

## Key Components

**Thought Generation**: Create multiple initial approaches to the problem, typically 3-5 diverse strategies.

**Evaluation**: Score each thought's promise using criteria like feasibility, completeness, or likelihood of success.

**Expansion**: Develop promising thoughts further while pruning low-scoring branches.

**Backtracking**: Return to earlier decision points when current paths fail.

## Best Use Cases

- **Planning Tasks**: Project planning, resource allocation, strategic decisions
- **Puzzle Solving**: Logic puzzles, mathematical proofs, optimization problems
- **Code Architecture**: System design decisions, algorithm selection
- **Creative Problem Solving**: When multiple valid approaches exist

## Cost Considerations

ToT is compute-intensive, requiring multiple LLM calls per level. A 3-breadth, 3-depth search needs ~27 LLM calls versus 1 for standard prompting. Use it surgically for high-stakes decisions where the improved accuracy justifies the cost.

## Integration

ToT combines with [[Self-Consistency]] (run multiple ToT searches and vote), [[Program-of-Thought]] (evaluate branches through code execution), and [[Agent Frameworks]] (implement as multi-step agent workflows).

---
*Related: [[Chain-of-Thought]], [[Program-of-Thought]], [[Agent Frameworks]], [[Prompt Engineering]]*
