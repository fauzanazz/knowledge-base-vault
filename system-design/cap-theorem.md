---
title: "CAP Theorem"
category: system-design
summary: "The CAP theorem states that distributed systems can only guarantee two of three properties: Consistency, Availability, and Partition tolerance. Since network partitions are inevitable, systems must choose between consistency and availability."
sources:
  - raw/articles/key-value-store-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:40:37.039Z
---

# CAP Theorem

> The CAP theorem states that distributed systems can only guarantee two of three properties: Consistency, Availability, and Partition tolerance. Since network partitions are inevitable, systems must choose between consistency and availability.

# CAP Theorem

The **CAP theorem** is a fundamental principle in distributed systems stating that only two of three guarantees can be achieved simultaneously:

## Three Properties

### Consistency (C)
All clients see the same data simultaneously across all nodes in the system.

### Availability (A)
The system responds to every request, even when some nodes are down or unreachable.

### Partition Tolerance (P)
The system continues to operate despite network partitions that prevent communication between nodes.

## System Classifications

### CP Systems
**Consistency + Partition Tolerance**
- Sacrifice availability during network partitions
- Block operations to maintain data consistency
- Examples: Banking systems, financial applications
- Use case: When data accuracy is critical

### AP Systems
**Availability + Partition Tolerance**
- Sacrifice consistency for continued operation
- Accept potentially stale data during partitions
- Implement eventual consistency models
- Examples: Social media feeds, content delivery
- Use case: When system uptime is prioritized

### CA Systems
**Consistency + Availability**
- Cannot exist in real-world distributed systems
- Network partitions are inevitable in distributed environments
- Only theoretical in single-node systems

## Practical Implications

Since network failures are unavoidable in distributed systems, **partition tolerance is mandatory**. This forces a choice between:

**Choosing Consistency (CP)**:
- Block write operations during partitions
- Prevent data inconsistency
- Risk system unavailability

**Choosing Availability (AP)**:
- Continue accepting reads and writes
- Accept temporary data inconsistency
- Synchronize data when partition resolves

## Real-World Applications

Most modern distributed systems implement **eventual consistency** models, accepting temporary inconsistencies in favor of availability. Systems like [[Key-Value Store]] implementations often choose AP characteristics with tunable consistency levels through techniques like quorum consensus.

The CAP theorem guides architectural decisions in designing [[Distributed Systems]], influencing choices in [[Database Replication]], [[Consistent Hashing]], and fault tolerance strategies.

---
*Related: [[Key-Value Store]], [[Distributed Systems]], [[Database Replication]], [[Consistent Hashing]], [[Eventual Consistency]]*
