---
title: "Conflict-Free Replicated Data Types"
category: distributed-systems
summary: "CRDTs are data structures that automatically resolve conflicts in distributed systems without requiring consensus, enabling strong eventual consistency through mathematical properties."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.020Z
---

# Conflict-Free Replicated Data Types

> CRDTs are data structures that automatically resolve conflicts in distributed systems without requiring consensus, enabling strong eventual consistency through mathematical properties.

# Conflict-Free Replicated Data Types

Conflict-free replicated data types (CRDTs) are specialized data structures that enable [[Database Replication]] without requiring [[Consensus]] to resolve conflicts. They achieve **strong eventual consistency** by ensuring replicas automatically converge to the same state.

## Core Properties

CRDTs work by satisfying two mathematical conditions:

1. **Semilattice Structure**: The object's possible states form a semilattice where elements can be partially ordered
2. **Merge Operation**: The merge function must be:
   - **Idempotent**: x + x = x
   - **Commutative**: x + y = y + x  
   - **Associative**: (x + y) + z = x + (y + z)

These properties ensure that merging states in any order produces the same result, even with message reordering or duplication.

## Types and Examples

**Convergent CRDTs** include:
- **Registers**: Memory cells with byte arrays
  - Last-Writer-Wins (LWW): Uses timestamps with [[Logical Clocks]]
  - Multi-Value (MV): Stores concurrent updates using [[Vector Clocks]]
- **Counters**: Support increment operations that converge to global maximum
- **Sets**: Enable add/remove operations with conflict resolution
- **Dictionaries**: Composed CRDTs for complex data structures

## Benefits

CRDTs eliminate the need for coordination during normal operation, providing:
- Better availability than total order broadcast
- Higher performance by avoiding consensus on the critical path
- Partition tolerance with automatic conflict resolution

CRDTs are fundamental to [[Dynamo-style Data Stores]] and enable building systems that satisfy the [[CAP Theorem]] by choosing availability and partition tolerance while maintaining eventual consistency.

---
*Related: [[Database Replication]], [[Consensus]], [[Consistency Models]], [[Vector Clocks]], [[Logical Clocks]]*
