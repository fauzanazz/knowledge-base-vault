---
title: "Redundancy"
category: system-design
summary: "Redundancy is the replication of functionality or state across multiple nodes to provide fault tolerance and improved availability in distributed systems."
sources:
  - raw/articles/resiliency-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:18:59.152Z
---

# Redundancy

> Redundancy is the replication of functionality or state across multiple nodes to provide fault tolerance and improved availability in distributed systems.

# Redundancy

Redundancy is the replication of functionality or state across multiple nodes, serving as the first line of defense against failures in distributed systems. When functionality is replicated, other nodes can take over during failures.

## Prerequisites for Effective Redundancy

- Complexity introduced must not cost more availability than it adds
- System must reliably detect healthy vs unhealthy components
- System must run in degraded mode when some replicas fail
- System must recover to fully redundant mode

## Geographic Distribution

**Availability Zones (AZs)**: Cloud providers replicate infrastructure across multiple data centers within regions. AZs are:
- Far enough apart to minimize correlated failures
- Close enough for low latency and synchronous replication
- Cross-connected with high-speed network links

**Multi-Region**: For catastrophic disaster protection, entire application stacks can be duplicated across regions using:
- Global DNS [[Load Balancer]] for traffic distribution
- Asynchronous replication due to high inter-region latency
- Higher costs and complexity

## Correlation Challenges

Redundancy only helps when failures are not correlated. Examples:
- Memory corruption on one server is unlikely on another
- Data center outages affect all servers unless replicated across AZs
- Software bugs affect all replicas running the same code

## Implementation with Load Balancers

[[Load Balancer]] systems implement redundancy by:
- Maintaining pools of redundant nodes
- Using health checks to detect faulty nodes
- Ensuring remaining replicas have capacity for increased load
- Automatically adding recovered servers back to the pool

Redundancy enables distributed applications to achieve better availability than single-node applications, but requires careful design to avoid introducing more complexity than benefit.

---
*Related: [[System Resiliency]], [[Load Balancer]], [[Multi-Data Center Setup]], [[Fault Isolation]]*
