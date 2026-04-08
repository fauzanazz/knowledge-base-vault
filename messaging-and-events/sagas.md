---
title: "Sagas"
category: distributed-systems
summary: "Sagas provide a pattern for managing distributed transactions through a sequence of local transactions, each with compensating actions to handle failures without blocking resources."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.024Z
---

# Sagas

> Sagas provide a pattern for managing distributed transactions through a sequence of local transactions, each with compensating actions to handle failures without blocking resources.

# Sagas

Sagas provide an alternative to [[Two-Phase Commit]] for managing distributed transactions by breaking them into a sequence of local transactions, each with compensating actions for failure recovery.

## Core Concept

A saga consists of:
- **Local Transactions**: Individual operations on separate services
- **Compensating Transactions**: Reverse operations that undo completed steps
- **Guarantee**: Either all transactions succeed, or compensating transactions restore the original state

## Example: Travel Booking

Booking a trip requires coordinating multiple services:

**Forward Transactions**:
- T1: Book flight via airline API
- T2: Reserve hotel via booking API

**Compensating Transactions**:
- C1: Cancel flight reservation
- C2: Cancel hotel reservation

**Failure Handling**: If T2 fails, execute C1 to cancel the flight booking

## Implementation Patterns

**Orchestrator-Based**:
- Central coordinator manages transaction sequence
- Communicates via [[Message Queue]] for reliability
- Maintains state in persistent storage for crash recovery
- Handles retries and idempotency

**Choreography-Based**:
- Services coordinate through events
- Each service knows its next step and compensating action
- More decentralized but harder to monitor

## Workflow Engines

Production implementations often use workflow engines like:
- **Temporal**: Provides durable execution and state management
- **Apache Airflow**: For batch processing workflows
- **AWS Step Functions**: Cloud-native orchestration

## Trade-offs

**Benefits**:
- Non-blocking: No resource locking during execution
- Resilient: Handles long-running transactions
- Scalable: Works across organizational boundaries

**Limitations**:
- **No Isolation**: Intermediate states visible to other transactions
- **Complexity**: Requires careful compensating action design
- **Eventual Consistency**: System temporarily inconsistent during execution

## Isolation Workarounds

**Semantic Locks**: Mark data as "dirty" during saga execution
**Read Views**: Provide consistent snapshots hiding intermediate states
**Commutative Operations**: Design operations that work regardless of order

Sagas are essential for microservice architectures requiring distributed transaction coordination without the blocking behavior of traditional [[ACID Properties]] transactions.

---
*Related: [[Two-Phase Commit]], [[Message Queue]], [[ACID Properties]], [[Outbox Pattern]], [[System Resiliency]]*
