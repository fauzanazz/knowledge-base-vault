---
title: "NewSQL"
category: database
summary: "NewSQL databases combine the ACID guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems, exemplified by systems like Google Spanner and CockroachDB."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:34:00.075Z
---

# NewSQL

> NewSQL databases combine the ACID guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems, exemplified by systems like Google Spanner and CockroachDB.

# NewSQL

NewSQL represents a new generation of databases that combine the [[ACID Properties]] and strong consistency guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems.

## Historical Context

- **SQL era**: Strong consistency but limited scalability
- **NoSQL era**: High scalability but sacrificed ACID guarantees
- **NewSQL era**: Both strong consistency and horizontal scalability

## Google Spanner Architecture

Spanner pioneered the NewSQL approach:

**Partitioning**: Data is divided into partitions for horizontal scaling

**Replication**: Each partition is replicated using [[Raft Algorithm]] across multiple nodes

**Transaction Processing**:
- Leaders handle writes using state machine replication
- [[Two-Phase Commit]] coordinates multi-partition transactions
- [[Concurrency Control]] uses Two-Phase Locking for isolation
- MVCC enables high-performance read-only transactions

**Global Consistency**: Uses synchronized physical clocks (GPS + atomic clocks) with TrueTime API to assign globally consistent timestamps

## CockroachDB Alternative

CockroachDB provides similar capabilities without expensive atomic clocks:
- Uses hybrid logical clocks instead of TrueTime
- Implements similar partitioning and replication strategies
- More accessible for organizations without Google's infrastructure

## Key Innovations

- Combines consensus algorithms with traditional database techniques
- Leverages precise time synchronization for global consistency
- Demonstrates that the traditional consistency vs. scalability trade-off can be overcome

NewSQL systems prove that with sufficient engineering investment, distributed systems can achieve both strong consistency and horizontal scalability.

---
*Related: [[ACID Properties]], [[Raft Algorithm]], [[Two-Phase Commit]], [[Concurrency Control]], [[Physical Clocks]]*
