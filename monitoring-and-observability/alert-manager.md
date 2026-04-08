---
title: "Alert Manager"
category: system-design
summary: "An alert manager processes monitoring rules, generates alerts when thresholds are breached, and manages alert delivery through various notification channels. It handles deduplication, filtering, and retry logic for reliable alerting."
sources:
  - raw/articles/metrics-monitoring-and-alerting-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:39:01.483Z
---

# Alert Manager

> An alert manager processes monitoring rules, generates alerts when thresholds are breached, and manages alert delivery through various notification channels. It handles deduplication, filtering, and retry logic for reliable alerting.

# Alert Manager

An alert manager is a system component that evaluates monitoring rules against metrics data and generates notifications when predefined conditions are met. It serves as the bridge between monitoring data and human operators.

## Core Functions

**Rule Evaluation**: Periodically queries metrics data against configured alert rules. Rules are typically defined in YAML format:

```yaml
- name: instance_down
  rules:
  - alert: instance_down
    expr: up == 0
    for: 5m
    labels:
      severity: page
```

**Alert Processing**: When rules trigger, the manager creates alert events with metadata including severity, labels, and timestamps.

## Alert Management Features

**Deduplication**: Prevents multiple alerts for the same condition by tracking alert state and suppressing duplicates.

**Filtering**: Routes alerts based on labels, severity, or other criteria to appropriate notification channels.

**Grouping**: Combines related alerts to reduce notification noise during widespread incidents.

**Silencing**: Temporarily suppresses alerts during maintenance windows or known issues.

## Delivery Mechanisms

Alert managers support multiple notification channels:
- Email
- SMS/Phone calls
- PagerDuty integration
- Slack/Teams webhooks
- Custom HTTP endpoints

## Reliability Features

**Retry Logic**: Ensures alert delivery through configurable retry policies with exponential backoff.

**State Persistence**: Stores alert state in databases like Cassandra to survive system restarts.

**At-Least-Once Delivery**: Uses [[Message Queue]] systems like Kafka to guarantee alert delivery.

## Access Control

Alert managers implement role-based access control to restrict alert configuration and management operations to authorized personnel only.

Modern implementations often integrate with existing tools like Prometheus AlertManager or PagerDuty rather than building custom solutions due to the complexity of reliable alerting systems.

---
*Related: [[Metrics Monitoring and Alerting System]], [[Message Queue]], [[Notification System]], [[System Monitoring]]*
