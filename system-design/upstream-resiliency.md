---
title: "Upstream Resiliency"
category: system-design
summary: "Upstream resiliency patterns protect services from being overwhelmed by client requests through load shedding, rate limiting, and constant work principles."
sources:
  - raw/articles/resiliency-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:18:59.153Z
---

# Upstream Resiliency

> Upstream resiliency patterns protect services from being overwhelmed by client requests through load shedding, rate limiting, and constant work principles.

# Upstream Resiliency

Upstream resiliency focuses on protecting your service from being overwhelmed by client requests through various defensive patterns.

## Load Shedding

Reject excess requests once server capacity is reached to maintain performance for ongoing requests.

**Implementation**: Use a counter tracking concurrent requests. When it exceeds a threshold, reject new requests with HTTP 503 (Service Unavailable).

**Enhancements**: Differentiate between high and low priority requests, only rejecting low-priority ones.

**Limitations**: Rejecting requests still consumes resources (TCP connection, request parsing), so extreme load can still cause degradation.

## Load Leveling

Use [[Message Queue]] systems to let services process requests at their own pace when clients don't need immediate responses.

**Benefits**: Handles short-lived load spikes effectively
**Limitations**: Creates backlog during sustained load increases

## Rate Limiting

Reject requests once quotas are exceeded, typically applied per user, API key, or IP address.

**Multiple Quotas**: Track both requests per second and bytes per second
**Error Handling**: Return HTTP 429 (Too Many Requests) with `Retry-After` header

**Use Cases**:
- Prevent well-intentioned clients from overwhelming services
- Shield against client bugs causing excessive load
- Implement pricing tiers for platform products

### Implementation Approaches

**Single Process**: Use sliding window with time buckets to track requests efficiently without storing individual request timestamps.

**Distributed**: Move logic to external key-value store using atomic operations like `getAndIncrement`. Batch updates and fallback to in-memory during outages.

## Constant Work Pattern

Perform the same amount of work under high load as under normal load to create predictable, **antifragile** systems that perform better under stress.

**Example**: Instead of broadcasting individual configuration changes, periodically dump complete configuration to file store. Data planes bulk-load entire configuration regardless of change volume.

**Benefits**:
- Reliable and predictable performance
- Self-healing properties
- Simpler implementation than event-sourcing

**Trade-offs**: More expensive than doing only necessary work, but provides increased reliability and reduced complexity.

Upstream resiliency patterns are often combined with autoscaling to handle sustained load increases while protecting against temporary spikes.

---
*Related: [[System Resiliency]], [[Message Queue]], [[Load Balancer]], [[API Idempotency]]*
