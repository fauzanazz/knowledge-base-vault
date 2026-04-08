---
title: "Load Balancer"
category: system-design
summary: "A load balancer distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It provides redundancy, scalability, and improved system reliability."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.247Z
---

# Load Balancer

> A load balancer distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It provides redundancy, scalability, and improved system reliability.

# Load Balancer

A load balancer distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It provides redundancy, scalability, and improved system reliability.

## Types of Load Balancing

**DNS Load Balancing**: Simple approach adding multiple server IPs as DNS records. Clients pick one server, but lacks fault tolerance since DNS continues routing to failed servers. Mainly used for routing traffic across data centers.

**Network Load Balancing (L4)**: Operates at TCP layer with virtual IPs (VIPs) mapped to server pools. Supports TCP termination and direct server return optimization. Very fast but limited to TCP-level features. Examples: AWS Network Load Balancer, Azure Load Balancer.

**Application Load Balancing (L7)**: HTTP reverse proxies supporting advanced features like TLS termination, rate limiting, sticky sessions, and HTTP connection multiplexing. Lower throughput than L4 but much more feature-rich.

## Load Balancing Algorithms

- **Round Robin**: Simple rotation through servers
- **Consistent Hashing**: Deterministic server selection
- **Load-based**: Routes based on server load metrics
- **Randomized Least Loaded**: Randomizes among least loaded servers (often performs best)

## Service Discovery

Load balancers discover server pools through:
- Static configuration files (painful to maintain)
- Dynamic discovery using Zookeeper or etcd
- Auto-scaling integration in cloud providers

## Health Checks

**Passive**: Piggybacks on existing requests, detecting failures through timeouts or 503 errors.

**Active**: Dedicated `/health` endpoints polled by load balancers. Can perform simple OK responses or sophisticated resource checks.

## Availability Benefits

Theoretical availability = 1 - (product of server failure rates). Two 99% available servers = 99.99% availability. However, this assumes independent failures and doesn't account for detection delays or cascading failures.

## Sidecar Proxy Pattern

Service mesh approach where each application has a local L7 load balancer, delegating load balancing to clients and avoiding single points of failure.

---
*Related: [[System Scaling]], [[Content Delivery Network]], [[Horizontal Scaling]], [[Multi-Data Center Setup]]*
