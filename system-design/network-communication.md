---
title: "Network Communication"
category: system-design
summary: "Network communication in distributed systems involves protocols arranged in layers, from hardware interfaces to application-level protocols. Understanding these layers is crucial for building reliable distributed systems."
sources:
  - raw/articles/introduction-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.257Z
---

# Network Communication

> Network communication in distributed systems involves protocols arranged in layers, from hardware interfaces to application-level protocols. Understanding these layers is crucial for building reliable distributed systems.

# Network Communication

Network communication in [[Distributed Systems]] requires processes to agree on protocols that define how data is transmitted and processed. These protocols are arranged in a hierarchical stack where each layer builds upon the abstraction provided by the layer below.

## Protocol Stack

**Link Layer**: Provides an interface for operating on underlying network hardware via protocols like Ethernet.

**Internet Layer**: Enables sending data between machines with best-effort delivery through the IP protocol. Data can be lost, corrupted, or arrive out of order.

**Transport Layer**: Enables data transmission between processes on different machines. TCP is the most important protocol at this layer, adding reliability to IP.

**Application Layer**: High-level protocols targeted by applications, including HTTP, DNS, and custom protocols.

## Addressing and Routing

Communication requires two key mechanisms:
- **Addressing**: Handled by the IP protocol to identify destination nodes
- **Routing**: Managed by routers using routing tables that map destination addresses to next hops along the path

The Border Gateway Protocol (BGP) handles building and communicating routing tables across the internet infrastructure.

## Protocol Selection

While TCP provides reliability, it comes with latency and bandwidth overhead. UDP serves as an alternative for building custom protocols that need only specific TCP features. Games often use UDP-based protocols since retransmitting missed frames is unnecessary when game state has already progressed.

Understanding these communication fundamentals is essential for designing robust [[System Resiliency]] and optimizing performance in distributed architectures.

---
*Related: [[TCP Protocol]], [[UDP Protocol]], [[IP Protocol]], [[Network Protocols]], [[Distributed Systems]]*
