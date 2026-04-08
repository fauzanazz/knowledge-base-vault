---
title: "HTTP Caching"
category: system-design
summary: "HTTP caching enables browsers and servers to store static resources locally to reduce network calls and improve performance. It uses Cache-Control headers and ETags to manage resource freshness and validation."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.237Z
---

# HTTP Caching

> HTTP caching enables browsers and servers to store static resources locally to reduce network calls and improve performance. It uses Cache-Control headers and ETags to manage resource freshness and validation.

# HTTP Caching

HTTP caching enables browsers and servers to store static resources locally to reduce network calls and improve performance. It uses Cache-Control headers and ETags to manage resource freshness and validation.

## How HTTP Caching Works

HTTP caching is limited to GET and HEAD methods. The server issues a `Cache-Control` header that tells browsers how to handle resources:

- **max-age**: Time-to-live (TTL) of the resource
- **ETag**: Version identifier for the resource
- **age**: How long the resource has been cached

If a subsequent request is made for a cached resource, it's served from cache as long as it's still fresh (TTL hasn't expired). However, if the resource changes on the server, clients won't get the latest version immediately, making reads not strongly consistent.

## Cache Validation

When a resource expires, the cache forwards a conditional request to the server asking if it has changed. If unchanged, the server updates the TTL without retransmitting the resource.

## Immutable Resources Strategy

A powerful technique treats static resources as immutable. When a resource changes, instead of updating it, a new file is published with a version tag and references are updated. This enables atomic changes to all static resources with the same version.

## CQRS Pattern

HTTP caching implements the Command-Query Responsibility Segregation (CQRS) pattern by treating read and write paths differently, since reads are far more common than writes.

## Reverse Proxies

[[Content Delivery Network]] services and reverse proxies like NGINX and HAProxy can cache static resources server-side, sharing the cache among multiple clients to reduce application server burden.

---
*Related: [[Content Delivery Network]], [[Caching]], [[System Scaling]], [[Load Balancer]]*
