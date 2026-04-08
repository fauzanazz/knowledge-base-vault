---
title: "Isolation Levels"
category: database
summary: "Isolation levels define the degree to which transactions are isolated from each other, balancing consistency guarantees against performance through different concurrency control mechanisms."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.480Z
---

# Isolation Levels

> Isolation levels define the degree to which transactions are isolated from each other, balancing consistency guarantees against performance through different concurrency control mechanisms.

# Isolation Levels

Isolation levels define the degree to which database transactions are isolated from each other, providing a spectrum of trade-offs between consistency guarantees and performance.

## Concurrency Problems

Isolation levels address various race conditions:
- **Dirty Write**: Overwriting uncommitted changes
- **Dirty Read**: Reading uncommitted values  
- **Fuzzy Read**: Seeing different values in repeated reads
- **Phantom Read**: Query results changing due to concurrent modifications

## Serializability

**Strongest Level**: Serializability guarantees that executing transactions concurrently has the same effect as executing them sequentially. It provides the strongest consistency but requires significant coordination, reducing performance.

**Goal**: Maximize concurrency while preserving the appearance of serial execution.

## Concurrency Control Protocols

**Pessimistic Protocols** use locks to prevent conflicts:
- **Two-Phase Locking (2PL)**: Acquires read/write locks, releases after commit
- **Read locks**: Shared among multiple transactions, block write locks
- **Write locks**: Exclusive, prevent both read and write locks
- **Deadlock handling**: Detection and victim transaction abortion

**Optimistic Protocols** assume conflicts are rare:
- **Optimistic Concurrency Control (OCC)**: Execute without blocking, validate at commit
- **Local workspace**: Changes made locally, validated before commit
- **Conflict detection**: Restart transaction if workspace collides
- **Best for**: Read-heavy workloads with few conflicts

## Multi-Version Concurrency Control (MVCC)

**Read Optimization**: Addresses performance issues with read-only transactions:
- **Version tags**: Maintained for each record
- **Write operations**: Create new versions
- **Read operations**: Access newest version since transaction start
- **No blocking**: Read-only transactions don't conflict with writes

**Trade-offs**: Improved performance for reads at the cost of increased storage for version history.

MVCC is the most widely used concurrency control scheme in modern databases due to its performance benefits for mixed read-write workloads.

---
*Related: [[ACID Properties]], [[Database Transactions]], [[Concurrency Control]], [[Two-Phase Locking]]*
