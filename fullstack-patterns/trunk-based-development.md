---
title: "Trunk-Based Development"
category: fullstack-patterns
summary: "A source control practice where all developers integrate small, frequent changes directly into a single shared branch, eliminating long-lived feature branches and enabling true continuous integration."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Trunk-Based Development

> A source control practice where all developers integrate small, frequent changes directly into a single shared branch, eliminating long-lived feature branches and enabling true continuous integration.

## What It Is

Trunk-based development (TBD) means all developers commit to `main` (the "trunk") at least once per day. Feature branches exist for hours, not days. This is the foundation of high-performing engineering teams per DORA research — the practice most correlated with deployment frequency and lead time.

**Not this** (Gitflow anti-pattern):
```
main ←──────────────────────────── merge after 3 weeks
feature/big-feature ──────────────────────────────────→
```

**This** (TBD):
```
main ←── commit ←── commit ←── commit ←── commit
           ↑ every dev, every day, <24h branches
```

## Short-Lived Feature Branches

Branches should be **deleted within 1–2 days**. Rules of thumb:
- Branch off `main`, PR back to `main`
- No branch lives longer than 24 hours (strict TBD) or 2 days (practical TBD)
- Branch names: `feat/add-login-button`, not `feat/redesign-entire-auth-system`

**Why this matters**: branches older than 2 days start creating merge conflicts. Older than 1 week = integration tax that compounds exponentially.

## CI Gates on Every PR

Every commit to a PR branch must trigger CI before merge:

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm run type-check
      - run: pnpm run lint
      - run: pnpm run test:unit
      - run: pnpm run test:integration
      - run: pnpm run build

  # Branch protection: require CI green before merge
```

**Merge queue** (GitHub): serialize PRs to prevent "passed CI individually but breaks when merged together." Critical for high-velocity teams.

## Feature Flags Are Required

With TBD, incomplete features must hide behind feature flags. This is non-negotiable:

```typescript
// Day 1: New payment feature committed to main, flag OFF
export async function checkout(cart: Cart, userId: string) {
  if (await featureFlags.isEnabled('stripe-v2', { userId })) {
    return checkoutV2(cart);   // incomplete, safe because flag is OFF
  }
  return checkoutV1(cart);     // stable path
}

// Day 5: Feature complete, flag enabled for 5% canary
// Day 7: Flag enabled for 100%, old code path deleted in next PR
```

Never merge broken code and rely on "the flag will protect us" — the code must be functionally correct; the flag gates the user-visible behavior.

## Branch Protection Rules

```json
// GitHub branch protection (required)
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci/test", "ci/lint", "ci/build"]
  },
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null,
  "enforce_admins": true,
  "require_linear_history": true   // no merge commits → clean git log
}
```

**Linear history** (`--rebase` or squash merge): keeps `git log` readable and `git bisect` usable.

## Continuous Deployment Integration

TBD + CI = true CD. Every merge to main deploys automatically:

```
Push to main
  → CI runs (unit, integration, type-check) [~5 min]
  → Build Docker image, push to registry
  → Deploy to staging automatically
  → Run E2E smoke tests
  → Deploy to production (canary 5%)
  → Automated metric analysis
  → Promote to 100%
```

Total time from merge to production: **15–30 minutes** in mature setups.

## Handling Large Refactors

Large changes that can't ship in one PR:

1. **Abstract branch pattern**: introduce abstraction (interface/adapter), ship it, then migrate implementations incrementally
2. **Parallel change**: add new implementation alongside old, migrate callers, delete old
3. **Branch by abstraction**: create abstraction layer, run both implementations, remove old

```typescript
// Step 1: Add abstraction (ships to prod, no behavior change)
interface EmailService { send(to: string, subject: string, body: string): Promise<void>; }

// Step 2: New implementation behind it (ships, tested)
class SendGridEmailService implements EmailService { ... }

// Step 3: Swap injection in DI container (1-line change, easy rollback)
// Step 4: Delete old implementation
```

## DORA Metrics Correlation

DORA (DevOps Research and Assessment) 2023 findings:
- Elite teams deploy **multiple times per day** → requires TBD
- Elite teams have lead time **< 1 hour** → requires TBD + CD
- Gitflow teams average **1–4 deploys per month**

TBD is the highest-leverage practice for improving deployment frequency.

## Trade-offs

| Pro | Con |
|-----|-----|
| Eliminates merge conflicts | Requires feature flags discipline |
| Forces smaller, safer changes | Harder for junior devs initially |
| Enables true CI/CD | Requires strong CI gates |
| Fast feedback loops | Incomplete features need careful management |
| Clean git history | Culture shift from Gitflow teams |

## When to Use

✅ Teams deploying > 1x per week  
✅ Teams with mature CI/CD pipelines  
✅ Teams with feature flag infrastructure  
✅ High-performing product teams (DORA elite tier)  
❌ Open-source projects with external contributors (PR review latency makes TBD impractical)  
❌ Teams without CI automation (merging to main without gates is dangerous)

---
*Related: [[Feature Flags]], [[Monorepo Architecture]], [[Canary Deployments]]*
