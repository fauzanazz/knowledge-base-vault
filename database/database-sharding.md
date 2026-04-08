---
title: "Database Sharding"
category: database
summary: "Database sharding is a horizontal scaling technique that divides large databases into smaller, more manageable parts called shards. Each shard contains a subset of data and shares the same schema but stores unique data."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:10:47.576Z
---

# Database Sharding

> Database sharding is a horizontal scaling technique that divides large databases into smaller, more manageable parts called shards. Each shard contains a subset of data and shares the same schema but stores unique data.

# Database Sharding

Database sharding is a horizontal scaling technique that splits data across different servers when an application's data grows beyond what a single server can handle. It increases both storage capacity and load handling capability by dispersing requests across multiple servers.

## Implementation Requirements

Sharding requires:
- **Gateway service** - routes requests to appropriate shards (usually a reverse proxy)
- **Coordination service** - maintains partition-to-server mapping (e.g., Zookeeper, etcd)
- **Large key space** - the number of possible keys must be very large for effective distribution

## Partitioning Strategies

### Range Partitioning
Data is split into lexicographically sorted partitions. Data within partitions is kept sorted on disk for efficient range scanning.

**Challenges:**
- Uneven distribution (e.g., English alphabet letters have different frequencies)
- Hotspots when most requests target recent data (e.g., current date)
- Workaround: add random prefixes at the cost of complexity

### Hash Partitioning
Uses a hash function to deterministically map keys to seemingly random numbers assigned to partitions. Hash functions typically distribute keys uniformly but lose sorted order.

**Consistent Hashing** solves rebalancing issues by:
- Distributing partitions and keys randomly in a circle
- Assigning each key to the closest clockwise partition
- Minimizing data movement when adding/removing partitions

## Rebalancing Strategies

- **Static partitioning** - define many partitions initially, nodes serve multiple partitions
- **Dynamic partitioning** - start with single partition, split/merge based on data growth

## Complexity Challenges

Sharding introduces significant complexity including cross-partition joins, distributed transactions via [[Two-Phase Commit]], and handling hot partitions that become bottlenecks.

---
*Related: [[Database Replication]], [[System Scaling]], [[Two-Phase Commit]], [[Load Balancer]]*
