---
title: "CrewAI"
category: machine-learning
summary: "CrewAI is a role-based multi-agent framework that models agent systems as crews of specialized agents with defined roles, goals, and backstories. Agents can delegate tasks to each other in sequential or hierarchical processes."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.030Z
---

# CrewAI

> CrewAI is a role-based multi-agent framework that models agent systems as crews of specialized agents with defined roles, goals, and backstories. Agents can delegate tasks to each other in sequential or hierarchical processes.

# CrewAI

CrewAI models agent systems as **crews** — teams of specialized agents with defined roles, goals, and backstories. It's designed around the metaphor of workplace teams where agents have specific expertise and can delegate tasks to each other.

## Core Concepts

### Agents
Each agent has:
- **Role**: Their job title/specialization
- **Goal**: What they're trying to achieve
- **Backstory**: Context that shapes their behavior
- **Tools**: Functions they can execute

### Tasks
Work items with:
- **Description**: What needs to be done
- **Expected Output**: Format and content requirements
- **Context**: Dependencies on other tasks
- **Agent**: Who should execute it

## Example Implementation

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="Senior Research Analyst",
    goal="Uncover cutting-edge AI developments",
    backstory="Expert at synthesizing complex research papers",
    tools=[search_tool, read_tool]
)

writer = Agent(
    role="Technical Writer",
    goal="Write compelling technical summaries",
    backstory="Converts dense research into clear docs"
)

research_task = Task(
    description="Research latest RAG improvements",
    agent=researcher
)

write_task = Task(
    description="Write 500-word technical brief",
    agent=writer,
    context=[research_task]  # Receives research output
)

crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)
```

## Process Types

**Sequential**: Tasks execute in order with outputs passed between agents.

**Hierarchical**: A manager agent delegates tasks to workers and validates outputs.

## Memory System

- **Short-term memory**: In-context conversation history
- **Long-term memory**: SQLite-backed persistent storage
- **Entity memory**: Structured knowledge about entities

## When to Use

CrewAI excels when workflows naturally decompose into specialized roles (researcher → analyst → writer). It's faster to prototype than [[LangGraph]] for team-based workflows but less flexible for complex branching logic.

---
*Related: [[Agent Frameworks]], [[LangGraph]], [[AutoGen]], [[AI Memory Systems]]*
