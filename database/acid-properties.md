---
title: "ACID Properties"
category: database
summary: "ACID properties define the fundamental guarantees that database transactions must provide: Atomicity, Consistency, Isolation, and Durability for reliable data operations."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:22:31.022Z
---

# ACID Properties

> ACID properties define the fundamental guarantees that database transactions must provide: Atomicity, Consistency, Isolation, and Durability for reliable data operations.

# ACID Properties

ACID properties define the essential characteristics that database transactions must satisfy to ensure reliable and predictable behavior in data operations.

## The Four Properties

**Atomicity**: Transactions are all-or-nothing operations. Either all operations within a transaction succeed, or all fail, preventing partial updates that could leave the database in an inconsistent state.

**Consistency**: Transactions can only transition the database from one valid state to another. The database maintains its integrity constraints and business rules throughout the transaction lifecycle.

**Isolation**: Concurrent transactions appear to execute sequentially, preventing race conditions and ensuring that intermediate transaction states are not visible to other transactions.

**Durability**: Once a transaction commits, its changes are permanently stored and survive system failures, crashes, or power outages.

## Implementation Challenges

In single-node databases, ACID properties are straightforward to implement. However, [[Distributed Systems]] face additional complexity:

**Atomicity**: Requires [[Two-Phase Commit]] protocol across multiple nodes
**Consistency**: Must coordinate constraint checking across distributed data
**Isolation**: Needs distributed locking or optimistic concurrency control
**Durability**: Requires replication and distributed consensus

## Trade-offs

Strict ACID compliance often conflicts with:
- **Performance**: Strong guarantees require coordination overhead
- **Availability**: Strict consistency may block operations during failures
- **Scalability**: Coordination limits horizontal scaling

Modern systems often relax ACID properties:
- NoSQL databases trade consistency for availability
- [[NewSQL]] systems attempt to provide ACID at scale
- Microservices use [[Sagas]] for distributed transaction patterns

ACID properties remain fundamental for applications requiring strong data integrity, while distributed systems explore alternative consistency models based on application requirements.

---
*Related: [[Two-Phase Commit]], [[Distributed Systems]], [[NewSQL]], [[Sagas]], [[Consistency Models]]*
