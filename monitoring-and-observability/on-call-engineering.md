---
title: "On-Call Engineering"
category: system-design
summary: "On-call engineering involves developers taking responsibility for monitoring and responding to production issues in their services. Healthy on-call practices require actionable alerts, proper compensation, and focus on mitigation over immediate fixes."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:15:09.675Z
---

# On-Call Engineering

> On-call engineering involves developers taking responsibility for monitoring and responding to production issues in their services. Healthy on-call practices require actionable alerts, proper compensation, and focus on mitigation over immediate fixes.

# On-Call Engineering

On-call engineering is the practice where developers take responsibility for monitoring and responding to production issues in the services they build. This approach incentivizes building systems with [[Software Maintainability]] and operability in mind.

## Healthy On-Call Practices

Healthy on-call requires several key elements:

- **Developer ownership**: Engineers who built the service should be on-call since they know it most intimately
- **Actionable alerts**: All alerts must link to relevant dashboards and runbooks
- **Proper compensation**: Being on-call is stressful and should be compensated accordingly
- **No feature work expectation**: On-call engineers shouldn't be expected to progress on feature development

## Incident Response Process

During outages, the priority is mitigation first, not fixing. The process follows:

1. **Mitigate** the immediate impact
2. **Communicate** all actions in a global outages channel for handoff visibility
3. **Understand root cause** during normal working hours
4. **Prevent recurrence** through repair items

When incidents burn significant error budget, postmortems should identify root causes and create repair items. For severe incidents, teams should pause feature work to focus solely on reliability improvements.

The SRE books provide comprehensive guidance for creating healthy on-call rotations and incident response processes.

---
*Related: [[System Monitoring]], [[Alerting]], [[Service Level Objectives]], [[Software Maintainability]]*
