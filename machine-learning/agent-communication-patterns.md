---
title: "Agent Communication Patterns"
category: machine-learning
summary: "Agent communication patterns define how multiple AI agents exchange information and coordinate work through shared state, message passing, events, and direct calls."
sources:
  - raw/articles/multi-agent-orchestration-patterns.md
updated: 2026-04-08T19:23:53.499Z
---

# Agent Communication Patterns

> Agent communication patterns define how multiple AI agents exchange information and coordinate work through shared state, message passing, events, and direct calls.

# Agent Communication Patterns

Agent communication patterns define the mechanisms by which multiple AI agents exchange information and coordinate work in [[Multi-Agent Orchestration Patterns]]. The choice of communication pattern significantly impacts system reliability, scalability, and complexity.

## Communication Mechanisms

### Shared Blackboard
Agents read and write to a shared state store (Redis, database). Best for asynchronous, loosely coupled agents that need to share complex state.

```python
# Example with Redis
board = redis.Redis()

def research_agent(task_id: str):
    results = do_research()
    board.set(f"research:{task_id}", json.dumps(results))
    board.publish("task_events", json.dumps({"type": "research_done", "task_id": task_id}))
```

### Message Passing
Explicit messages via queues (RabbitMQ, SQS, Kafka). Provides durable, auditable workflows with guaranteed delivery semantics.

### Event-Driven
Agents emit and subscribe to events. Ideal for reactive, trigger-based systems where agents respond to state changes.

### Direct Call
Synchronous function or API calls between agents. Best for low-latency, tightly coupled pipelines where immediate response is required.

### Shared State Object
Single Python/JavaScript object passed through agent chain. Simplest approach for in-process pipelines.

## Pattern Selection Criteria

**Latency Requirements:** Direct calls for real-time, message queues for batch processing.

**Durability Needs:** Message queues and blackboards provide persistence, direct calls do not.

**Coupling Tolerance:** Shared blackboards enable loose coupling, direct calls create tight coupling.

**Audit Requirements:** Message passing provides complete audit trails, shared state may not.

## Implementation Considerations

All communication patterns must handle network failures, agent crashes, and message loss. Implement timeouts, retries, and circuit breakers. Consider message ordering guarantees and exactly-once delivery semantics for critical workflows.

For production systems, instrument communication channels with metrics on message volume, latency, and error rates to detect bottlenecks and failures.

---
*Related: [[Multi-Agent Orchestration Patterns]], [[Message Queue]], [[Microservices Architecture]], [[System Models]]*
