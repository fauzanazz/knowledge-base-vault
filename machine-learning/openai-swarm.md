---
title: "OpenAI Swarm"
category: machine-learning
summary: "OpenAI Swarm is a minimalist agent framework built on two primitives: agents (LLMs with instructions and tools) and handoffs (transferring control between agents). It's designed for prototyping rather than production use."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.031Z
---

# OpenAI Swarm

> OpenAI Swarm is a minimalist agent framework built on two primitives: agents (LLMs with instructions and tools) and handoffs (transferring control between agents). It's designed for prototyping rather than production use.

# OpenAI Swarm

Swarm is OpenAI's minimalist framework (released October 2024) built on two core primitives: **agents** (LLMs with instructions and tools) and **handoffs** (transferring control between agents). It's intentionally lightweight and designed as a reference implementation rather than a production framework.

## Core Concepts

### Agents
LLMs with specific instructions and available functions:

```python
from swarm import Swarm, Agent

client = Swarm()

def transfer_to_billing():
    return billing_agent

triage_agent = Agent(
    name="Triage",
    instructions="Route users to the correct department",
    functions=[transfer_to_billing, transfer_to_support]
)

billing_agent = Agent(
    name="Billing",
    instructions="Handle billing questions and refunds",
    functions=[process_refund, get_invoice]
)
```

### Handoffs
First-class primitives for transferring control between agents. Functions can return other agents to trigger handoffs:

```python
response = client.run(
    agent=triage_agent,
    messages=[{"role": "user", "content": "I need a refund"}]
)
```

## Design Philosophy

Swarm is intentionally thin with:
- No persistent state management
- No built-in memory systems
- No production retry logic
- No complex orchestration features

This minimalism makes it excellent for learning agent concepts and rapid prototyping, but unsuitable for production systems.

## Conceptual Influence

Despite its simplicity, Swarm's handoff model has influenced how other frameworks think about multi-agent routing. The idea of explicit control transfer between specialized agents appears in more sophisticated forms in [[CrewAI]] and [[LangGraph]].

## When to Use

Swarm is ideal for:
- Learning agent framework concepts
- Rapid prototyping of multi-agent interactions
- Simple routing pipelines with clear handoff semantics
- Educational purposes and experimentation

For production systems, consider [[LangGraph]], [[CrewAI]], or [[AutoGen]] which provide the state management, error handling, and observability features that production applications require.

---
*Related: [[Agent Frameworks]], [[LangGraph]], [[CrewAI]], [[AutoGen]]*
