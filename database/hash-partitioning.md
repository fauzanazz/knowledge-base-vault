---
title: "Hash Partitioning"
category: database
summary: "Hash partitioning uses hash functions to deterministically map keys to partitions, providing uniform distribution but losing sorted order. Consistent hashing enables efficient rebalancing."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.243Z
---

# Hash Partitioning

> Hash partitioning uses hash functions to deterministically map keys to partitions, providing uniform distribution but losing sorted order. Consistent hashing enables efficient rebalancing.

# Hash Partitioning

Hash partitioning uses hash functions to deterministically map keys to partitions, providing uniform distribution but losing sorted order. Consistent hashing enables efficient rebalancing.

## How Hash Partitioning Works

A hash function deterministically maps keys to seemingly random numbers, which are then assigned to partitions. Good hash functions distribute keys uniformly across partitions, avoiding the hotspot problems common in [[Range Partitioning]].

## Partition Assignment Strategies

**Naive Approach**: Use the modulo operator to assign hash values to partitions. The major problem is that rebalancing (adding/removing partitions) requires moving almost all data since hash % (n+1) differs significantly from hash % n.

**Consistent Hashing**: A superior approach where partitions and keys are distributed on a virtual circle. Each key is assigned to the closest partition appearing clockwise on the circle. Adding a new partition only requires rebalancing keys that are now closer to the new partition.

## Trade-offs

**Advantages**:
- Uniform key distribution across partitions
- Eliminates hotspots from uneven data access
- Predictable performance characteristics

**Disadvantages**:
- Sorted order is lost, making range queries inefficient
- Cannot perform efficient range scans across the dataset
- Requires full table scans for range-based queries

## Mitigation Strategies

The loss of sorted order can be partially mitigated by:
- Sorting data within each partition based on a secondary key
- Using compound partitioning strategies
- Implementing secondary indexes for range queries

## Prerequisites

Hash partitioning requires a large number of possible keys. Boolean or small enum values are inappropriate for hash partitioning since they don't provide sufficient distribution.

Hash partitioning works particularly well for [[Caching]] systems since they typically store key-value pairs and don't require cross-partition joins or transactions.

---
*Related: [[Range Partitioning]], [[Database Sharding]], [[Caching]], [[System Scaling]]*
