---
title: "Causal Consistency"
category: distributed-systems
summary: "Causal consistency is the strongest consistency model that enables highly available and partition-tolerant systems by preserving happens-before relationships without requiring global ordering."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.022Z
---

# Causal Consistency

> Causal consistency is the strongest consistency model that enables highly available and partition-tolerant systems by preserving happens-before relationships without requiring global ordering.

# Causal Consistency

Causal consistency is a [[Consistency Models|consistency model]] that preserves happens-before relationships while allowing concurrent operations to be observed in different orders across replicas.

## Key Properties

**Partial Ordering**: Imposes order only on causally related operations, unlike strong consistency which requires global ordering
**Availability**: Enables highly available and partition-tolerant systems
**Happens-Before**: Guarantees that if operation B causally depends on operation A, all clients observe A before B

## Example Scenario

Consider these operations:
- Client A writes value X (operation A)
- Client B reads value X (operation B) 
- Client B writes value Y (operation C)
- Client C writes unrelated value Z (operation D)

Causal consistency guarantees:
- Operation A happens-before C (causally related)
- Operation D can be observed in any order relative to A and C (not causally related)

## COPS Implementation

COPS (Clusters of Order-Preserving Servers) demonstrates causal consistency:

**Client Operations**:
- Maintains local key-version dictionary tracking dependencies
- Sends dependency information with write requests

**Replica Behavior**:
- Assigns versions to writes
- Delays commit until all dependencies are satisfied
- Propagates writes to other replicas

**Trade-offs**:
- Potential data loss if replica fails before broadcasting
- Avoids waiting for long-distance requests before acknowledgment

## Benefits

Causal consistency provides:
- Stronger guarantees than eventual consistency
- Better availability than strong consistency
- Natural ordering for user-perceived causality
- Foundation for building intuitive distributed applications

This model bridges the gap between eventual and strong consistency, making it ideal for applications requiring intuitive ordering without sacrificing availability during network partitions.

---
*Related: [[Consistency Models]], [[Vector Clocks]], [[CAP Theorem]], [[Distributed Systems]], [[Logical Clocks]]*
