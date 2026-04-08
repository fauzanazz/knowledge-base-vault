---
title: "ReAct Pattern"
category: machine-learning
summary: "ReAct (Reasoning + Acting) is the foundational pattern for LLM agents where models alternate between thinking, acting through tool calls, and observing results until reaching a goal. Most agent frameworks implement variants of this pattern."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.031Z
---

# ReAct Pattern

> ReAct (Reasoning + Acting) is the foundational pattern for LLM agents where models alternate between thinking, acting through tool calls, and observing results until reaching a goal. Most agent frameworks implement variants of this pattern.

# ReAct Pattern

ReAct (Reasoning + Acting) is the foundational pattern for LLM agents, introduced in Yao et al. 2022. It structures agent behavior as an iterative loop: **Think → Act → Observe** until the goal is achieved.

## Pattern Structure

The ReAct pattern follows this sequence:

1. **Thought**: The model reasons about what action to take next
2. **Action**: Execute a tool or function call
3. **Observation**: Process the results from the action
4. **Repeat**: Continue until the goal is met or max iterations reached

## Example Flow

```
Thought: I need to find the current price of AAPL.
Action: search_web("AAPL stock price")
Observation: AAPL is trading at $189.42.
Thought: I have the price. The user asked for comparison with MSFT.
Action: search_web("MSFT stock price")
Observation: MSFT is trading at $415.20.
Thought: I have both prices. I can answer.
Answer: AAPL ($189.42) vs MSFT ($415.20) — MSFT is ~2.2× higher.
```

## Implementation Across Frameworks

All major [[Agent Frameworks]] implement variants of ReAct:

- **[[LangGraph]]**: Models the pattern as graph nodes with conditional edges
- **[[CrewAI]]**: Embeds ReAct within role-based agent interactions
- **[[AutoGen]]**: Uses conversational turns to implement the reasoning loop
- **[[OpenAI Swarm]]**: Simplifies ReAct with handoff-based control flow

## Production Considerations

### Loop Guards
Always implement **max_iterations** limits and **timeouts**. LLMs can get stuck in reasoning loops that run for minutes at significant cost.

### Error Handling
The "Observe" step must handle tool failures, API timeouts, and malformed responses gracefully.

### State Management
Complex ReAct implementations need persistent state to survive interruptions and enable human-in-the-loop workflows.

## Variations

Some frameworks extend ReAct with:
- **Planning phases** that decompose goals before acting
- **Reflection steps** that evaluate progress and adjust strategy
- **Multi-agent coordination** where multiple ReAct loops interact

The ReAct pattern remains the core abstraction that makes LLM agents practical for real-world tool usage and goal-directed behavior.

---
*Related: [[Agent Frameworks]], [[LangGraph]], [[CrewAI]], [[AutoGen]], [[OpenAI Swarm]]*
