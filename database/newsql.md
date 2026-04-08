---
title: "NewSQL"
category: database
summary: "NewSQL databases provide both ACID guarantees and horizontal scalability, combining the consistency of traditional SQL databases with the scale-out capabilities of NoSQL systems."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.023Z
---

# NewSQL

> NewSQL databases provide both ACID guarantees and horizontal scalability, combining the consistency of traditional SQL databases with the scale-out capabilities of NoSQL systems.

# NewSQL

NewSQL databases represent a new generation of database systems that provide both [[ACID Properties]] guarantees and horizontal scalability, bridging the gap between traditional SQL and NoSQL systems.

## Historical Context

**Traditional SQL**: Offered ACID guarantees but limited scalability
**NoSQL Era**: Sacrificed ACID for horizontal scalability
**NewSQL**: Combines strong consistency with scale-out architecture

## Google Spanner Architecture

Spanner exemplifies NewSQL design principles:

**Partitioning**: Breaks data into partitions for horizontal scaling
**Replication**: Each partition replicated using [[Raft Algorithm]] (Paxos)
**Leadership**: Each replication group has a leader handling writes
**Locking**: Leaders implement Two-Phase Locking (2PL) for isolation

**Transaction Handling**:
- Single-partition: Direct leader coordination
- Multi-partition: [[Two-Phase Commit]] with leader as coordinator
- Recovery: Write-Ahead Log (WAL) enables crash recovery

**Concurrency Control**:
- **MVCC**: Multi-Version Concurrency Control for read-only transactions
- **2PL**: Two-Phase Locking for write transactions
- **Timestamping**: Physical clocks with GPS/atomic clock synchronization

## Clock Synchronization

Spanner's innovation lies in timestamp management:
- Calculates maximum possible clock offset
- Transactions wait for offset duration to ensure ordering
- Requires highly accurate GPS and atomic clocks

## Alternative Approaches

**CockroachDB**: Uses hybrid-logical clocks instead of expensive atomic clocks, making NewSQL more accessible while maintaining similar guarantees.

## Benefits

- **ACID Compliance**: Full transactional guarantees
- **Horizontal Scaling**: Partition-based architecture
- **Strong Consistency**: Global transaction ordering
- **SQL Interface**: Familiar query language and tools

NewSQL systems demonstrate that the traditional trade-off between consistency and scalability can be overcome through sophisticated distributed systems engineering.

---
*Related: [[ACID Properties]], [[Raft Algorithm]], [[Two-Phase Commit]], [[Consistency Models]], [[Distributed Systems]]*
