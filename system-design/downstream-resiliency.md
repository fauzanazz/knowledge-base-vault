---
title: "Downstream Resiliency"
category: system-design
summary: "Downstream resiliency patterns protect systems from failures in their dependencies through timeouts, retries, and circuit breakers."
sources:
  - raw/articles/resiliency-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:18:59.153Z
---

# Downstream Resiliency

> Downstream resiliency patterns protect systems from failures in their dependencies through timeouts, retries, and circuit breakers.

# Downstream Resiliency

Downstream resiliency focuses on preventing faults from propagating from dependencies to your service through tactical patterns that handle external service failures gracefully.

## Timeout

Always set timeouts when making external calls to prevent resource leaks. Without timeouts, requests may never return, effectively creating resource leaks.

**Configuration**: Set timeout to the 99.9th percentile of downstream service response time to achieve desired false timeout rate (e.g., 0.1%).

**Monitoring**: Track status codes, latency, and success/error rates at integration points, often managed by reverse proxy sidecars.

## Retry with Exponential Backoff

When requests timeout, retry with increasing delays to handle transient connectivity issues without overwhelming degraded services.

**Formula**: `delay = min(cap, initial_backoff * 2^attempt)`

**Jitter**: Add randomization to prevent retry storms:
`delay = random(0, min(cap, initial_backoff * 2^attempt))`

**Considerations**:
- Only retry transient errors, not consistent failures (e.g., authorization errors)
- Avoid retrying non-idempotent operations
- Be aware of retry amplification in service chains

## Retry Amplification

In service chains where each level retries, downstream services experience amplified load. For example, if 3 services each retry 3 times, the deepest service sees 27x the original load.

**Solution**: Retry at a single level and fail fast at all others.

## Circuit Breaker

Detects persistent service degradation and blocks requests until recovery, preventing slow downstream services from impacting callers.

**States**:
- **Closed**: All requests pass through
- **Open**: All requests blocked
- **Half-Open**: Occasional requests test for recovery

**Configuration Challenges**:
- How many failures trigger the open state?
- How long to wait before testing recovery?
- Requires historical data about failure patterns

Circuit breakers are most effective for non-transient errors where retries won't help, allowing subsystems to fail without slowing down callers.

---
*Related: [[System Resiliency]], [[API Idempotency]], [[Network Communication]], [[Load Balancer]]*
