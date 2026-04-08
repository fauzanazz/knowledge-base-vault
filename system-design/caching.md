---
title: "Caching"
category: system-design
summary: "Caching is a high-speed storage layer that buffers frequently accessed data to improve performance and reduce load on origin data stores through strategic placement and cache management policies."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/_done/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:24:57.511Z
---

# Caching

> Caching is a high-speed storage layer that buffers frequently accessed data to improve performance and reduce load on origin data stores through strategic placement and cache management policies.

# Caching

Caching is a high-speed storage layer that buffers frequently accessed data to improve application performance and reduce load on origin data stores. It provides best-effort guarantees since it's not the source of truth - cache state can always be rebuilt from the origin.

## When Caching is Effective

Caching works best when a significant portion of requests target frequently accessed objects. The **hit ratio** (proportion of requests served from cache vs. origin) determines cost-effectiveness.

Hit ratio depends on:
- **Total cacheable objects**: Fewer objects = better hit ratio
- **Access probability**: Higher likelihood of repeated access = better performance
- **Cache size**: Larger caches = more objects stored = higher hit ratios

## Cache Placement Strategy

The higher in the call stack a cache is placed, the more requests it can capture. Multiple caching layers can be implemented:

1. **Browser cache**: [[HTTP Caching]] at the client level
2. **[[Content Delivery Network]]**: Geographic distribution of cached content
3. **Reverse proxy cache**: Server-side caching via [[Load Balancer]]s
4. **Application cache**: In-memory caches within application servers
5. **Database cache**: Buffer pools and query result caching

## Cache Management

**Eviction Policies**: Determine which items to remove when cache is full
- LRU (Least Recently Used)
- LFU (Least Frequently Used)
- TTL (Time To Live) based expiration

**Cache Invalidation**: Ensuring cache consistency with origin data
- Write-through: Update cache and origin simultaneously
- Write-behind: Update cache immediately, origin asynchronously
- Cache-aside: Application manages cache updates

## Important Considerations

**Caching is an optimization, not a scaling solution**. The origin data store must handle all requests without cache assistance. It's acceptable for requests to become slower without cache, but not for the system to crash.

**Cache warming**: Pre-populating caches with frequently accessed data to avoid cold start performance issues.

**Monitoring**: Track hit ratios, latency improvements, and cache effectiveness to optimize placement and sizing.

## Integration with Scaling

Caching works synergistically with other [[System Scaling]] techniques:
- Reduces database load, enabling higher throughput
- Complements [[Database Sharding]] by reducing cross-partition queries
- Works with [[Load Balancer]]s to distribute cached responses

Effective caching strategies can dramatically improve system performance while reducing infrastructure costs and complexity.

---
*Related: [[HTTP Caching]], [[Content Delivery Network]], [[System Scaling]], [[Load Balancer]], [[Database Sharding]]*
