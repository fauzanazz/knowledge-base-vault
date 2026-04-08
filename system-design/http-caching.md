---
title: "HTTP Caching"
category: system-design
summary: "HTTP caching enables browsers and servers to store frequently accessed resources locally, reducing network calls and improving performance through cache control headers and validation mechanisms."
sources:
  - raw/articles/_done/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:24:57.509Z
---

# HTTP Caching

> HTTP caching enables browsers and servers to store frequently accessed resources locally, reducing network calls and improving performance through cache control headers and validation mechanisms.

# HTTP Caching

HTTP caching enables browsers and servers to store frequently accessed resources locally, reducing network calls and improving performance. It's limited to GET and HEAD HTTP methods and treats static and dynamic resources differently.

## Cache Control Mechanism

Servers issue `Cache-Control` headers that instruct browsers how to handle resources:
- **max-age**: Time-to-live (TTL) of the resource
- **ETag**: Version identifier for the resource
- **age**: How long the resource has been cached

Resources are served from cache as long as they remain "fresh" (TTL hasn't expired). However, this means reads are not strongly consistent - clients may not immediately receive updated versions.

## Cache Validation

When a cached resource expires, the cache forwards a conditional request to the server asking if the resource has changed. If unchanged, the server updates the TTL without retransmitting the full resource.

## Immutable Resources Strategy

A powerful technique treats static resources as immutable. When resources change, new files are published with version tags rather than updating existing files. This enables atomic updates of all static resources with the same version.

## Command-Query Responsibility Segregation (CQRS)

HTTP caching implements the CQRS pattern by treating read and write paths differently, optimizing for the fact that reads are far more common than writes.

## Reverse Proxies

[[Load Balancer]]s can implement server-side caching through reverse proxies that intercept client calls and cache static resources. This shared cache reduces application server burden.

Reverse proxies provide additional benefits:
- Request authentication
- Response compression
- Rate limiting for DDoS protection
- Load balancing across multiple servers

Popular implementations include NGINX and HAProxy, though [[Content Delivery Network]]s have largely commoditized static resource caching.

HTTP caching is fundamental to web performance optimization and forms the foundation for more sophisticated [[Caching]] strategies in distributed systems.

---
*Related: [[Caching]], [[Content Delivery Network]], [[Load Balancer]], [[System Scaling]]*
