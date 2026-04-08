---
title: "System Resiliency"
category: system-design
summary: "System resiliency is the ability of a distributed system to continue operating despite failures through fault tolerance, redundancy, and recovery mechanisms."
sources:
  - raw/articles/introduction-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
  - raw/articles/resiliency-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:18:59.152Z
---

# System Resiliency

> System resiliency is the ability of a distributed system to continue operating despite failures through fault tolerance, redundancy, and recovery mechanisms.

# System Resiliency

System resiliency is the ability of a distributed system to continue operating despite failures. As systems scale and add more components, the probability of failures increases, making resiliency critical for maintaining availability.

## Common Failure Causes

**Hardware Faults**: Physical components like HDDs, SSDs, NICs, and CPUs can fail. Entire data centers can go down due to power cuts or natural disasters.

**Incorrect Error Handling**: A 2014 study found that most catastrophic failures in distributed data stores were due to poor error handling - ignoring errors, catching overly generic exceptions, or incomplete handlers with TODO comments.

**Configuration Changes**: Among the leading causes of major failures. Misconfiguration or rarely-used valid configurations can cause delayed effects, making root cause analysis difficult.

**Single Points of Failure (SPOF)**: Components whose failure brings down the entire system, such as manual processes requiring human intervention or DNS dependencies.

**Network Faults**: Slow network calls are silent killers - clients don't know if responses will arrive, leading to timeouts and performance degradation.

**Resource Leaks**: Memory leaks, thread pool exhaustion, and socket pool depletion cause steady performance degradation over time.

**Load Pressure**: Systems have capacity limits. Sudden load increases from seasonality, expensive requests, or DDoS attacks can cause failures.

**Cascading Failures**: When component dependencies cause failures to propagate across services, potentially bringing down entire systems.

## Building Resilient Systems

Resilient systems must detect, react to, and repair failures. Key strategies include:

- **[[Redundancy]]**: Replicating functionality across multiple nodes
- **[[Fault Isolation]]**: Partitioning systems to limit blast radius
- **[[Load Balancer]]**: Distributing traffic and detecting unhealthy nodes
- **Timeout and Retry**: Preventing resource leaks and handling transient failures
- **Circuit Breaker**: Stopping requests to persistently failing services
- **Rate Limiting**: Protecting against excessive load

Effective resiliency requires measuring the probability and impact of faults, then prioritizing the most likely and impactful ones for mitigation.

---
*Related: [[Redundancy]], [[Fault Isolation]], [[Load Balancer]], [[Distributed Systems]], [[System Scaling]]*
