---
title: "Unique ID Generator"
category: system-design
summary: "A unique ID generator creates globally unique, sortable identifiers in distributed systems. The Twitter Snowflake approach is the most effective solution for high-throughput requirements."
sources:
  - raw/articles/unique-id-generator-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:49:54.843Z
---

# Unique ID Generator

> A unique ID generator creates globally unique, sortable identifiers in distributed systems. The Twitter Snowflake approach is the most effective solution for high-throughput requirements.

# Unique ID Generator

A unique ID generator creates globally unique identifiers in distributed systems where traditional auto-increment keys are unsuitable due to scalability and synchronization challenges.

## Requirements

- IDs must be **unique** and **numerical**
- IDs must fit within **64 bits**
- IDs should be **sortable by date**
- System must generate **over 10,000 IDs per second**
- IDs increment with time but not strictly by +1

## Design Approaches

### Multi-Master Replication
Uses database auto_increment with step increments across multiple servers. However, this approach has scaling issues and IDs don't always increase with time.

### UUID (Universally Unique Identifier)
Generates 128-bit unique identifiers independently on each server. While requiring no coordination, UUIDs exceed the 64-bit requirement and aren't sortable by time.

### Ticket Server
Uses a centralized database server to increment and assign IDs. Simple for small-scale systems but creates a single point of failure.

### Twitter Snowflake Approach
The most effective solution divides 64-bit IDs into sections:
- **Sign Bit (1 bit):** Always 0
- **Timestamp (41 bits):** Milliseconds since custom epoch, ensures time-ordering
- **Datacenter ID (5 bits):** Supports up to 32 datacenters
- **Machine ID (5 bits):** Supports up to 32 machines per datacenter
- **Sequence Number (12 bits):** Up to 4,096 IDs per millisecond per machine

## Key Considerations

### Clock Synchronization
Use **Network Time Protocol (NTP)** to minimize clock drift across servers, as ID generation assumes synchronized clocks.

### High Availability
ID generators are mission-critical and require redundancy and failover mechanisms to prevent system failures.

### Section Length Tuning
Adjust bit allocation based on specific use cases - fewer sequence bits for more timestamp bits or vice versa.

The Snowflake approach provides scalability, decentralization, and time-ordered IDs while meeting the 64-bit constraint and high-throughput requirements.

---
*Related: [[Database Sharding]], [[Microservices Architecture]], [[System Scaling]], [[URL Shortener]]*
