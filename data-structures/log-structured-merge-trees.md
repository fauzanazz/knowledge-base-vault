---
title: "Log-Structured Merge Trees"
category: data-structures
summary: "Log-Structured Merge Trees (LSM) are data structures optimized for write-heavy workloads by storing data in memory until thresholds are reached, then merging to disk layers. They're used in databases like Cassandra, BigTable, and RocksDB."
sources:
  - raw/articles/_done/distributed-email-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:52:10.259Z
---

# Log-Structured Merge Trees

> Log-Structured Merge Trees (LSM) are data structures optimized for write-heavy workloads by storing data in memory until thresholds are reached, then merging to disk layers. They're used in databases like Cassandra, BigTable, and RocksDB.

# Log-Structured Merge Trees

Log-Structured Merge Trees (LSM) are data structures designed to optimize write performance in storage systems by using sequential writes and periodic merging operations.

## Core Concept

LSM trees store data in memory (memtable) until a predefined threshold is reached, then flush to disk in sorted order. Multiple disk levels are maintained, with periodic merging between levels to maintain efficiency.

## Architecture

**Memory Layer (Memtable)**
- Stores recent writes in memory
- Typically implemented as balanced trees or skip lists
- Provides fast write operations

**Disk Layers (SSTables)**
- Immutable sorted files on disk
- Multiple levels with increasing size limits
- Background compaction merges levels

## Write Process

1. Writes go to memtable in memory
2. When memtable fills, flush to Level 0 on disk
3. Background processes merge levels when size thresholds exceeded
4. Maintains sorted order across all levels

## Read Process

1. Check memtable first
2. Search disk levels from newest to oldest
3. Use bloom filters to avoid unnecessary disk reads
4. Merge results from multiple levels

## Advantages

- **Write optimization**: Sequential writes are much faster than random writes
- **Scalability**: Handles high write throughput efficiently
- **Compaction**: Removes deleted/outdated data automatically

## Use Cases

LSM trees are used in:
- **Cassandra**: Distributed NoSQL database
- **BigTable**: Google's distributed storage system
- **RocksDB**: Embedded key-value store
- **Custom search engines**: For write-heavy email indexing systems

LSM trees trade read performance for write performance, making them ideal for applications with heavy write workloads like logging, time-series data, and real-time analytics.

---
*Related: [[Database Sharding]], [[Caching]], [[Skip List]]*
