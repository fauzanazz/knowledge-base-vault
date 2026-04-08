---
title: "DNS System"
category: networking
summary: "DNS is a distributed, hierarchical, eventually consistent key-value store that maps hostnames to IP addresses through a multi-level resolution process involving caches, resolvers, and authoritative name servers."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.259Z
---

# DNS System

> DNS is a distributed, hierarchical, eventually consistent key-value store that maps hostnames to IP addresses through a multi-level resolution process involving caches, resolvers, and authoritative name servers.

# DNS System

The Domain Name System (DNS) is a distributed, hierarchical, eventually consistent key-value store that translates human-readable hostnames into IP addresses required for network communication.

## Resolution Process

DNS resolution follows a hierarchical lookup process:

1. **Browser cache**: First checks local cache for hostname
2. **DNS resolver**: ISP-maintained server checks its cache
3. **Root name server**: Maps top-level domains (.com, .org) to appropriate name servers
4. **TLD name server**: Returns authoritative name server for the specific domain
5. **Authoritative name server**: Provides the final IP address mapping

## Caching and TTL

Extensive caching occurs at each level to avoid repeated round trips. Cache refresh is controlled by Time-To-Live (TTL) values:
- **High TTL**: Slower updates when domain changes occur
- **Low TTL**: Increased average request times

This creates a fundamental trade-off between update speed and performance.

## Security and Reliability

Modern DNS implementations use:
- **DNS over TLS**: Encrypted queries using TCP instead of plain-text UDP
- **Static stability**: Resolvers serve stale entries rather than errors when name servers are unavailable

Static stability allows systems to continue functioning even when dependencies fail, improving overall system resilience.

## Single Point of Failure

DNS name servers can become single points of failure leading to outages. Smart mitigation strategies include serving stale entries during name server downtime rather than returning errors.

The DNS system demonstrates how [[Distributed Systems]] can achieve global scale through hierarchical design and aggressive caching strategies.

---
*Related: [[Network Communication]], [[Distributed Systems]], [[Caching]], [[System Resiliency]], [[Static Stability]]*
