---
title: "CALM Theorem"
category: distributed-systems
summary: "The CALM theorem states that monotonic programs can achieve consistency without coordination, providing a framework for building coordination-free distributed applications."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.022Z
---

# CALM Theorem

> The CALM theorem states that monotonic programs can achieve consistency without coordination, providing a framework for building coordination-free distributed applications.

# CALM Theorem

The CALM (Consistency and Logical Monotonicity) theorem provides guidance on when distributed applications need [[Consensus]] versus when they can use eventually consistent data stores.

## Core Principle

A program can be **consistent without coordination** if and only if it is **monotonic**.

## Monotonicity

An application is monotonic when new inputs only refine the output rather than reverting to previous states.

**Monotonic Examples**:
- Counter with increment operations: `inc(1), inc(2) = 3` equals `inc(2), inc(1) = 3`
- Set union operations
- Maximum value computations

**Non-monotonic Examples**:
- Variable assignment: `set(5), set(3) = 3` differs from `set(3), set(5) = 5`
- Arbitrary updates without ordering

## Making Programs Monotonic

Non-monotonic operations can be transformed:
- Convert registers to Last-Writer-Wins (LWW) registers using timestamps
- Use Multi-Value (MV) registers with [[Vector Clocks]]
- Apply [[Conflict-Free Replicated Data Types]] patterns

## CALM vs CAP

**CAP Theorem**: Focuses on read/write consistency in storage systems
**CALM Theorem**: Addresses application-level consistency and program output

CALM enables building applications that are:
- Consistent at the application level
- Available during partitions
- Partition tolerant
- Potentially inconsistent at the storage level

## Practical Implications

Monotonic programs can leverage:
- [[Dynamo-style Data Stores]] for high availability
- Eventually consistent storage without coordination overhead
- Distributed architectures that scale horizontally

CALM provides a theoretical foundation for coordination-free distributed computing, complementing the [[CAP Theorem]] by focusing on application semantics rather than storage consistency.

---
*Related: [[Consensus]], [[CAP Theorem]], [[Conflict-Free Replicated Data Types]], [[Consistency Models]], [[Vector Clocks]]*
