---
title: "Causal Consistency"
category: distributed-systems
summary: "Causal consistency preserves happens-before relationships between operations while allowing concurrent operations to be observed in different orders, providing a middle ground between eventual and strong consistency."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.479Z
---

# Causal Consistency

> Causal consistency preserves happens-before relationships between operations while allowing concurrent operations to be observed in different orders, providing a middle ground between eventual and strong consistency.

# Causal Consistency

Causal consistency is a [[Consistency Models|consistency model]] that preserves happens-before relationships between operations while allowing concurrent operations to be observed in different orders. It represents the strongest consistency model that enables building highly available and partition-tolerant systems.

## Core Principles

**Partial Ordering**: Unlike strong consistency which imposes global ordering, causal consistency only orders causally related operations. Operations that don't have a happens-before relationship can be observed in any order.

**Client-Centric View**: Clients only care about happens-before relationships of operations concerning them, not all operations in the system.

## Example Scenario

Consider this sequence:
1. Client A writes value X (Operation A)
2. Client B reads value X (Operation B) 
3. Client B writes value Y (Operation C)
4. Client C writes unrelated value Z (Operation D)

**Causal Guarantees**:
- Operation C happens-after A (causally related)
- Any client observing C must also observe A first
- Operation D has no causal relationship with others
- Clients may observe A and D in any order

## COPS Implementation

COPS (Clusters of Order-Preserving Servers) demonstrates practical causal consistency:

**Dependency Tracking**:
- Clients maintain local key-version dictionaries
- Writes include dependency information
- Replicas assign versions and propagate writes

**Commit Protocol**:
- Writes aren't committed until all dependencies are satisfied
- Ensures causal ordering is preserved across replicas

## Trade-offs

**Benefits**:
- Stronger guarantees than [[Eventual Consistency]]
- Maintains high availability and partition tolerance
- Preserves intuitive ordering for related operations

**Limitations**:
- Potential data loss if replica fails before broadcasting
- More complex than eventual consistency
- Weaker than strong consistency for some use cases

Causal consistency provides an important middle ground for applications requiring ordered operations without full coordination overhead.

---
*Related: [[Consistency Models]], [[Logical Clocks]], [[Vector Clocks]], [[Eventual Consistency]]*
