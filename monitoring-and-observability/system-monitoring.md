---
title: "System Monitoring"
category: system-design
summary: "System monitoring detects production issues quickly and provides health information through dashboards. It combines black-box monitoring for external validation with white-box monitoring for internal system instrumentation."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.399Z
---

# System Monitoring

> System monitoring detects production issues quickly and provides health information through dashboards. It combines black-box monitoring for external validation with white-box monitoring for internal system instrumentation.

# System Monitoring

System monitoring detects production issues quickly and alerts human operators responsible for the system. It also provides high-level health information through dashboards.

## Monitoring Approaches

**Black-box Monitoring** - Reports only if service is up/down from external perspective. Useful for:
- Tracking health of external dependencies
- Validating impact to users
- Catching issues not visible to the application (e.g., DNS failures)

**White-box Monitoring** - Instruments code to track specific feature health. Useful for identifying root causes of failures.

## Synthetic Testing

Synthetics are periodic scripts that send requests to test endpoints, monitoring latency and success rates. Benefits include:
- Catching connectivity issues invisible to applications
- Exercising endpoints not frequently used by real users
- Providing consistent baseline measurements

## Core Metrics

Minimum metrics every service should track:
- **Request throughput** - Volume of incoming requests
- **Internal state** - Memory usage, cache hit rates, queue lengths
- **Dependencies** - Availability and performance of external services

## Metrics Implementation

Metrics are time series of measurements with timestamps and optional labels (key-value pairs). Labels enable filtering by region, environment, or node but increase storage complexity.

**Event-based Reporting Process:**
1. Service reports events to local telemetry agent
2. Agent batches and sends to remote telemetry service
3. Events pre-aggregated into time buckets (1m, 5m, 1h)
4. Pre-aggregation reduces storage costs but loses granular re-aggregation ability

Deliberate instrumentation effort by developers is required to capture meaningful metrics that provide insight into system behavior and user experience.

---
*Related: [[Service Level Indicators]], [[Observability]], [[Alerting]], [[Dashboards]]*
