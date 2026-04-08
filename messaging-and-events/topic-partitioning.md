---
title: "Topic Partitioning"
category: system-design
summary: "Topic partitioning is a scaling technique that divides message topics into smaller, manageable segments called partitions to enable parallel processing and horizontal scaling in distributed message queues."
sources:
  - raw/articles/distributed-message-queue-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T08:54:30.219Z
---

# Topic Partitioning

> Topic partitioning is a scaling technique that divides message topics into smaller, manageable segments called partitions to enable parallel processing and horizontal scaling in distributed message queues.

# Topic Partitioning

**Topic partitioning** is a fundamental scaling technique in distributed [[Message Queue]] systems that divides topics into smaller, manageable segments called partitions.

## Core Concepts

**Partitions** are subdivisions of topics that enable parallel processing and [[Horizontal Scaling]]. Each partition operates as an independent queue with FIFO ordering guarantees.

**Partition Keys** determine which partition a message lands in, typically using hash-based distribution: `hash(key) % numPartitions`. Common partition keys include user IDs to guarantee message ordering for specific users.

**Offsets** represent the position of messages within partitions, enabling precise message location via `topic`, `partition`, and `offset` coordinates.

## Distribution Strategy

Messages are evenly distributed across partitions to balance load. Servers hosting partitions are called **brokers**, which can host multiple partitions from different topics.

## Consumer Group Coordination

**Consumer Groups** enable parallel processing while maintaining ordering guarantees:
- Each partition can only be consumed by one consumer within a group
- Multiple consumer groups can read the same messages independently
- Consumer count cannot exceed partition count within a group

## Rebalancing

When consumers join or leave groups, **rebalancing** redistributes partition assignments:
1. Coordinator detects consumer changes via heartbeats
2. New group leader calculates partition assignment
3. Assignment broadcast to all group members
4. Consumers begin processing new partitions

## Scaling Considerations

**Adding Partitions**: New messages route to additional partitions while existing data remains in original partitions during transition.

**Removing Partitions**: More complex process requiring retention period management and consumer rebalancing after data expiration.

## Performance Impact

More partitions enable higher parallelism but increase coordination overhead. Optimal partition count balances throughput requirements with operational complexity.

---
*Related: [[Message Queue]], [[Horizontal Scaling]], [[Consistent Hashing]], [[Load Balancer]]*
