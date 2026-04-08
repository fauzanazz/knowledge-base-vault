---
title: "Consistency Models"
category: distributed-systems
summary: "Consistency models define trade-offs between data consistency and system performance in distributed systems, ranging from strong to eventual consistency."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.012Z
---

# Consistency Models

> Consistency models define trade-offs between data consistency and system performance in distributed systems, ranging from strong to eventual consistency.

# Consistency Models

**Consistency models** define the trade-offs between data consistency and system performance in replicated distributed systems.

## Strong Consistency

**Strong consistency** ensures all observers see writes after they're persisted. This requires routing all reads and writes through the leader, guaranteeing the latest state but limiting throughput.

Leaders must confirm they're still the leader before responding to reads, adding latency to serve requests.

## Sequential Consistency

**Sequential consistency** allows followers to serve reads while attaching clients to specific followers. Different clients may see different states, but operations always occur in the same order.

Example: Producer/consumer systems synchronized with queues where all participants see events in the same order but at different times.

## Eventual Consistency

**Eventual consistency** allows clients to use any follower, maximizing availability. Clients may receive stale data if followers lag behind, but they're guaranteed to eventually see the latest state.

This model works well for applications like view counters where slight inaccuracies are acceptable.

## CAP Theorem

The **CAP theorem** states that during network partitions, systems must choose between:
- **Consistency**: Guarantee strong consistency by declining reads that can't reach the leader
- **Availability**: Remain available by allowing follower queries, sacrificing strong consistency

## PACELC Theorem

**PACELC** extends CAP: "In case of Partition, choose between Availability and Consistency; Else, choose between Latency and Consistency."

This highlights that even without partitions, there's a fundamental trade-off between consistency strength and system latency.

---
*Related: [[Database Replication]], [[CAP Theorem]], [[Distributed Systems]]*
