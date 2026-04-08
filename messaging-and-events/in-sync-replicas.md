---
title: "In-Sync Replicas"
category: system-design
summary: "In-Sync Replicas (ISR) are replicas in a distributed message queue that stay synchronized with the leader replica within acceptable lag thresholds, balancing data durability with system performance and availability."
sources:
  - raw/articles/_done/distributed-message-queue-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:56:33.775Z
---

# In-Sync Replicas

> In-Sync Replicas (ISR) are replicas in a distributed message queue that stay synchronized with the leader replica within acceptable lag thresholds, balancing data durability with system performance and availability.

# In-Sync Replicas

**In-Sync Replicas (ISR)** are replicas in a distributed [[Message Queue]] system that maintain synchronization with the leader replica within acceptable lag thresholds. ISR is a critical concept for balancing data durability, performance, and availability in distributed systems.

## Core Concept

In a replicated partition, only the **leader replica** accepts writes from producers. **Follower replicas** continuously pull new messages from the leader to stay synchronized. A follower is considered "in-sync" when its lag behind the leader is within acceptable limits.

## Lag Measurement

The primary metric for ISR membership is `replica.lag.max.messages`, which defines the maximum number of messages a replica can lag behind the leader while still being considered in-sync.

For example:
- Leader has committed offset 13 and received 2 new uncommitted messages (offset 14-15)
- Replica 2: Fully caught up (offset 15) - **In ISR**
- Replica 3: Fully caught up (offset 15) - **In ISR** 
- Replica 4: Lagging behind (offset 12) - **Removed from ISR**

## Message Commitment

A message is only **committed** (available for consumption) once all replicas in the ISR have successfully synchronized that message. This ensures data durability while maintaining system availability.

## Performance vs. Durability Trade-off

ISR represents a fundamental trade-off:

### High Durability Approach
- Wait for all replicas to sync before committing
- Risk: Slow replicas can make entire partition unavailable
- Benefit: Maximum data protection

### High Performance Approach
- Maintain smaller ISR with only fast replicas
- Risk: Reduced fault tolerance if ISR replicas fail
- Benefit: Faster message commitment

## Acknowledgment Configuration

Producers can configure acknowledgment requirements:

- **ACK=all**: All ISR replicas must acknowledge (highest durability, lowest performance)
- **ACK=1**: Only leader must acknowledge (medium durability and performance)
- **ACK=0**: No acknowledgment required (lowest durability, highest performance)

## ISR Management

The **leader replica** maintains the ISR list by:
- Continuously monitoring follower lag
- Adding replicas that catch up within lag thresholds
- Removing replicas that fall behind thresholds
- Updating the ISR list in the coordination service (e.g., [[Service Discovery]])

## Failure Scenarios

### Replica Failure
When an ISR replica fails:
1. Leader removes it from ISR immediately
2. Remaining ISR replicas continue normal operation
3. Messages commit when remaining ISR replicas acknowledge

### Leader Failure
When the leader fails:
1. New leader is elected from ISR replicas
2. Non-ISR replicas must catch up before rejoining ISR
3. System maintains availability with reduced replica count

## Configuration Considerations

- **Minimum ISR size**: Balance between availability and durability
- **Lag thresholds**: Tune based on network conditions and performance requirements
- **Replica placement**: Distribute ISR replicas across different brokers/data centers
- **Monitoring**: Track ISR size and replica lag metrics

## Benefits

- **Fault tolerance**: System continues operating despite replica failures
- **Performance optimization**: Excludes slow replicas from critical path
- **Flexible durability**: Configurable based on application requirements
- **Automatic recovery**: Replicas automatically rejoin ISR when they catch up

ISR is essential for maintaining both high availability and data consistency in distributed [[Message Queue]] systems, automatically adapting to changing network conditions and replica performance while preserving configurable durability guarantees.

---
*Related: [[Message Queue]], [[Database Replication]], [[Service Discovery]], [[Load Balancer]]*
