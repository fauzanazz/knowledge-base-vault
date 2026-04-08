---
title: "Canary Deployments"
category: fullstack-patterns
summary: "A progressive rollout strategy that exposes new versions to a small subset of traffic first, using automated metric analysis to promote or roll back without human intervention."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Canary Deployments

> A progressive rollout strategy that exposes new versions to a small subset of traffic first, using automated metric analysis to promote or roll back without human intervention.

## What It Is

Named after "canary in a coal mine," canary deployments gradually shift traffic from the stable version (baseline) to the new version (canary). Metrics are compared between the two populations in real time. If canary metrics degrade, automated rollback triggers before the majority of users are affected.

```
Step 1:  95% → v1 (stable)   |  5% → v2 (canary)
Step 2:  75% → v1            | 25% → v2  [metrics OK]
Step 3:  50% → v1            | 50% → v2  [metrics OK]
Step 4:   0% → v1            |100% → v2  [promoted]
```

## Core Implementation: Argo Rollouts

Argo Rollouts is the Kubernetes standard for progressive delivery:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-api
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }
        - analysis:
            templates:
              - templateName: error-rate
        - setWeight: 25
        - pause: { duration: 10m }
        - setWeight: 50
        - pause: { duration: 10m }
      canaryService: my-api-canary
      stableService: my-api-stable
      trafficRouting:
        nginx:
          stableIngress: my-api-ingress
```

## Automated Analysis (Kayenta / Argo Analysis)

The key differentiator from manual canary is **automated metric promotion**:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate
spec:
  metrics:
    - name: error-rate
      interval: 1m
      successCondition: result[0] < 0.01   # <1% error rate
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{status=~"5..",job="{{args.service-name}}"}[5m]))
            /
            sum(rate(http_requests_total{job="{{args.service-name}}"}[5m]))
    - name: p99-latency
      successCondition: result[0] < 0.5    # <500ms p99
      provider:
        prometheus:
          query: |
            histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

**Flagger** (from Flux ecosystem) offers similar capabilities with simpler YAML and native integration with Istio, Linkerd, and NGINX.

## Traffic Splitting Strategies

### Kubernetes Service Mesh (Istio)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
spec:
  http:
    - route:
        - destination:
            host: my-api
            subset: stable
          weight: 95
        - destination:
            host: my-api
            subset: canary
          weight: 5
```

### Header-Based Canary
Route specific users (internal testers, beta users) to canary regardless of percentage:
```yaml
- match:
    - headers:
        x-canary:
          exact: "true"
  route:
    - destination:
        host: my-api-canary
```

### AWS CloudFront + Lambda@Edge
```javascript
// Lambda@Edge: route 5% to canary origin
exports.handler = (event, context, callback) => {
  const request = event.Records[0].cf.request;
  if (Math.random() < 0.05) {
    request.origin.custom.domainName = 'canary.api.example.com';
  }
  callback(null, request);
};
```

## Metrics to Monitor During Canary

**Error signals** (fast feedback):
- HTTP 5xx rate vs. baseline
- Exception rate in APM (Datadog, Sentry)
- Timeout rate

**Latency signals**:
- p50, p95, p99 response times (compare canary vs. stable cohort)

**Business signals** (slower, but critical):
- Conversion rate, add-to-cart, checkout completion
- Revenue per request

**Infrastructure signals**:
- CPU, memory, DB connection pool exhaustion

## Canary vs. Blue-Green

| Dimension | Canary | Blue-Green |
|-----------|--------|------------|
| Rollout speed | Gradual (minutes to hours) | Instant flip |
| Real user validation | Yes, on subset | Smoke tests only |
| Rollback time | Seconds (weight to 0) | Seconds (pointer swap) |
| Infrastructure cost | Minimal overhead | 2x during deploy |
| Database migrations | Same expand-contract | Expand-contract required |
| Complexity | Higher | Moderate |

## Automated Rollback

```bash
# Argo Rollouts manual abort
kubectl argo rollouts abort my-api

# Auto-rollback triggers when AnalysisRun fails
# Rollout automatically sets weight back to 0
# Previous stable ReplicaSet scales back up
```

Alert routing: failed canary analysis should page on-call and create an incident. Never silently roll back.

## When to Use

✅ High-traffic services where you want real-user validation  
✅ APIs with measurable success metrics (latency, error rate)  
✅ Kubernetes environments with Argo Rollouts or Flagger  
✅ Changes to critical paths (payment, auth, checkout)  
❌ Low-traffic services where 5% = near-zero users (not statistically meaningful)  
❌ Major schema changes (still need expand-contract pattern)  
❌ Teams without Prometheus/metrics infrastructure (no automated analysis possible)

---
*Related: [[Blue-Green Deployments]], [[Observability Stack]], [[GitOps]]*
