---
title: "Microservices Architecture"
category: system-design
summary: "Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:12:09.754Z
---

# Microservices Architecture

> Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity.

# Microservices Architecture

Microservices architecture functionally decomposes large monolithic codebases into independently deployable services that communicate via APIs. This creates hard boundaries between components that are difficult to violate.

## Benefits Over Monoliths

- **Team Autonomy**: Each service is owned by a small team, reducing cross-team communication overhead
- **Smaller Surface Area**: Applications become more digestible to engineers
- **Technology Freedom**: Teams can adopt preferred tech stacks and hardware
- **Independent Deployment**: Changes don't require rebuilding the entire application
- **Fault Isolation**: Bugs in one component don't impact unrelated components

## Key Challenges

### Communication Complexity
Remote calls are expensive and non-deterministic compared to in-process calls, introducing [[Distributed Systems]] complexity.

### Coupling Issues
Microservices must be loosely coupled to avoid creating a "distributed monolith" with downsides of both approaches. Examples of tight coupling include:
- Fragile APIs requiring client updates on every change
- Shared libraries updated in lockstep
- Static IP address references

### Operational Overhead
- **Resource Provisioning**: Need standardized approaches for hardware and data store provisioning
- **Testing**: Integration testing becomes more complex
- **Deployment**: Requires sophisticated deployment and observability platforms
- **Debugging**: Local debugging becomes challenging

### Data Consistency
The data model no longer resides in a single data store, making atomic updates challenging. Systems often fall back to [[Eventual Consistency]] rather than strong consistency.

## Best Practices

- APIs should have small surface areas but encapsulate significant functionality
- Services shouldn't be too "micro" to avoid excessive operational load
- Enforce reasonable technology standardization
- Start with a monolith and decompose gradually when experiencing growing pains

## When to Use

Microservices are only worthwhile when you can amortize the complexity cost across many development teams. It's usually best to start with a monolith and decompose it once there's sufficient reason.

---
*Related: [[API Gateway]], [[Message Queue]], [[Distributed Systems]], [[Load Balancer]], [[Eventual Consistency]]*
