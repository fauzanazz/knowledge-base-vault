---
title: "Range Partitioning"
category: database
summary: "Range partitioning splits data into lexicographically sorted partitions, enabling efficient range queries but facing challenges with uneven data distribution and hotspots."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.241Z
---

# Range Partitioning

> Range partitioning splits data into lexicographically sorted partitions, enabling efficient range queries but facing challenges with uneven data distribution and hotspots.

# Range Partitioning

Range partitioning splits data into lexicographically sorted partitions, enabling efficient range queries but facing challenges with uneven data distribution and hotspots.

## How Range Partitioning Works

Data is divided into partitions based on key ranges, with each partition containing a contiguous range of sorted keys. Within each partition, data is kept in sorted order on disk to optimize range scanning operations.

## Key Challenges

**Uneven Distribution**: Simply dividing key ranges evenly doesn't account for real-world usage patterns. For example, partitioning alphabetically by first letter results in unbalanced partitions since some letters are used more frequently in English.

**Hotspots**: When most requests target a specific range, they hit the same partition. A common example is partitioning by date where current-day requests always hit the same partition. A workaround involves adding random prefixes, though this increases complexity.

## Rebalancing Strategies

As data grows or shrinks, partitions need rebalancing while minimizing data movement:

**Static Partitioning**: Define many partitions initially, with nodes serving multiple partitions rather than one. Drawbacks include fixed partition sizes and the trade-off between too many partitions (decreased performance) and too few (limited scalability).

**Dynamic Partitioning**: Start with a single partition and rebalance into sub-partitions over time. When data grows, partitions split into two. When data shrinks, sub-partitions merge. This approach adapts to actual usage patterns.

## Use Cases

Range partitioning works well for:
- Time-series data requiring range queries
- Applications needing sorted data access
- Systems where range scans are common operations

Range partitioning is often combined with [[Hash Partitioning]] in hybrid approaches to balance the benefits of both strategies.

---
*Related: [[Hash Partitioning]], [[Database Sharding]], [[System Scaling]]*
