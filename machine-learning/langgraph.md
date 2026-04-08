---
title: "LangGraph"
category: machine-learning
summary: "LangGraph is a production-grade agent framework that models workflows as directed graphs with nodes as functions and edges as transitions. It supports state persistence, human-in-the-loop checkpoints, and complex branching logic."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.030Z
---

# LangGraph

> LangGraph is a production-grade agent framework that models workflows as directed graphs with nodes as functions and edges as transitions. It supports state persistence, human-in-the-loop checkpoints, and complex branching logic.

# LangGraph

LangGraph is the production-grade successor to LangChain for complex agent flows. It models agent workflows as **directed graphs** where nodes represent functions and edges represent transitions, supporting cycles, conditional branching, and human-in-the-loop checkpoints.

## Core Architecture

LangGraph uses a state machine approach where each node can read and modify a shared state object:

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    tool_calls: list
    iterations: int

def call_model(state: AgentState) -> AgentState:
    response = llm.invoke(state["messages"])
    return {"messages": state["messages"] + [response]}

def should_continue(state: AgentState) -> str:
    if state["messages"][-1].tool_calls and state["iterations"] < 10:
        return "tools"
    return END
```

## Key Features

### State Persistence
**Checkpointing** enables pause/resume execution that survives restarts. Critical for long-running workflows and error recovery.

### Human-in-the-Loop
Add `interrupt_before=["tools"]` to require human approval before tool execution. Essential for high-stakes operations.

### Streaming
Provides both token-level and node-level event streaming for real-time user feedback.

### Subgraphs
Compose graphs as nodes in parent graphs, enabling modular workflow design.

## When to Use

LangGraph is the production default for complex, stateful workflows requiring:
- Branching logic and cycles
- State persistence across sessions
- Human approval gates
- Observability through LangSmith
- Integration with existing LangChain ecosystem

Its graph model handles complexity that linear chains cannot manage effectively.

---
*Related: [[Agent Frameworks]], [[CrewAI]], [[AutoGen]], [[AI Memory Systems]], [[RAG Architecture]]*
