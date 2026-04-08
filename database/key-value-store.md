---
title: "Key-Value Store"
category: database
summary: "A key-value store is a non-relational database that stores data as unique key-value pairs, providing simple get/put operations with high scalability and availability. It forms the foundation for many distributed systems requiring fast data access."
sources:
  - raw/articles/key-value-store-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:40:37.037Z
---

# Key-Value Store

> A key-value store is a non-relational database that stores data as unique key-value pairs, providing simple get/put operations with high scalability and availability. It forms the foundation for many distributed systems requiring fast data access.

# Key-Value Store

A **key-value store** is a type of non-relational database where data is stored as key-value pairs. Each key is unique, and values are accessed using these keys through simple operations:

- `put(key, value)` - Insert or update data
- `get(key)` - Retrieve data

## Single Server Implementation

A basic key-value store uses a **hash table** to store pairs in memory. Optimizations include:
- Data compression
- Storing less frequently accessed data on disk

However, single server memory limitations require distributed approaches for scalability.

## Distributed Key-Value Store

Distributed key-value stores partition data across multiple servers and must address the **[[CAP Theorem]]** trade-offs:

- **CP Systems**: Prioritize consistency and partition tolerance (e.g., banking systems)
- **AP Systems**: Prioritize availability and partition tolerance with eventual consistency
- **CA Systems**: Cannot exist in real-world distributed systems due to inevitable network partitions

## Core Components

### Data Partitioning
Uses **[[Consistent Hashing]]** to distribute data evenly across servers, enabling automatic scaling and supporting heterogeneous server capacities through virtual nodes.

### Data Replication
Replicates data across N servers for high availability, placing replicas in distinct data centers for improved reliability.

### Consistency Models
Implements **quorum consensus** with parameters:
- `N`: Total replicas
- `W`: Write quorum size
- `R`: Read quorum size
- Rule: `W + R > N` ensures strong consistency

### Conflict Resolution
Uses **vector clocks** to track data versions and resolve conflicts when replicas diverge. Vector clocks are [server, version] pairs that detect whether versions are ancestors, descendants, or siblings (conflicting).

### Failure Handling
- **Detection**: Uses gossip protocol with heartbeat counters
- **Temporary Failures**: Implements sloppy quorum and hinted handoff
- **Permanent Failures**: Uses Merkle trees for efficient replica synchronization

## Architecture

The system features:
- Decentralized design with no single point of failure
- Coordinator nodes acting as proxies between clients and storage
- Ring-based topology using [[Consistent Hashing]]
- Multiple [[Database Replication]] strategies
- Write path: commit log → memory cache → SSTable on disk
- Read path: memory cache → Bloom filter → SSTable retrieval

Key-value stores power many large-scale applications requiring high availability and low latency data access.

---
*Related: [[Consistent Hashing]], [[Database Replication]], [[CAP Theorem]], [[Distributed Systems]], [[NoSQL]]*
