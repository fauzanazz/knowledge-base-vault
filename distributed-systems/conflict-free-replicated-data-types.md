---
title: "Conflict-Free Replicated Data Types"
category: distributed-systems
summary: "CRDTs are data structures that automatically resolve conflicts in distributed systems without requiring consensus, enabling strong eventual consistency through mathematical properties."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.475Z
---

# Conflict-Free Replicated Data Types

> CRDTs are data structures that automatically resolve conflicts in distributed systems without requiring consensus, enabling strong eventual consistency through mathematical properties.

# Conflict-Free Replicated Data Types

Conflict-free replicated data types (CRDTs) are specialized data structures that enable automatic conflict resolution in distributed systems without requiring [[Consensus Algorithms|consensus]]. They achieve **strong eventual consistency** by ensuring replicas converge to the same state when they have applied the same updates.

## Core Requirements

For a data type to be a CRDT, it must satisfy two mathematical conditions:

1. **Semilattice Structure**: The object's possible states form a semilattice where elements can be partially ordered
2. **Merge Operation Properties**: The merge operation must be:
   - **Idempotent**: x + x = x
   - **Commutative**: x + y = y + x  
   - **Associative**: (x + y) + z = x + (y + z)

These properties ensure that merging states works correctly regardless of message order or duplicate deliveries.

## Types and Examples

**Convergent CRDTs** include:
- **Registers**: Memory cells storing byte arrays
  - Last-Writer-Wins (LWW) registers use timestamps
  - Multi-Value (MV) registers store concurrent updates
- **Counters**: Support increment operations that converge to global maximum
- **Sets**: Collections that merge through union operations
- **Dictionaries**: Key-value stores composed of CRDT values

## Advantages

CRDTs offer significant benefits over traditional [[Consistency Models|consistency]] approaches:
- No coordination required for conflict resolution
- Better availability and performance than total order broadcast
- Can use unreliable broadcast protocols with anti-entropy mechanisms
- Conflicts resolved by design rather than consensus

CRDTs enable building highly available systems that maintain consistency without the coordination overhead of traditional distributed consensus protocols.

---
*Related: [[Consistency Models]], [[Database Replication]], [[Consensus Algorithms]], [[Eventual Consistency]]*
