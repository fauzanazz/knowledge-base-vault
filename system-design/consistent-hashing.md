---
title: "Consistent Hashing"
category: system-design
summary: "Consistent hashing is a distributed hashing technique that minimizes data redistribution when servers are added or removed from a system. It enables horizontal scaling by ensuring only a fraction of keys need to be remapped during topology changes."
sources:
  - raw/articles/consistent-hashing-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-08T19:05:33.471Z
---

# Consistent Hashing

> Consistent hashing is a distributed hashing technique that minimizes data redistribution when servers are added or removed from a system. It enables horizontal scaling by ensuring only a fraction of keys need to be remapped during topology changes.

# Consistent Hashing

Consistent hashing is a technique essential for achieving [[Horizontal Scaling]] by efficiently distributing requests and data across servers. Unlike traditional hashing methods like `serverIndex = hash(key) % N`, consistent hashing minimizes data redistribution when servers are added or removed.

## The Rehashing Problem

Traditional hashing methods cause significant issues when the server count changes:
- Removing a server causes most keys to be reassigned, leading to cache misses
- Adding a server results in unnecessary key redistributions
- This approach only works well when the server pool size is fixed

## How Consistent Hashing Works

### Hash Ring Structure
The hash space forms a continuous ring with values distributed from `0` to `2^160-1` (using hash functions like SHA-1). Both servers and keys are mapped onto this ring using the same hash function.

### Server Lookup
A key's server is determined by traversing clockwise on the ring until a server is found. This ensures deterministic placement while maintaining locality.

### Adding and Removing Servers
- **Adding a server**: Only keys between the new server and its predecessor are redistributed
- **Removing a server**: Only keys from the removed server are reassigned to the next server clockwise

## Virtual Nodes Solution

Basic consistent hashing can lead to uneven partition sizes and non-uniform key distribution. The solution is **virtual nodes**:
- Each physical server is represented by multiple virtual nodes distributed uniformly on the ring
- Virtual nodes improve key distribution and balance load
- As the number of virtual nodes increases, distribution becomes more balanced

## Benefits

- **Minimized Redistribution**: Only a fraction of keys are reassigned during topology changes
- **Scalability**: Enables efficient [[Horizontal Scaling]]
- **Mitigates Hotspots**: Balances data distribution to avoid server overload
- **Fault Tolerance**: Graceful handling of server failures

## Real-World Applications

Consistent hashing is widely used in distributed systems:
- Amazon DynamoDB
- Apache Cassandra
- Discord
- Akamai [[Content Delivery Network]]
- Maglev [[Load Balancer]]

Consistent hashing is fundamental to building scalable distributed systems that can grow and shrink dynamically without major disruptions.

---
*Related: [[Hash Partitioning]], [[Horizontal Scaling]], [[Load Balancer]], [[Database Sharding]], [[Dynamo-Style Data Stores]]*
