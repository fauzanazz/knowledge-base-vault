---
title: "Concurrency Control"
category: database
summary: "Concurrency control protocols manage concurrent access to shared data in databases, using pessimistic locking, optimistic validation, or multi-version approaches to maintain transaction isolation."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:34:00.075Z
---

# Concurrency Control

> Concurrency control protocols manage concurrent access to shared data in databases, using pessimistic locking, optimistic validation, or multi-version approaches to maintain transaction isolation.

# Concurrency Control

Concurrency control protocols manage how multiple transactions access shared data simultaneously while maintaining [[ACID Properties|isolation]]. The goal is maximizing concurrency while preserving the appearance of serial execution.

## Pessimistic Concurrency Control

**Two-Phase Locking (2PL)** is the most common pessimistic approach:
- **Read locks**: Multiple transactions can hold read locks simultaneously
- **Write locks**: Exclusive access, blocking both read and write operations
- **Two phases**: Growing phase (acquiring locks) and shrinking phase (releasing locks)

**Deadlock handling**: When transactions wait for each other's locks, the system detects deadlocks and aborts victim transactions.

## Optimistic Concurrency Control

**Optimistic Concurrency Control (OCC)** assumes conflicts are rare:
- Transactions work on local copies of data
- At commit time, the system validates that no conflicts occurred
- If conflicts are detected, the transaction restarts

Best suited for read-heavy workloads with infrequent writes.

## Multi-Version Concurrency Control

**MVCC** addresses limitations of both approaches:
- Maintains multiple versions of each record with version tags
- Read operations access the appropriate version without blocking
- Write operations use traditional pessimistic or optimistic control
- Read-only transactions never block or get aborted

## Performance Trade-offs

- **Pessimistic**: Better for conflict-heavy workloads, avoids wasted work
- **Optimistic**: Superior for read-heavy scenarios with rare conflicts
- **MVCC**: Optimal for mixed workloads, widely adopted despite storage overhead

MVCC has become the dominant approach in modern databases due to its excellent performance characteristics for typical application workloads.

---
*Related: [[ACID Properties]], [[Database Replication]], [[Two-Phase Commit]], [[NewSQL]]*
