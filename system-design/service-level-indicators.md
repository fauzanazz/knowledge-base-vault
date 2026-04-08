---
title: "Service Level Indicators"
category: system-design
summary: "Service Level Indicators (SLIs) are metrics tied to user-perceived service quality, typically expressed as ratios of good events to total events. Common SLIs include response time, availability, and throughput."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T19:11:26.758Z
---

# Service Level Indicators

> Service Level Indicators (SLIs) are metrics tied to user-perceived service quality, typically expressed as ratios of good events to total events. Common SLIs include response time, availability, and throughput.

# Service Level Indicators

Service Level Indicators (SLIs) are metrics directly tied to the level of service provided to users. Unlike general system metrics, SLIs focus on user-perceived quality such as response time, error rate, and throughput.

## Characteristics

SLIs are typically:
- **Ratio-based** - "Good events" over total events (0 = broken, 1 = perfect)
- **Time-windowed** - Aggregated over rolling periods
- **User-focused** - Measured as close to user experience as possible

## Common SLIs

**Response Time** - Percentage of requests completing faster than a threshold. Measured using percentiles rather than averages since outliers significantly skew average calculations.

**Availability** - Ratio of successful requests to total requests over a time period. Indicates how often the service is usable.

**Throughput** - Rate of successful operations, indicating system capacity and performance.

## Measurement Considerations

**Location** - Measure as close to user experience as possible. If too costly, choose the next best measurement point (load balancer vs. service vs. client).

**Distribution Handling** - Response times follow long-tailed, right-skewed distributions. Percentiles (50th, 95th, 99th) better represent user experience than averages. The 99th percentile shows that 99% of requests complete faster than the specified time.

**Long-tail Impact** - Monitor high percentiles (99.9th) as they may represent important business users. Long-tail latencies can dramatically impact system capacity - if 1% of requests take 20x longer, thread pools can become exhausted.

## Business Impact

Studies show 100ms delay in load time hurts conversion rates by 7%. Reducing long-tail latencies typically improves average-case performance as well.

SLIs form the foundation for [[Service Level Objectives]] and alerting strategies in [[Monitoring]] systems.

---
*Related: [[Service Level Objectives]], [[Monitoring]], [[Performance Optimization]], [[User Experience]]*
