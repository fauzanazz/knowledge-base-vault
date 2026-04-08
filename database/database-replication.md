---
title: "Database Replication"
category: database
summary: "Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability. The most common pattern is the master-slave model where writes go to the master and reads are distributed across slaves."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.248Z
---

# Database Replication

> Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability. The most common pattern is the master-slave model where writes go to the master and reads are distributed across slaves.

# Database Replication

Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability. The most common pattern is the master-slave model where writes go to the master and reads are distributed across slaves.

## Leader-Follower Replication

In leader-follower replication:
- Clients send writes to the leader
- Leader persists changes in its write-ahead log
- Replicas connect to the leader and stream log entries
- Replicas can disconnect/reconnect, maintaining their log sequence number

## Benefits

- **Increased Read Capacity**: Distribute read queries across multiple replicas
- **Higher Availability**: System continues operating if leader fails
- **Query Isolation**: Run expensive analytics on followers without impacting primary workload

## Replication Modes

**Fully Synchronous**: Leader waits for all followers to acknowledge before responding to client. Highly consistent but not performant—a single slow replica slows all writes.

**Fully Asynchronous**: Leader immediately responds after persisting locally. Minimum latency but not fault tolerant—leader can crash after acknowledgment but before broadcast.

**Hybrid (Semi-synchronous)**: Some followers receive writes synchronously, others asynchronously. This is PostgreSQL's default behavior. Provides balance between consistency and performance.

## Failover Process

If a leader crashes, the system can fail over to a synchronous follower without data loss. This requires:
- Detection of leader failure
- Promotion of a follower to leader
- Redirection of client traffic
- Reconfiguration of remaining followers

## Limitations

Replication increases read capacity but still requires the database to fit on a single machine. For true horizontal scaling of both reads and writes, [[Database Sharding]] is necessary.

Replication also introduces eventual consistency challenges, where reads from followers may return stale data until replication catches up.

---
*Related: [[Database Sharding]], [[System Scaling]], [[ACID Properties]], [[NewSQL Databases]]*
