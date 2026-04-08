---
title: "Continuous Deployment"
category: software-engineering
summary: "Continuous deployment automatically releases code changes to production once merged into the main branch. It reduces deployment risk through automation, staged rollouts, and automatic rollbacks while freeing developers from manual deployment tasks."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.397Z
---

# Continuous Deployment

> Continuous deployment automatically releases code changes to production once merged into the main branch. It reduces deployment risk through automation, staged rollouts, and automatic rollbacks while freeing developers from manual deployment tasks.

# Continuous Deployment

Continuous deployment (CD) automatically deploys code changes to production once they're merged into the main repository branch. Manual deployments are infrequent, risky, and waste developer time.

## Problems with Manual Deployment

- Infrequent releases lead to batched changes, increasing rollout risk
- Difficult to identify which specific change caused issues
- Developers must monitor dashboards and alerts during deployment
- Time-consuming process prevents developers from picking up new tasks

## CD Pipeline Stages

**Review and Build**
- Pull request review with automated linters and tests
- Team member approval using checklists covering tests, metrics, backward compatibility, and rollback safety
- Build stage creates release artifacts
- All changes (code, configuration, infrastructure) should be version controlled

**Pre-production**
- Deploy to non-production environments for end-to-end testing
- Multiple environments possible: simple smoke tests to production traffic mirroring
- Assess artifact health using same metrics as production

**Production**
- Initial release to small number of instances
- Gradual rollout to remaining instances if health checks pass
- Deploy to lower-traffic regions first, then sequential stages
- Ensure sufficient capacity during deployment

## Rollback Strategy

CD pipelines must verify artifact health at every step using:
- End-to-end test results
- Health metrics and error rates
- Upstream/downstream dependency health
- Configurable bake times between stages

Automatic rollback triggers when indicators fail, though operators may choose manual intervention or rolling forward with fixes for backward-incompatible changes.

---
*Related: [[Software Maintainability]], [[Infrastructure as Code]], [[System Monitoring]], [[Deployment Strategies]]*
