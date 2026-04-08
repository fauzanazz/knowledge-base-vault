---
title: "Development Process and Crisis Management"
tags: [development-process, git-workflows, ci-cd, tech-debt, crisis-management, postmortem, five-whys, trunk-based-development]
category: cto-leadership
sources:
  - "Comparing Git Workflows - Atlassian"
  - "A Successful Git Branching Model - Vincent Driessen (2010, reflection 2020)"
  - "Trunk Based Development - Atlassian"
  - "Why You Should Love Technical Debt - Medium"
  - "Five Whys - Eric Ries"
  - "Testing in Production the Safe Way - Cindy Sridharan"
  - "Write Better Error Messages - Jenni Nadler (Wix)"
last_updated: 2026-04-08
---

# Development Process and Crisis Management

## Scrum/Agile from the CTO Perspective

### When Scrum Works
- **Small teams (5-9)** building a single product
- **Unclear requirements** that need iterative discovery
- **Stakeholders who need regular visibility** into progress
- **New teams** that need structure to build habits

### When Scrum Breaks Down
- **Multiple teams** working on a shared codebase (coordination overhead)
- **Mature products** with well-understood requirements
- **Platform/infrastructure work** that doesn't fit sprint cycles
- **When velocity becomes a management metric** instead of a team tool

### CTO Decision Framework

| Team Stage | Recommended Process |
|------------|-------------------|
| 1-5 engineers | Kanban + weekly sync |
| 5-15 engineers | Light Scrum or Shape Up |
| 15-50 engineers | RFC-driven + quarterly planning |
| 50+ engineers | Custom (see Big Tech approaches) |

**Key principle:** Process should **decrease** as team maturity increases. If you're adding more ceremonies over time, something is wrong.

## Git Workflows Compared

### Centralized Workflow
- Single `main` branch, everyone commits directly
- **Use when:** Solo dev or very small team, simple project
- **Problem:** No isolation, conflicts on every push

### Feature Branch Workflow
- One branch per feature, merge via PR
- **Use when:** Most teams, most of the time
- **The standard** for modern development

### Gitflow (Vincent Driessen, 2010)

```
main ──────────────────────────────────────
  \                                    /
   release/1.0 ─────────────────────
    \                              /
     develop ──────────────────────
      \        \        /       /
       feature/a  feature/b
```

**Branches:**
- `main` — production-ready, tagged releases only
- `develop` — integration branch for next release
- `feature/*` — individual feature work
- `release/*` — release preparation (bug fixes only)
- `hotfix/*` — emergency production fixes

**⚠️ Driessen's 2020 reflection:** "If your team is doing continuous delivery, adopt a much simpler workflow (like GitHub flow) instead of trying to shoehorn git-flow."

**Use Gitflow when:**
- Software with explicit release versions (mobile apps, SDKs, self-hosted)
- Multiple versions maintained in parallel
- Release candidates need a stabilization period

**Don't use when:**
- Web apps with continuous deployment
- Single version in production

### GitHub Flow
- `main` is always deployable
- Branch from `main`, PR back to `main`, deploy from `main`
- Simplest workflow that supports collaboration

### Trunk-Based Development

```
main ═══════════════════════════════════
  ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑
  │   │   │   │   │   │   │   │
  small commits merged frequently
  (short-lived branches < 1 day)
```

**Core principles:**
1. **One branch: `main`** (trunk)
2. Developers commit to trunk **at least daily**
3. Short-lived feature branches (< 1 day, max 2 days)
4. **Feature flags** instead of feature branches for incomplete work
5. **No long-lived branches** — they are merge debt

**Why it works:**
- Eliminates merge hell
- Forces small, reviewable changes
- Enables true continuous integration
- Google, Meta, and most Big Tech use this

**Required infrastructure:**
- Fast CI pipeline (< 10 min)
- Feature flag system (LaunchDarkly, Unleash, etc.)
- Strong automated testing (high confidence in trunk)
- Code review culture (every commit reviewed quickly)

### Decision Matrix

| Factor | Gitflow | GitHub Flow | Trunk-Based |
|--------|---------|-------------|-------------|
| Deploy frequency | Weekly/monthly | Daily | Multiple/day |
| Team size | Any | Small-medium | Any (with discipline) |
| Release model | Versioned | Continuous | Continuous |
| Complexity | High | Low | Medium |
| Requires feature flags | No | Optional | Yes |
| CI/CD maturity needed | Low | Medium | High |

## CI/CD Principles

### Fundamental Principles of CI

1. **Maintain a single source repository** — one canonical truth
2. **Automate the build** — one command to build everything
3. **Make your build self-testing** — tests run automatically on every commit
4. **Everyone commits to mainline every day** — avoid integration debt
5. **Every commit triggers a build** — immediate feedback
6. **Fix broken builds immediately** — broken trunk blocks everyone
7. **Keep the build fast** — target < 10 minutes for core pipeline
8. **Test in a clone of production** — environment parity
9. **Make it easy to get the latest deliverables** — any team member, any time
10. **Everyone can see what's happening** — visibility builds trust

### CI/CD Pipeline Architecture

```
Commit → Build → Unit Tests → Integration Tests → Staging → Canary → Production
         ↓         ↓              ↓                ↓         ↓
       < 2min    < 5min        < 15min          < 30min   gradual rollout
```

### Common Anti-Patterns

- **"CI" that only runs on PRs** — that's not CI, that's gated builds
- **Long-running branches with "CI"** — CI means integrating to trunk
- **Tests that pass locally but fail in CI** — environment parity issue
- **Manual deployment steps** — if a human can forget it, automate it

## Tech Debt Philosophy

### Tech Debt as Financial Debt

Ward Cunningham's original metaphor: tech debt is like financial debt. Taking on debt isn't bad — it's a **strategic tool**:

| Financial Debt | Technical Debt |
|---------------|----------------|
| Mortgage to buy a house | Quick hack to hit a deadline |
| Monthly payments | Ongoing maintenance burden |
| Default → bankruptcy | Accumulated debt → feature paralysis |
| Refinancing | Refactoring |

### The Tech Debt Quadrant (Martin Fowler)

|  | Deliberate | Inadvertent |
|--|-----------|-------------|
| **Prudent** | "We know this is a shortcut, we'll fix it in Q2" | "Now we know how this should have been built" |
| **Reckless** | "We don't have time for design" | "What's a design pattern?" |

**Only prudent debt is acceptable.** Reckless debt is just bad engineering.

### Managing Tech Debt

**Don't:**
- Create a "tech debt sprint" (it'll get cancelled)
- Track tech debt as a separate backlog (it'll be deprioritized)
- Ask permission to fix tech debt (you'll be told no)

**Do:**
- **Budget 20-30% of capacity for maintenance** — non-negotiable
- **Boy Scout Rule** — leave code better than you found it
- **Tie debt to business impact** — "This tech debt causes 2 hours of manual work per deploy"
- **Make debt visible** — ADR (Architecture Decision Record) when taking on intentional debt
- **Sunset explicitly** — set a date when the debt must be repaid

## Crisis Management

### Five Whys (Eric Ries)

Root cause analysis technique from Toyota Production System, adapted for startups:

**Process:**
1. State the problem: "Production went down for 2 hours"
2. Ask "Why?" → "The database ran out of connections"
3. Ask "Why?" → "Connection pooling wasn't configured"
4. Ask "Why?" → "The default settings were used"
5. Ask "Why?" → "Our setup docs don't include connection pool configuration"
6. Ask "Why?" → "We don't have a production readiness checklist"

**Root cause:** Missing production readiness checklist → Fix: Create and enforce one.

**Rules:**
- **Stop at systemic causes**, not human blame
- Each "why" should lead to a **proportional investment** in the fix
- The **most senior person** in the room should be the last to speak
- **Document and share** — the learning is the value, not the blame

### Writing Postmortems

**Structure:**
```markdown
## Incident Summary
- What happened, when, impact (users affected, duration, revenue)

## Timeline
- Chronological events with timestamps

## Root Cause
- Five Whys analysis

## What Went Well
- Detection, response, communication

## What Went Poorly
- Gaps in monitoring, slow response, communication failures

## Action Items
- [ ] Preventive: ensure this can't happen again
- [ ] Detective: ensure we catch it faster next time
- [ ] Mitigative: ensure impact is smaller next time
```

**Key principle:** Postmortems are **blameless**. If your postmortem identifies "human error" as root cause, you haven't gone deep enough. Humans make errors — systems should prevent those errors from causing outages.

### Testing in Production

Cindy Sridharan's framework for safe production testing:

**Techniques (safest to riskiest):**

| Technique | Risk | Use Case |
|-----------|------|----------|
| **Feature flags** | Very low | Gradual rollout, A/B testing |
| **Canary deployments** | Low | New code on small % of traffic |
| **Shadow traffic** | Low | Compare new system output without serving |
| **Blue-green deployment** | Low | Instant rollback capability |
| **Chaos engineering** | Medium | Resilience testing (Netflix Chaos Monkey) |
| **Load testing in prod** | Medium-High | Capacity verification |

**Prerequisites for safe production testing:**
1. **Observability** — you must see what's happening in real-time
2. **Fast rollback** — < 5 minutes to previous version
3. **Blast radius control** — limit exposure (percentage, region, user segment)
4. **Kill switches** — instant feature disable without deploy

## Error Message Design

Wix's "Errorgate 2021" — they audited 7,643 error messages and found most were useless.

### The Three Components of a Good Error Message

1. **What happened** — state the problem in user terms
2. **Why it happened** — give context (if known)
3. **What to do next** — actionable next step

**Bad:** `Error 500: Internal Server Error`
**Good:** `We couldn't save your changes because the server is temporarily unavailable. Your draft has been saved locally — try again in a few minutes.`

### Error Message Principles

| Principle | Bad Example | Good Example |
|-----------|------------|--------------|
| **Be specific** | "Something went wrong" | "Your payment was declined by your bank" |
| **Be human** | "Error: CONN_REFUSED" | "We can't reach the server right now" |
| **Offer a next step** | "Failed" | "Failed to upload. Try a smaller file (max 10MB)" |
| **Don't blame the user** | "Invalid input" | "Phone numbers need 10 digits — yours has 9" |
| **Use progressive disclosure** | Full stack trace | "See details" expandable |

### Error Handling Architecture

```
User action → Validation (client) → Validation (server) → Business logic → External service
     ↓              ↓                    ↓                    ↓                ↓
  Inline hint   Form error          API error             Domain error    Retry/fallback
  (preventive)  (immediate)         (structured)          (meaningful)    (resilient)
```

**Each layer should add context**, not replace it. A 500 from Stripe should become "Your payment couldn't be processed" at the user layer, not "Internal Server Error."

## Key Takeaways

1. **Process maturity should increase with team maturity** — start structured, become lighter
2. **Trunk-based development is the gold standard** for CI/CD, but requires infrastructure investment
3. **Gitflow is for versioned software** — if you deploy continuously, use GitHub Flow or trunk-based
4. **Tech debt is a strategic tool** — budget for it, don't ask permission
5. **Postmortems are blameless** — "human error" is never root cause
6. **Test in production safely** — feature flags + canary + observability
7. **Error messages are UX** — invest in them as you would any user-facing feature
