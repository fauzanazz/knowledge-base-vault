---
title: "Partitioning Strategies"
category: system-design
summary: "Partitioning strategies determine how data is split across multiple servers, with range partitioning using lexicographic ordering and hash partitioning using hash functions for uniform distribution."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:37:36.743Z
---

# Partitioning Strategies

> Partitioning strategies determine how data is split across multiple servers, with range partitioning using lexicographic ordering and hash partitioning using hash functions for uniform distribution.

# Partitioning Strategies

Partitioning strategies determine how data is split across multiple servers when a single server can no longer handle the data volume or load. Two main mechanisms are used: range partitioning and hash partitioning.

## Range Partitioning

Range partitioning splits data into lexicographically sorted partitions. Data within each partition is kept in sorted order on disk to enable efficient range scanning.

**Challenges:**
- **Uneven distribution**: Simply splitting keys evenly doesn't account for usage patterns (e.g., some letters in the alphabet are used more frequently)
- **Hotspots**: Time-based partitioning can create hotspots when most requests target recent data
- **Rebalancing complexity**: Adding/removing nodes requires careful data movement

**Rebalancing Solutions:**
- **Static partitioning**: Create many partitions initially, with nodes serving multiple partitions
- **Dynamic partitioning**: Start with one partition and split/merge as data grows/shrinks

## Hash Partitioning

Hash partitioning uses a hash function to deterministically map keys to seemingly random numbers, which are then assigned to partitions. Hash functions typically distribute keys uniformly.

**Consistent Hashing:**
The naive approach using modulo operations requires moving almost all data during rebalancing. Consistent hashing solves this by:
- Distributing partitions and keys randomly on a circle
- Assigning each key to the closest partition clockwise
- Only rebalancing keys affected by the new partition when adding nodes

**Trade-offs:**
- **Advantage**: Uniform distribution and efficient rebalancing
- **Disadvantage**: Loses sorted order (can be mitigated with secondary sorting within partitions)

## Implementation Considerations

Partitioning requires a large key space and introduces complexity through gateway services, cross-partition operations, and rebalancing challenges.

---
*Related: [[Database Sharding]], [[Load Balancer]], [[System Scaling]], [[Dynamo-Style Data Stores]]*
