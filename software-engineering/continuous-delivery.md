---
title: "Continuous Delivery"
category: software-engineering
summary: "Continuous delivery automates the deployment pipeline from code merge to production release, reducing manual effort and deployment risks through automated testing, staged rollouts, and rollback capabilities."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T19:11:26.757Z
---

# Continuous Delivery

> Continuous delivery automates the deployment pipeline from code merge to production release, reducing manual effort and deployment risks through automated testing, staged rollouts, and rollback capabilities.

# Continuous Delivery

Continuous delivery (CD) automates the deployment of changes from repository merge to production release. Manual deployments are infrequent, risky, and waste developer time on deployment monitoring rather than feature development.

## Pipeline Stages

**Review and Build** - Pull requests undergo automated validation through linters and tests, plus human review checking for:
- Appropriate test coverage
- Metrics, logs, and traces inclusion
- Backwards compatibility
- Safe rollback capability

Configuration changes, infrastructure-as-code, and static assets should follow the same CD pipeline as application code.

**Pre-production** - Changes deploy to non-production environments for end-to-end testing. Multiple environments may exist, from simple smoke tests to production traffic mirroring for realistic validation.

**Production** - Gradual rollout starting with small instance counts, then expanding. Multi-region deployments should start with lower-traffic regions and proceed sequentially to minimize risk.

## Rollback Strategy

CD pipelines must verify artifact health at each stage using:
- End-to-end test results
- Health metrics and error rates
- Upstream/downstream dependency health
- Configurable bake times between stages

Automatic rollbacks trigger when indicators fail. For backwards-incompatible changes, break them into smaller compatible changes:
1. **Prepare** - Support both old and new formats
2. **Activate** - Switch to new format
3. **Cleanup** - Remove old format support

## Infrastructure as Code

Infrastructure should be defined as code using tools like Terraform, enabling version control and review processes for infrastructure changes. This prevents configuration-related outages through proper change management.

---
*Related: [[Software Maintainability]], [[Software Testing]], [[Monitoring]], [[Infrastructure as Code]]*
