---
title: "Caching"
category: system-design
summary: "Caching is a technique that stores frequently accessed data in fast memory storage to reduce database load and improve response times. It serves as a temporary data store layer that is much faster than traditional databases."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:49:29.762Z
---

# Caching

> Caching is a technique that stores frequently accessed data in fast memory storage to reduce database load and improve response times. It serves as a temporary data store layer that is much faster than traditional databases.

# Caching

Caching is a technique that stores frequently accessed data in fast memory storage to reduce database load and improve response times. It serves as a temporary data store layer that is much faster than traditional databases.

## Cache Policies

When there's a cache miss, the missing object must be requested from the origin. There are two main policies:

**Side Cache (Write-Through-Aside)**: The application requests the object from the origin and stores it in the cache. The cache is treated as a key-value store.

**Inline Cache (Write-Through)**: The cache communicates with the origin directly, requesting the missing object on behalf of the application. The app only accesses the cache.

**Write-Back Cache**: Acts as a write-through cache but asynchronously updates the data store, adding complexity but improving performance.

## Eviction and Expiration

When cache capacity is limited, entries must be evicted. The most common eviction policy is **LRU (Least Recently Used)** - least-recently used elements are evicted first.

The **expiration policy** defines how long objects are stored before being refreshed from the origin (TTL). Higher TTL increases hit ratio but also increases the chance of serving stale objects.

Cache invalidation - automatically expiring objects when they change - is hard to implement in practice, which is why TTL is used as a workaround.

## Local Cache

The simplest implementation uses a library (like Guava in Java or RocksDB) for in-memory caching embedded within the application.

**Advantages**: Simple to implement

**Disadvantages**:
- Different replicas have different caches, wasting memory
- Cannot be partitioned or replicated
- Consistency issues - separate clients can see different versions
- More application replicas mean more downstream traffic
- Vulnerable to "thundering herd" when caches are empty

## External Cache

External caches are dedicated services (like [[Redis]] or [[Memcached]]) that provide shared caching functionality.

**Advantages**:
- Shared among clients for consistency
- Can scale via replication and partitioning
- Request load doesn't grow with application instances
- Consistent hashing helps minimize data movement during scaling

**Disadvantages**:
- Higher maintenance cost
- Higher latency due to network calls
- Single point of failure requiring backup strategies

Applications can maintain an in-process cache as backup when external cache fails.

---
*Related: [[Load Balancer]], [[Database Replication]], [[Content Delivery Network]], [[HTTP Caching]]*
