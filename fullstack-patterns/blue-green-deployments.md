---
title: "Blue-Green Deployments"
category: fullstack-patterns
summary: "A zero-downtime deployment strategy that maintains two identical production environments and switches traffic instantly between them, enabling instant rollback."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Blue-Green Deployments

> A zero-downtime deployment strategy that maintains two identical production environments and switches traffic instantly between them, enabling instant rollback.

## What It Is

Blue-green deployment runs two identical production environments ("blue" and "green"). At any point, only one is live (receiving traffic). New releases deploy to the idle environment; after validation, traffic switches over atomically. Rollback = switch back.

```
Users → Load Balancer → [BLUE]  (live v1.2)
                      → [GREEN] (idle, being deployed v1.3)

After validation:
Users → Load Balancer → [BLUE]  (idle, old v1.2)
                      → [GREEN] (live v1.3)
```

## Infrastructure Patterns

### AWS Pattern
```yaml
# Route 53 weighted routing or ALB target group swap
Resources:
  BlueTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
  GreenTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
  
  # Switch: update ALB listener default action
  ListenerRule:
    Type: AWS::ElasticLoadBalancingV2::ListenerRule
    Properties:
      Actions:
        - Type: forward
          TargetGroupArn: !Ref GreenTargetGroup  # was Blue
```

### Kubernetes Pattern (Argo Rollouts)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    blueGreen:
      activeService: my-app-active
      previewService: my-app-preview
      autoPromotionEnabled: false   # manual promotion gate
      prePromotionAnalysis:
        templates:
          - templateName: success-rate
        args:
          - name: service-name
            value: my-app-preview
```

### Vercel / Netlify Pattern
Vercel's "Instant Rollback" is blue-green under the hood: every deploy gets a unique URL; promoting to production is just a DNS/CDN pointer swap.

## Database Migrations — The Hard Problem

Database changes are the biggest challenge. You cannot run DDL that breaks the old schema while blue is still live.

**Expand-Contract Pattern (required for blue-green)**:

```sql
-- Phase 1: EXPAND (deploy with both old + new code)
ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;
-- Old app ignores new column. New app writes to it.

-- Phase 2: MIGRATE (backfill data, run alongside)
UPDATE users SET email_verified = TRUE WHERE verified_at IS NOT NULL;

-- Phase 3: CONTRACT (after old version fully gone)
ALTER TABLE users DROP COLUMN verified_at;
```

Never do breaking schema changes (rename/drop column) in a single migration during blue-green. Always expand first, then contract in a later release.

**Backward-compatible migration checklist**:
- ✅ ADD column with default  
- ✅ ADD nullable column  
- ✅ ADD new table  
- ✅ ADD index concurrently (Postgres)  
- ❌ DROP column (breaks old app reading it)  
- ❌ RENAME column  
- ❌ Change column type (unless fully compatible)  
- ❌ ADD NOT NULL without default

## Traffic Switching Strategies

**Hard cutover**: 100% flip. Simple, fast, but no gradual validation.

**Weighted switch**: Gradually shift 10% → 50% → 100% (this becomes a canary deployment hybrid).

**Session-sticky switch**: Existing sessions stay on blue; new sessions go to green. Avoids in-flight request interruption.

```nginx
# Nginx upstream swap via dynamic reconfiguration
upstream app {
    server green-app:8080 weight=100;
    server blue-app:8080 weight=0 backup;
}
```

## Pre-Promotion Validation

Before cutting traffic to green:
1. **Smoke tests**: automated HTTP checks against preview URL
2. **Load test**: validate green handles production traffic volume
3. **Security scan**: DAST against preview environment
4. **Manual sign-off**: product/QA approval gate

```bash
# GitHub Actions promotion gate
- name: Run smoke tests against green
  run: npx playwright test --base-url=$GREEN_URL

- name: Promote to production
  if: ${{ needs.smoke-tests.result == 'success' }}
  run: aws elbv2 modify-listener --listener-arn $ALB_ARN \
       --default-actions Type=forward,TargetGroupArn=$GREEN_TG_ARN
```

## Cost Implications

Blue-green doubles infrastructure costs during deployment. Strategies to reduce:
- **Scaled-down idle env**: run idle at 20% capacity; scale up before promotion
- **Spot/preemptible for idle**: use cheap instances for standby
- **Serverless**: FaaS blue-green is essentially free (Lambda aliases, Cloud Run revisions)

## Trade-offs

| Pro | Con |
|-----|-----|
| Instant rollback (seconds) | 2x infrastructure cost |
| Zero downtime deployments | Database migrations are complex |
| Full environment validation before go-live | Stateful services (websockets, cache) need re-warming |
| Clean separation of versions | Long-lived sessions may break on switch |

## When to Use

✅ Applications where downtime has direct revenue impact  
✅ Regulated environments requiring rollback capability  
✅ Stateless API services (easiest case)  
✅ Kubernetes clusters with Argo Rollouts  
❌ Stateful services with complex session management (prefer canary)  
❌ Resource-constrained environments where 2x cost is prohibitive  
❌ Databases themselves (use canary or rolling for DB upgrades)

---
*Related: [[Canary Deployments]], [[GitOps]], [[Feature Flags]]*
