---
title: "Service Level Objectives"
category: system-design
summary: "Service Level Objectives (SLOs) define acceptable ranges for SLIs that indicate healthy service operation. They enable error budgets, prioritization decisions, and form the basis for Service Level Agreements with users."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.400Z
---

# Service Level Objectives

> Service Level Objectives (SLOs) define acceptable ranges for SLIs that indicate healthy service operation. They enable error budgets, prioritization decisions, and form the basis for Service Level Agreements with users.

# Service Level Objectives

Service Level Objectives (SLOs) define the range of acceptable values for [[Service Level Indicators]] that indicate a service is operating healthily.

## SLO Structure

Example SLO: "99% of API calls to endpoint X should complete below 200ms over a rolling window of 1 week."

This creates an **error budget** - the 1% of requests allowed to exceed 200ms latency. Error budgets represent tolerable failure levels.

## Error Budget Applications

**Team Prioritization:**
- Repair items prioritized over new features when error budget is exhausted
- Incident importance measured by error budget burn rate

**Time Window Trade-offs:**
- Small windows force quick action and bug fix prioritization
- Longer windows better suited for strategic project investment decisions

## Setting SLO Strictness

**Too Lenient** - Won't detect user-facing issues
**Too Strict** - Wastes engineering time on micro-optimizations with diminishing returns

Note: 100% reliability doesn't guarantee 100% user experience due to factors like last-mile connectivity.

## Best Practices

**Start Comfortable** - Begin with achievable ranges and adjust over time
**Avoid Over-Engineering** - Anything above 99.9% (3 nines) is very costly with diminishing returns
**Keep Simple** - Maintain as few SLOs as possible while providing good service health indication
**Stakeholder Agreement** - SLOs must be agreed upon with stakeholders for prioritization decisions

**Google's SRE Principle:** "If you can't win a conversation about priorities by quoting a particular SLO, it's probably not worth having that SLO."

## Controlled Failures

Users may become over-reliant on actual service behavior rather than documented SLAs. Consider introducing controlled failures to ensure dependencies can cope with target SLA levels.

---
*Related: [[Service Level Indicators]], [[Error Budgets]], [[System Monitoring]], [[Alerting]], [[Incident Management]]*
