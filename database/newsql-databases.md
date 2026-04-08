---
title: "NewSQL Databases"
category: database
summary: "NewSQL databases combine the ACID guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems through advanced distributed consensus and partitioning techniques."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.481Z
---

# NewSQL Databases

> NewSQL databases combine the ACID guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems through advanced distributed consensus and partitioning techniques.

# NewSQL Databases

NewSQL databases represent a new generation of database systems that combine the [[ACID Properties]] guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems. They emerged to address the false choice between consistency and scalability.

## Evolution Context

**Historical Trade-off**:
- **SQL databases**: Strong ACID guarantees but limited scalability
- **NoSQL databases**: High scalability but weaker consistency guarantees
- **NewSQL**: Both strong consistency and horizontal scalability

## Google Spanner Architecture

Spanner exemplifies NewSQL design principles:

**Partitioning and Replication**:
- Key-value pairs divided into partitions for scalability
- Each partition replicated using [[Raft Algorithm|Paxos]] consensus
- Replication groups have designated leaders

**Transaction Processing**:
- Leaders apply writes through majority consensus
- [[Two-Phase Locking]] for intra-partition isolation
- [[Two-Phase Commit]] for multi-partition transactions
- Write-ahead logging for crash recovery

**Concurrency Control**:
- **MVCC** for read-only transactions (non-blocking)
- **2PL** for write transactions (maximum concurrency)
- Physical clock timestamps with uncertainty bounds

**Clock Synchronization**:
- GPS and atomic clocks in every data center
- Transactions wait for maximum clock uncertainty
- Enables globally consistent timestamps

## CockroachDB Alternative

CockroachDB provides similar guarantees without expensive atomic clocks:
- Uses hybrid logical clocks instead of physical clocks
- Similar architecture to Spanner but more cost-effective
- Demonstrates that NewSQL principles can be implemented practically

## Key Innovations

NewSQL databases achieve their goals through:
- Advanced distributed consensus protocols
- Sophisticated timestamp management
- Careful coordination of replication and partitioning
- Optimized concurrency control for distributed environments

These systems prove that the consistency-scalability trade-off can be overcome with sufficient engineering sophistication.

---
*Related: [[ACID Properties]], [[Database Sharding]], [[Raft Algorithm]], [[Two-Phase Commit]]*
