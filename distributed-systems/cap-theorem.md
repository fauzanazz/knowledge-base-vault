---
title: "CAP Theorem"
category: distributed-systems
summary: "The CAP theorem states that distributed systems can only guarantee two of three properties: Consistency, Availability, and Partition tolerance, with PACELC extending this to include latency considerations."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:32:14.867Z
---

# CAP Theorem

> The CAP theorem states that distributed systems can only guarantee two of three properties: Consistency, Availability, and Partition tolerance, with PACELC extending this to include latency considerations.

# CAP Theorem

The **CAP theorem** states that in the presence of network partitions, distributed systems must choose between **Consistency** and **Availability** - you cannot have all three: Consistency, Availability, and Partition tolerance.

## The Three Properties

**Consistency**: All nodes see the same data simultaneously
**Availability**: System remains operational and responsive
**Partition tolerance**: System continues operating despite network failures

## The Choice

When network partitions occur (which they inevitably will), systems must decide:

**Choose Consistency**: Decline reads/writes that can't reach the leader, sacrificing availability
**Choose Availability**: Allow clients to query followers, sacrificing strong consistency

## PACELC Theorem

The **PACELC theorem** extends CAP by considering normal operation:

**P** (Partition): Choose between **A** (Availability) and **C** (Consistency)
**E** (Else): Choose between **L** (Latency) and **C** (Consistency)

This recognizes that even without partitions, there's a trade-off between consistency and latency - stronger consistency requires more coordination, increasing response times.

## Practical Implications

**Not binary**: Consistency and availability exist on a spectrum, not as binary choices
**Rare partitions**: Network partitions are usually infrequent, but the latency-consistency trade-off is constant
**Design strategy**: Move coordination away from critical paths to improve performance while maintaining necessary consistency

Systems like [[Chain Replication]] demonstrate how delegating coordination to separate control planes can optimize this trade-off.

---
*Related: [[Consistency Models]], [[PACELC Theorem]], [[Network Partitions]]*
