---
title: "Feature Flags"
category: fullstack-patterns
summary: "Runtime toggles that decouple code deployment from feature release, enabling progressive delivery, A/B testing, and safe trunk-based development."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Feature Flags

> Runtime toggles that decouple code deployment from feature release, enabling progressive delivery, A/B testing, and safe trunk-based development.

## What It Is

Feature flags (feature toggles) are conditional branches in code controlled by external configuration rather than code changes. They allow deploying code to production in a disabled state, then enabling it for specific users, percentages, or environments without a new deployment.

## Flag Types

| Type | TTL | Use Case |
|------|-----|----------|
| **Release flags** | Days–weeks | Hide incomplete features during development |
| **Experiment flags** | Days–weeks | A/B tests with metrics |
| **Ops flags** | Hours–days | Kill switches for risky new code paths |
| **Permission flags** | Months–years | Beta programs, paid tiers, internal users |

**Kill switches** are the most operationally critical: circuit breakers that disable a feature instantly without a deploy.

## LaunchDarkly in Production

LaunchDarkly is the industry standard for enterprise feature management:

```typescript
import { init } from '@launchdarkly/node-server-sdk';

const client = init(process.env.LD_SDK_KEY);
await client.waitForInitialization();

// Evaluate flag with user context
const showNewCheckout = await client.variation(
  'new-checkout-flow',
  { key: user.id, email: user.email, plan: user.plan },
  false // default value
);
```

**Targeting rules** allow: % rollouts, user attribute matching, segment membership, date/time windows.

**Streaming SDKs** push flag updates in <200ms globally — no polling lag.

## Open-Source Alternatives

- **Unleash**: Self-hosted, strong GDPR story, PostgreSQL-backed, REST API + SDKs
- **Flagsmith**: Open-source + cloud, supports remote config (not just booleans)
- **GrowthBook**: A/B testing focused, integrates with analytics warehouses
- **OpenFeature**: CNCF standard SDK abstraction — swap providers without code changes

```typescript
// OpenFeature vendor-agnostic pattern
import { OpenFeature } from '@openfeature/server-sdk';
OpenFeature.setProvider(new LaunchDarklyProvider(sdkKey));

const client = OpenFeature.getClient();
const enabled = await client.getBooleanValue('new-feature', false, { targetingKey: userId });
```

## Progressive Delivery Pattern

```
1. Deploy code with flag OFF (dark launch)
2. Enable for 1% → monitor error rates, latency, business metrics
3. Expand to 10% → 50% → 100% based on metric thresholds
4. Flag cleanup: remove code branch once fully rolled out
```

**Metrics gates** (automated promotion): integrate with Prometheus/Datadog to auto-promote or roll back based on error rate, p99 latency, conversion rate.

## A/B Testing Integration

```typescript
// Track experiment exposure
analytics.track('Experiment Viewed', {
  experimentId: 'new-checkout-flow',
  variationId: showNewCheckout ? 'treatment' : 'control',
  userId: user.id
});

// Consistent assignment: same user always gets same variant
// LaunchDarkly uses MurmurHash on user.key + flag key
```

**Statistical significance**: run experiments for minimum detectable effect; avoid peeking. Tools like GrowthBook handle sequential testing properly.

## Trunk-Based Development Integration

Feature flags are the enabler of trunk-based development at scale:

```typescript
// Feature under development — merged to main but hidden
export async function processPayment(order: Order) {
  if (await flags.isEnabled('stripe-v2', { userId: order.userId })) {
    return stripeV2.charge(order);  // new code path
  }
  return stripeV1.charge(order);    // existing code path
}
```

Incomplete features live in `main` behind a flag — no long-lived branches, no merge conflicts.

## Flag Hygiene (Technical Debt)

Flags rot. Enforce TTLs:
- Add `// TODO: cleanup flag 'xyz' by 2026-06-01` comments
- Linting rules that flag stale constants (e.g., custom ESLint rule checking age)
- LaunchDarkly stale flag notifications; Unleash has flag archival
- Track flag count per service — over 50 active flags = smell

## Trade-offs

| Pro | Con |
|-----|-----|
| Deploy without releasing | Code complexity from branches |
| Instant rollback (no deploy needed) | Testing matrix explosion |
| Gradual exposure reduces blast radius | Flag debt if not cleaned up |
| Separate deploy from business decisions | Latency overhead (mitigated by local caching) |
| Enables trunk-based development | Wrong defaults = production incidents |

## When to Use

✅ Any feature with user-visible risk  
✅ Trunk-based development teams  
✅ A/B experiments on conversion funnels  
✅ Ops kill switches for downstream dependencies  
✅ Multi-tenant SaaS with per-customer features  
❌ Pure infrastructure changes (use canary deployments instead)  
❌ One-off config that never changes (use env vars)

---
*Related: [[Trunk-Based Development]], [[Canary Deployments]], [[Blue-Green Deployments]]*
