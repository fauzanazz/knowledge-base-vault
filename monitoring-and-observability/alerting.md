---
title: "Alerting"
category: system-design
summary: "Alerting systems trigger actions when metrics cross thresholds, either through automated scripts or paging on-call engineers. Effective alerts are actionable and based on user-facing SLOs rather than internal metrics."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.400Z
---

# Alerting

> Alerting systems trigger actions when metrics cross thresholds, either through automated scripts or paging on-call engineers. Effective alerts are actionable and based on user-facing SLOs rather than internal metrics.

# Alerting

Alerting is the component of a monitoring system that triggers actions when metrics cross defined thresholds.

## Alert Actions

Depending on severity and type, alerts can:
- Trigger automated scripts (e.g., service restart)
- Page the on-call engineer for the impacted service
- Send notifications to team channels

## Actionable Alerts

For alerts to be useful, they must be **actionable** - operators should quickly assess impact and urgency.

**Poor Alert:** CPU usage above 80%
**Good Alert:** [[Service Level Objectives]] error budget 50% consumed

SLO-based alerts directly indicate user impact and provide clear context for response.

## Precision vs Recall Trade-off

**Precision** - Fraction of significant events over total alerts (How many alerts indicate real incidents?)
**Recall** - Fraction of significant events that triggered alerts (Are you missing significant events?)

Low precision creates noisy, non-actionable alerts. Low recall leads to undetected outages. Improving one typically reduces the other.

## Burn Rate Alerting

For SLO-based alerting, use **burn rate** instead of simple threshold crossing:

**Burn Rate** = (% of error budget consumed) / (% of elapsed SLO time window)

- Burn rate = 1: Error budget exhausted in full time window (30 days)
- Burn rate = 2: Error budget exhausted in half time window (15 days)
- Burn rate = 10: Error budget exhausted in 1/10th time window (3 days)

**Example Alert Configuration:**
- Low severity: Burn rate ≥ 2
- High severity: Burn rate ≥ 10

## Alert Categories

**Primary:** Most alerts should be configured based on [[Service Level Objectives]]
**Secondary:** Alerts on known failure modes (e.g., memory leaks) with automatic mitigation

This approach focuses attention on user-facing issues while handling known operational problems automatically.

---
*Related: [[Service Level Objectives]], [[System Monitoring]], [[Incident Management]], [[On-Call Engineering]]*
