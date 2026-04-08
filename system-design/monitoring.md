---
title: "Monitoring"
category: system-design
summary: "Monitoring detects production issues quickly and provides system health visibility through metrics, alerts, and dashboards. It combines black-box external monitoring with white-box internal instrumentation."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T19:11:26.758Z
---

# Monitoring

> Monitoring detects production issues quickly and provides system health visibility through metrics, alerts, and dashboards. It combines black-box external monitoring with white-box internal instrumentation.

# Monitoring

Monitoring detects production issues quickly and alerts human operators responsible for system health. It also provides high-level health information through dashboards.

## Monitoring Types

**Black-box Monitoring** - Reports external service availability without internal knowledge. Useful for detecting symptoms and validating user impact through synthetic tests that send periodic requests to endpoints.

**White-box Monitoring** - Instruments internal code to track specific feature health. Helps identify root causes of failures and provides detailed system insights.

## Metrics

Metrics are time series of measurements for resources (CPU load) or behaviors (request count). Each sample contains a floating-point number and timestamp, optionally tagged with labels for filtering.

Minimum service metrics:
- Request throughput
- Internal state (e.g., cache status)
- Dependencies availability and performance

Metrics flow through event-based reporting where telemetry agents batch events and send them to remote services. Pre-aggregation into time buckets (1m, 5m, 1h) reduces storage costs but loses granular re-aggregation capability.

## [[Service Level Indicators]]

SLIs are metrics tied to user-perceived service quality:
- **Response Time** - Requests faster than threshold
- **Availability** - Successful requests / total requests

Measure as close to user experience as possible. Use percentiles rather than averages for response time since outliers skew averages significantly.

## [[Service Level Objectives]]

SLOs define acceptable SLI ranges indicating healthy service. They establish error budgets (tolerable failure percentage) and guide prioritization decisions. Teams should prioritize repairs over features when error budgets are exhausted.

## Alerts and Dashboards

Alerts trigger actions when metrics cross thresholds. Effective alerts are actionable and based on SLOs rather than resource metrics. Use burn rate (error budget consumption speed) for better precision and recall.

Dashboards should be audience-specific, version-controlled, and follow best practices like consistent time ranges and minimal chart complexity.

---
*Related: [[Service Level Indicators]], [[Service Level Objectives]], [[Software Maintainability]], [[Observability]]*
