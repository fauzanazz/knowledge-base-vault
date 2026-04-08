---
title: "Database Replication"
category: database
summary: "Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability through various consistency models and replication strategies."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.872Z
---

# Database Replication

> Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability through various consistency models and replication strategies.

# Database Replication

Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability. It's fundamental for [[Distributed Systems]] that need to scale beyond single machines.

## Reasons for Replication

**Availability** - If data is stored on a single process and it goes down, the data becomes unavailable. Replication provides redundancy.

**Scalability** - More replicas allow supporting more concurrent clients by distributing read load.

**Performance** - Data can be placed closer to users geographically, reducing latency.

## Master-Slave Replication

The most common pattern involves:
- **Master** - Handles all write operations
- **Slaves** - Replicate data from master and serve read requests
- **Asynchronous replication** - Slaves eventually receive updates

## State Machine Replication

A stronger consistency approach where:
- Leader emits events for state changes
- All followers execute the same sequence of operations
- Results in identical state across all replicas
- Requires consensus protocols like [[Raft Algorithm]]

## Consistency Models

**Strong Consistency** - All reads/writes go through leader, guaranteeing latest state but limiting throughput

**Sequential Consistency** - Clients attached to specific followers see operations in same order but at different times

**Eventual Consistency** - Clients can use any replica, guaranteed to eventually see latest state but may see stale data temporarily

## Trade-offs

Replication involves fundamental trade-offs described by the CAP theorem - you can't simultaneously guarantee strong consistency, availability, and partition tolerance. The choice depends on application requirements.

---
*Related: [[Distributed Systems]], [[Consistency Models]], [[CAP Theorem]], [[Raft Algorithm]]*
