---
title: "Agent Frameworks"
category: machine-learning
summary: "Agent frameworks provide infrastructure for building LLM-powered agents that can reason, act, and iterate through tool usage. Major frameworks include LangChain/LangGraph, CrewAI, AutoGen, and OpenAI Swarm."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.029Z
---

# Agent Frameworks

> Agent frameworks provide infrastructure for building LLM-powered agents that can reason, act, and iterate through tool usage. Major frameworks include LangChain/LangGraph, CrewAI, AutoGen, and OpenAI Swarm.

# Agent Frameworks

Agent frameworks provide the infrastructure needed to build LLM-powered agents that follow the ReAct pattern (Reasoning + Acting). An agent is a control loop where a model decides what action to take next, executes tools, observes results, and iterates until a goal is met.

## Core Challenges

The hard part isn't the concept but the infrastructure: state management, tool routing, inter-agent communication, error recovery, and observability. Production agents require:

- **State persistence** across sessions
- **Tool execution** with error handling
- **Multi-agent coordination** for complex workflows
- **Human-in-the-loop** capabilities
- **Observability** and debugging tools

## Major Frameworks

### [[LangGraph]]
Production-grade framework modeling agent workflows as directed graphs with state management, checkpointing, and human-in-the-loop support.

### [[CrewAI]]
Role-based multi-agent framework where specialized agents work as teams with defined roles, goals, and backstories.

### [[AutoGen]]
Conversational multi-agent system with native code execution sandbox and teachable agents that learn from interactions.

### [[OpenAI Swarm]]
Minimalist framework built on handoff primitives for transferring control between agents. Designed for prototyping rather than production.

## Production Considerations

All production agents need **max_iterations** guards and **timeouts** on agentic loops. Unbounded agents will eventually hit LLM loops and run for minutes at significant cost. State persistence through checkpointers or memory systems is non-negotiable for multi-session workflows.

The choice between frameworks depends on workflow complexity, team expertise, and specific requirements like code execution or role-based decomposition.

---
*Related: [[LangGraph]], [[CrewAI]], [[AutoGen]], [[OpenAI Swarm]], [[RAG Architecture]], [[AI Memory Systems]]*
