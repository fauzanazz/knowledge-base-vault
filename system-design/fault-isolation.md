---
title: "Fault Isolation"
category: system-design
summary: "Fault isolation reduces the blast radius of failures by partitioning systems so that faults in one partition don't affect others."
sources:
  - raw/articles/resiliency-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:18:59.153Z
---

# Fault Isolation

> Fault isolation reduces the blast radius of failures by partitioning systems so that faults in one partition don't affect others.

# Fault Isolation

Fault isolation reduces the blast radius of failures by partitioning systems so that problems in one partition don't affect others. This is crucial for handling correlated failures where [[Redundancy]] alone won't help.

## The Problem

Some faults affect all replicas simultaneously:
- **Poison Pills**: Malformed requests that crash servers due to bugs
- **Noisy Neighbors**: Resource-intensive requests that degrade performance for all users
- **Software Bugs**: Code defects that impact all instances running the same code

## Partitioning Strategy

By partitioning user requests across multiple isolated groups:
- A noisy neighbor only impacts users in their partition
- If 6 instances are split into 3 partitions, blast radius is reduced to 33%
- More partitions = smaller blast radius

This approach is called the **bulkhead pattern**, named after ship compartments that prevent water from flooding the entire vessel.

## Shuffle Sharding

Traditional partitioning can consistently impact unlucky users in degraded partitions. Shuffle sharding improves this by:
- Assigning instances to multiple "virtual partitions"
- Making it unlikely two users share the same partition
- Allowing clients to retry and hit different instances
- Providing partial degradation instead of consistent degradation

## Cellular Architecture

The most comprehensive isolation approach partitions the entire application stack:
- Each cell contains the full stack and dependencies
- Cells are completely independent
- Gateway service routes requests to appropriate cells
- Maximum cell capacity enables thorough testing and benchmarking
- Scaling adds new cells rather than expanding existing ones

Azure Storage exemplifies cellular architecture with independent storage clusters as cells.

## Benefits

- Limits failure blast radius
- Enables predictable scaling patterns
- Facilitates thorough testing of maximum capacity
- Provides isolation from various fault types

Fault isolation is essential when redundancy cannot address correlated failures affecting multiple system components simultaneously.

---
*Related: [[System Resiliency]], [[Redundancy]], [[Load Balancer]], [[System Scaling]]*
