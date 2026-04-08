---
title: "ACID Properties"
category: database
summary: "ACID properties define the fundamental guarantees that database transactions must provide: Atomicity, Consistency, Isolation, and Durability."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.479Z
---

# ACID Properties

> ACID properties define the fundamental guarantees that database transactions must provide: Atomicity, Consistency, Isolation, and Durability.

# ACID Properties

ACID properties define the fundamental guarantees that database transactions must provide to ensure data integrity and reliability in concurrent environments.

## The Four Properties

**Atomicity**: Transactions are all-or-nothing operations. Either all operations within a transaction succeed, or all fail. Partial failures are not possible, ensuring the database never remains in an intermediate state.

**Consistency**: Transactions can only transition a database from one valid state to another. The application cannot be left in an invalid state that violates business rules or data constraints.

**Isolation**: Concurrent execution of transactions doesn't cause race conditions. Transactions appear to execute sequentially even when running concurrently, preventing interference between simultaneous operations.

**Durability**: Once a transaction is committed, its changes are permanently persisted. The database guarantees that committed changes survive system failures, crashes, or power outages.

## Implementation Challenges

**Concurrency Issues** that isolation prevents:
- **Dirty writes**: Overwriting uncommitted changes from another transaction
- **Dirty reads**: Reading uncommitted values
- **Fuzzy reads**: Seeing different values when reading the same data twice
- **Phantom reads**: Query results changing due to concurrent insertions/deletions

**Isolation Levels**: Different levels provide varying protection against these issues, with stronger isolation reducing performance due to increased coordination requirements.

## Distributed Challenges

ACID properties become significantly more complex in distributed systems:
- Atomicity across multiple databases requires [[Two-Phase Commit]]
- Consistency must be maintained across network partitions
- Isolation requires coordination between distributed components
- Durability must account for network failures and partial commits

ACID properties form the foundation for reliable transaction processing in both centralized and distributed database systems.

---
*Related: [[Database Transactions]], [[Isolation Levels]], [[Two-Phase Commit]], [[Concurrency Control]]*
