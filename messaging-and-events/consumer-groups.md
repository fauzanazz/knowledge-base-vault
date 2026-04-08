---
title: "Consumer Groups"
category: system-design
summary: "Consumer groups are sets of consumers working together to process messages from topics in distributed message queues, enabling parallel processing while maintaining ordering guarantees and automatic load balancing."
sources:
  - raw/articles/distributed-message-queue-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T08:54:30.219Z
---

# Consumer Groups

> Consumer groups are sets of consumers working together to process messages from topics in distributed message queues, enabling parallel processing while maintaining ordering guarantees and automatic load balancing.

# Consumer Groups

**Consumer groups** are fundamental coordination mechanisms in distributed [[Message Queue]] systems that enable multiple consumers to work together processing messages from topics.

## Core Functionality

Consumer groups provide **parallel processing** while maintaining message ordering guarantees. Each group maintains independent offsets, allowing multiple groups to consume the same messages at different rates.

**Key Properties**:
- Messages replicated per consumer group, not per individual consumer
- Each partition assigned to exactly one consumer within a group
- Consumer count cannot exceed partition count in a group
- Groups operate independently with separate offset tracking

## Coordinator Role

The **Consumer Group Coordinator** manages group membership and partition assignments:
- Found by hashing group name to ensure all group members connect to same broker
- Tracks consumer heartbeats and triggers rebalancing
- Facilitates leader election within groups
- Broadcasts partition assignment plans

## Rebalancing Process

Rebalancing occurs when consumers join, leave, or fail:

1. **Trigger Events**: Consumer joins/leaves, missed heartbeats, partition changes
2. **Leader Election**: Coordinator selects group leader among active consumers
3. **Assignment Calculation**: Leader generates new partition distribution plan
4. **Plan Distribution**: Coordinator broadcasts assignments to all members
5. **Consumption Resume**: Consumers begin processing newly assigned partitions

## Assignment Strategies

**Round-Robin**: Partitions distributed evenly across consumers in circular fashion.

**Range**: Consecutive partition ranges assigned to consumers.

**Sticky**: Minimizes partition reassignment during rebalancing to reduce disruption.

## State Management

Consumer groups maintain state in dedicated storage (typically ZooKeeper):
- Partition-to-consumer mappings
- Last committed offsets per partition
- Group membership information

This enables **fault tolerance** - if consumers crash, new consumers can resume from last committed offsets without message loss.

---
*Related: [[Message Queue]], [[Topic Partitioning]], [[Service Discovery]], [[Database Replication]]*
