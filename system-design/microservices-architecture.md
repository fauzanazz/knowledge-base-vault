---
title: "Microservices Architecture"
category: system-design
summary: "Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:49:29.762Z
---

# Microservices Architecture

> Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity.

# Microservices Architecture

Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity.

## Monolith vs Microservices

**Monolithic downsides**:
- Components become increasingly coupled over time
- Codebase becomes too large to understand fully
- Changes require rebuilding and redeploying entire application
- Bugs in one component can impact unrelated components
- Deployment reverts affect all developers

**Microservices benefits**:
- Each service owned by small teams with less communication overhead
- Smaller surface area makes applications more digestible
- Teams can choose their preferred tech stack and hardware
- Independent deployment and scaling

## Best Practices

- APIs should have small surface area but encapsulate significant functionality
- Services shouldn't be too "micro" to avoid operational overhead
- Start with monolith and decompose when experiencing growing pains
- Enforce reasonable standardization while allowing flexibility

## Key Challenges

**Communication**: Remote calls are expensive and non-deterministic compared to in-process calls.

**Coupling**: Services must be loosely coupled to avoid distributed monoliths. Avoid fragile APIs, shared libraries requiring lockstep updates, and static IP addresses.

**Testing**: Integration testing becomes more complex with cross-service dependencies.

**Operations**: Requires sophisticated deployment automation and observability platforms for debugging.

**Eventual Consistency**: Data model spans multiple stores, making atomic updates challenging. Strong consistency has high cost, often requiring eventual consistency.

**Resource Provisioning**: Need standardized approaches for provisioning services with dedicated hardware and data stores.

Microservices are only worthwhile when benefits can be amortized across many development teams.

---
*Related: [[API Gateway]], [[Message Queue]], [[Load Balancer]], [[System Scaling]]*
