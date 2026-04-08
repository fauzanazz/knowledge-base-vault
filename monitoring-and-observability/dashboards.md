---
title: "Dashboards"
category: system-design
summary: "Dashboards present real-time system health through metrics visualization. Effective dashboards are audience-specific, version-controlled, and follow best practices for chart organization and annotation."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.401Z
---

# Dashboards

> Dashboards present real-time system health through metrics visualization. Effective dashboards are audience-specific, version-controlled, and follow best practices for chart organization and annotation.

# Dashboards

Dashboards present real-time system health through metrics visualization. They can become dumping grounds for unused charts if not designed thoughtfully.

## Dashboard Design Principle

Always consider the **audience first** and work backwards:

**SLO Dashboard** - For organizational stakeholders to gain quick system health insight and assess user impact during incidents

**Public API Dashboard** - Shows health of public endpoints, used by operators to trace error paths during outages

**Service Dashboard** - Service-specific implementation details for developers with system context. Should include:
- Service-specific metrics
- Upstream dependency health
- Downstream dependent health
- Infrastructure components (message queues, load balancers)

## Best Practices

**Version Control** - Define dashboards using domain-specific languages and version control like code. Enables synchronization across environments and updates via pull requests.

**Layout Priority** - Most important charts at the top since operators see them first

**Standardization:**
- Same time resolution across all charts (1m, 5m, 1h)
- Same time range (24h, 7d, 1y)
- Default timezone for operator communication
- Choose resolution based on use case: 1h/1m for incidents, 1y/1d for capacity planning

**Chart Design:**
- Minimize metrics and data points per chart for speed and clarity
- Group metrics with similar ranges; split widely different ranges into separate charts
- Example: 10th, average, 90th percentile in one chart; 99.9th and 0.1th percentile in another

**Annotations:**
- Descriptions and links to runbooks, related dashboards, escalation contacts
- Horizontal lines for alert thresholds
- Vertical lines for deployments

**Metric Continuity** - Always emit metrics, even when values are zero. Gaps in charts make it unclear whether there were no errors or metrics stopped being emitted.

---
*Related: [[System Monitoring]], [[Service Level Indicators]], [[Observability]], [[Incident Management]]*
