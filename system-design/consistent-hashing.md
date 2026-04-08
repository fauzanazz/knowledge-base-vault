---
title: "Consistent Hashing"
category: system-design
summary: "Consistent hashing is a distributed hashing technique that minimizes data redistribution when servers are added or removed from a system. It enables horizontal scaling by ensuring only a fraction of keys need to be remapped during server changes."
sources:
  - raw/articles/_done/consistent-hashing-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:52:26.637Z
---

# Consistent Hashing

> Consistent hashing is a distributed hashing technique that minimizes data redistribution when servers are added or removed from a system. It enables horizontal scaling by ensuring only a fraction of keys need to be remapped during server changes.

# Consistent Hashing

Consistent hashing is a distributed hashing technique that solves the rehashing problem in traditional hash-based server selection. Unlike the standard approach of `serverIndex = hash(key) % N`, consistent hashing minimizes data redistribution when servers are added or removed from a system.

## The Rehashing Problem

Traditional hashing methods cause significant issues when the server count changes:
- Removing a server causes most keys to be reassigned, leading to cache misses
- Adding a server results in unnecessary key redistributions
- This approach only works well when the server pool size is fixed

## How Consistent Hashing Works

### Hash Ring Structure
The hash space forms a continuous ring with values distributed from `0` to `2^160-1` (using hash functions like SHA-1). Servers are mapped onto this ring based on their IP addresses or names using the same hash function.

### Server Lookup
To find which server handles a key, traverse clockwise on the ring from the key's hash position until a server is found.

### Adding and Removing Servers
- **Adding a server**: Only keys between the new server and its predecessor need redistribution
- **Removing a server**: Only keys from the removed server are reassigned to the next server clockwise

## Virtual Nodes Solution

Basic consistent hashing can lead to uneven partition sizes and non-uniform key distribution. The solution is **virtual nodes**:
- Each physical server is represented by multiple virtual nodes distributed uniformly on the ring
- Virtual nodes improve key distribution and balance load
- As the number of virtual nodes increases, the standard deviation decreases, leading to more balanced data distribution

## Benefits

- **Minimized Redistribution**: Only a fraction of keys are reassigned during server changes
- **[[Horizontal Scaling]]**: Enables efficient scaling across multiple servers
- **Mitigates Hotspots**: Balances data distribution to avoid server overload

## Real-World Applications

Consistent hashing is widely used in distributed systems including Amazon DynamoDB, Apache Cassandra, Discord, Akamai CDN, and Maglev [[Load Balancer]].

---
*Related: [[Load Balancer]], [[Horizontal Scaling]], [[Database Sharding]], [[System Scaling]]*
