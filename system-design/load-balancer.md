---
title: "Load Balancer"
category: system-design
summary: "A load balancer distributes incoming network traffic across multiple servers using various algorithms and health checks, operating at different network layers to provide scalability and fault tolerance."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/_done/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:24:57.510Z
---

# Load Balancer

> A load balancer distributes incoming network traffic across multiple servers using various algorithms and health checks, operating at different network layers to provide scalability and fault tolerance.

# Load Balancer

A load balancer distributes incoming network traffic across multiple servers to prevent any single server from becoming overwhelmed. It enables horizontal scaling of stateless applications and provides fault tolerance through health monitoring.

## Prerequisites and Benefits

Load balancing requires **stateless application servers** where state is pushed to dedicated services like databases and file stores. This enables:
- **Increased capacity**: Multiple servers handle more concurrent requests
- **Fault tolerance**: Failed servers are removed from the pool
- **Improved availability**: Theoretical availability = 1 - (product of server failure rates)

## Load Balancing Algorithms

**Round Robin**: Distributes requests sequentially across servers
**Consistent Hashing**: Maps requests to servers using hash functions
**Load-Based**: Considers server load, but can cause surprising behavior when servers report zero load
**Randomized Least Loaded**: Randomizes requests across least loaded servers for optimal results

## Service Discovery

Load balancers discover available servers through:
- **Static configuration**: Manual IP address lists (difficult to maintain)
- **Dynamic discovery**: Using coordination services like Zookeeper or etcd
- **Auto-scaling integration**: Automatic server addition/removal based on load

## Health Checks

**Passive**: Piggybacks on existing requests, detecting timeouts or 503 errors
**Active**: Dedicated `/health` endpoints polled by load balancers
- Can perform simple OK responses or sophisticated resource checks
- **Critical consideration**: Bugs in health endpoints can bring down entire applications

## Implementation Types

### DNS Load Balancing
Adds multiple server IPs as DNS records. **Limitation**: Lacks fault tolerance due to DNS caching delays.

### Network Load Balancing (L4)
Operates at TCP layer with Virtual IPs (VIPs):
- Very fast performance
- Supports TCP termination and direct server return
- Limited to TCP-level features

### Application Load Balancing (L7)
HTTP reverse proxies offering advanced features:
- HTTP connection multiplexing
- TLS termination
- Rate limiting and sticky sessions
- **Trade-off**: Lower throughput than L4 balancers

### Sidecar Proxy Pattern
L7 load balancer instances run on client machines, forming a service mesh. **Advantage**: Eliminates single point of failure. **Disadvantage**: Requires control plane management.

Load balancers are essential for [[System Scaling]] and work alongside [[Caching]] and [[Content Delivery Network]]s to handle massive traffic loads.

---
*Related: [[System Scaling]], [[Horizontal Scaling]], [[Caching]], [[Service Discovery]], [[Health Checks]]*
