---
title: "Consistency Models"
category: distributed-systems
summary: "Consistency models define trade-offs between data consistency and system performance in distributed systems, ranging from strong consistency to eventual consistency."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.873Z
---

# Consistency Models

> Consistency models define trade-offs between data consistency and system performance in distributed systems, ranging from strong consistency to eventual consistency.

# Consistency Models

Consistency models help define the trade-off between consistency and performance in [[Distributed Systems]]. They determine what guarantees clients have when reading data from replicated systems.

## Strong Consistency

All reads and writes go through the leader, guaranteeing that all observers see writes after they're persisted. This provides the strongest guarantees but limits throughput.

**Caveat**: Leaders can't instantly confirm writes - they must first confirm they're still the leader by sending requests to all followers, adding latency to reads.

## Sequential Consistency

Followers can serve reads while clients are attached to particular followers. This leads to different clients seeing different states, but operations always occur in the same order.

**Example**: Producer/consumer system synchronized with a queue where producers and consumers see events in the same order but at different times.

**Trade-off**: Higher throughput than strong consistency, but clients must handle follower failures by waiting or switching followers.

## Eventual Consistency

Clients can use any follower for reads. The only guarantee is that clients will eventually see the latest state.

**Challenge**: If follower B lags behind follower A, a client querying A then B will receive an earlier version of the state.

**Use cases**: Applications where perfect consistency isn't critical, like view counts on videos or social media likes.

## CAP and PACELC Theorems

**CAP Theorem**: During network partitions, choose between availability and consistency.

**PACELC Theorem**: Even without partitions, there's a trade-off between latency and consistency - stronger consistency means higher latency.

## Practical Considerations

Some databases allow configurable consistency levels (Amazon DynamoDB, Cassandra) while others provide counter-intuitive guarantees for better performance. The choice depends on application requirements and acceptable trade-offs.

---
*Related: [[Database Replication]], [[CAP Theorem]], [[Distributed Systems]], [[System Scaling]]*
