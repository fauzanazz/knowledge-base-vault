---
title: "Content Delivery Network"
category: system-design
summary: "A Content Delivery Network (CDN) is a geographically distributed network of servers that cache content closer to users and optimize network routing to improve performance and reduce server load."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/_done/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:24:57.509Z
---

# Content Delivery Network

> A Content Delivery Network (CDN) is a geographically distributed network of servers that cache content closer to users and optimize network routing to improve performance and reduce server load.

# Content Delivery Network

A Content Delivery Network (CDN) is a geographically distributed overlay network of caches (reverse proxies) designed to work around internet protocol limitations and deliver content efficiently to users worldwide.

## Architecture and Benefits

When using a CDN, clients access URLs that resolve to CDN servers. If requested content exists in the cache, it's served immediately. Otherwise, the CDN transparently fetches it from the origin server.

The primary benefit isn't caching - it's the underlying **overlay network** architecture that exploits advanced routing techniques to reduce response time and increase bandwidth.

## Overlay Network Optimization

The internet's BGP routing protocol prioritizes hop count over latency or congestion. CDNs overcome this by:

- **Geographic proximity**: DNS load balancing routes clients to the closest servers based on IP geolocation
- **Internet exchange points**: CDN servers are placed at network nodes where ISPs intersect
- **Advanced routing**: Algorithms consider real-time latency and congestion data
- **TCP optimizations**: Connection pooling on critical paths avoids setup overhead

## Multi-Layer Caching

CDNs implement hierarchical caching:

1. **Edge servers**: Deployed across geographical regions for maximum proximity
2. **Intermediary layers**: Fewer regional servers that edge servers query before hitting origin
3. **Partitioned storage**: Content is distributed across multiple servers with internal routing

This creates a trade-off: more edge servers reach more clients but reduce cache hit ratios since content must be cached in more locations.

## Dynamic Content Support

Beyond static caching, CDNs can transport dynamic resources efficiently using their optimized network infrastructure. This makes the CDN an effective application frontend that shields origin servers from DDoS attacks.

## Popular Providers

Well-known CDN providers include Amazon CloudFront and Akamai, which offer global infrastructure and advanced optimization features.

CDNs are essential for applications requiring global reach, forming a critical component of modern [[System Scaling]] strategies alongside [[HTTP Caching]] and [[Load Balancer]]s.

---
*Related: [[HTTP Caching]], [[Load Balancer]], [[System Scaling]], [[Caching]]*
