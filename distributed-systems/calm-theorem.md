---
title: "CALM Theorem"
category: distributed-systems
summary: "The CALM theorem states that monotonic programs can achieve consistency without coordination, providing a framework for building coordination-free distributed applications."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.479Z
---

# CALM Theorem

> The CALM theorem states that monotonic programs can achieve consistency without coordination, providing a framework for building coordination-free distributed applications.

# CALM Theorem

The CALM (Consistency As Logical Monotonicity) theorem provides a fundamental principle for determining when distributed applications need coordination. It states that a program can be consistent without coordination if and only if it is **monotonic**.

## Monotonicity Definition

A program is **monotonic** if new inputs only refine the output rather than revert it to a previous state:

**Monotonic Examples**:
- Counter with increment operations: `inc(1), inc(2) = 3` equals `inc(2), inc(1) = 3`
- Set union operations
- Maximum value computations

**Non-monotonic Examples**:
- Variable assignment: `set(5), set(3) = 3` differs from `set(3), set(5) = 5`
- Subtraction operations
- Arbitrary state updates

## Key Insights

**Coordination-Free Consistency**: Monotonic programs can achieve consistency, availability, and partition tolerance simultaneously without requiring [[Consensus Algorithms|consensus]] or coordination protocols.

**Transformation Strategy**: Non-monotonic operations can often be made monotonic through design changes:
- Transform registers into Last-Writer-Wins or Multi-Value registers
- Use [[Conflict-Free Replicated Data Types|CRDTs]] for automatic conflict resolution
- Design append-only data structures

## Consistency Context

CALM's definition of consistency differs from the [[CAP Theorem]]:
- **CAP consistency**: Refers to read/write consistency across replicas
- **CALM consistency**: Refers to program output consistency

This distinction enables building applications that are consistent at the application level while using eventually consistent storage systems.

## Practical Applications

CALM theorem guides architectural decisions by identifying when coordination can be avoided, leading to more scalable and available distributed systems through careful program design.

---
*Related: [[Consistency Models]], [[Conflict-Free Replicated Data Types]], [[CAP Theorem]], [[Consensus Algorithms]]*
