---
title: "Project Management for Engineering Leaders"
tags: [project-management, estimation, scheduling, tpm, planning, metrics, engineering-organization]
category: cto-leadership
sources:
  - "Evidence Based Scheduling - Joel Spolsky (2007)"
  - "How Big Tech Runs Tech Projects - Gergely Orosz"
  - "The Secret to a Great Planning Process - First Round (Airbnb/Eventbrite)"
  - "What TPMs Do - Gergely Orosz"
  - "How to Scope a New Feature - Prodify"
  - "Measuring an Engineering Organization - Will Larson"
last_updated: 2026-04-08
---

# Project Management for Engineering Leaders

## Evidence-Based Scheduling (EBS)

Joel Spolsky's scheduling system replaces gut-feel estimates with **statistical modeling** based on historical data.

### The Problem with Traditional Estimation

1. Developers are **systematically optimistic** — they estimate for the happy path
2. Managers add "buffers" that are equally arbitrary
3. Nobody tracks estimation accuracy, so nobody improves

### How EBS Works

**Step 1: Break tasks into small units (< 16 hours)**
- Any task estimated at > 16 hours is too vague
- Force granularity: "Implement OAuth" → "Set up OAuth provider config" (4h) + "Build callback handler" (3h) + "Token refresh logic" (4h) + "Error states" (2h) + "Tests" (3h)

**Step 2: Track actual vs estimated time**
- For each completed task, record: estimated hours, actual hours
- Calculate velocity: `actual / estimated` for each developer
- Build a **velocity history** per person

**Step 3: Monte Carlo simulation**
- For remaining tasks, randomly sample from each developer's velocity history
- Run 1000+ simulations
- Output: probability distribution of ship dates

```
Ship date    Probability
April 15     5%
April 22     25%
April 29     60%
May 6        85%
May 13       95%
```

**Key insight:** You don't give stakeholders a date — you give them a **confidence curve**. "We're 85% likely to ship by May 6" is infinitely more useful than "We'll ship May 1."

### Making EBS Work

- **Daily timesheets** — yes, really. 15-minute granularity. Not for surveillance, for data.
- **Honest tracking** — count interruptions, meetings, context switches
- **No punishment for misses** — the system improves from variance, not from blame
- **New developers** get average team velocity until they build history

## How Big Tech Runs Projects (Without Scrum)

Gergely Orosz's research into project management at Google, Meta, Amazon, Apple:

### The Surprising Finding

**Most Big Tech companies don't use Scrum.** They use lighter-weight, engineering-driven approaches:

| Company | Approach |
|---------|----------|
| Google | OKRs + engineering-driven planning, no sprints |
| Meta | Move fast, shipping-oriented, very flat |
| Amazon | 6-pagers + working backwards, 2-pizza teams |
| Apple | DRI (Directly Responsible Individual) model |
| Spotify | Squads + Chapters (but they've since moved away) |

### Common Patterns Across Big Tech

1. **Engineers own project management** — no separate PM layer for execution
2. **Planning cycles are quarterly**, not bi-weekly sprints
3. **Written communication** > standups (Amazon's 6-pagers, RFCs)
4. **DRI model** — one person accountable, not a team consensus
5. **Metrics-driven** — ship, measure, iterate (not estimate, commit, deliver)

### Why Scrum Doesn't Scale

- **Ceremony overhead** increases linearly with team size
- **Sprint commitments** create perverse incentives (sandbagging, cutting corners)
- **Velocity** becomes a management metric instead of a team tool
- **Cross-team coordination** doesn't fit 2-week cycles

### What Works Instead

- **RFC/Design Doc culture** — written proposals reviewed asynchronously
- **Milestone-based planning** — define outcomes, not task lists
- **Demo culture** — show working software, not slide decks
- **Incident-driven improvement** — postmortems > retrospectives

## Planning at Scale (Airbnb & Eventbrite)

### Airbnb's Planning Process

**Two planning horizons:**
1. **Strategic (annual):** CEO sets 3-5 company priorities
2. **Tactical (quarterly):** Teams propose projects aligned to priorities

**The "W Framework":**
```
Context (Why) → Strategy (What) → Roadmap (How) → Goals (Measure)
```

Each team writes a **one-pager** per quarter:
- What are we doing?
- Why does it matter?
- How will we know it worked?
- What are we NOT doing? (explicit de-scoping)

### Eventbrite's Approach

**Betting table model:**
- Leadership allocates "bets" (resources) to strategic areas
- Teams have autonomy within their bet
- Quarterly reviews: did the bet pay off?

### Common Pitfalls in Planning

1. **Planning too granularly** — quarterly plans should be directional, not task-level
2. **No explicit "no" list** — if everything is a priority, nothing is
3. **Ignoring maintenance** — reserve 20-30% for tech debt and reliability
4. **Planning in isolation** — cross-team dependencies must surface early

## The TPM Role

Technical Program Managers (TPMs) exist specifically for **cross-team coordination** that individual teams can't handle.

### What TPMs Actually Do

| Activity | % Time | Why Engineers Can't |
|----------|--------|-------------------|
| Cross-team dependency management | 30% | Requires org-wide visibility |
| Risk identification & mitigation | 25% | Engineers focus on their scope |
| Communication (up, down, sideways) | 20% | Time-consuming, different audiences |
| Process design & improvement | 15% | Requires systems thinking |
| Escalation & unblocking | 10% | Political capital management |

### When You Need a TPM

- **Cross-team projects** with 3+ teams involved
- **Platform migrations** affecting multiple consumers
- **Compliance/regulatory deadlines** with external accountability
- **M&A integration** technical workstreams

### When You Don't Need a TPM

- Single-team projects (engineer can PM themselves)
- Well-scoped features with clear ownership
- Exploratory/research work

## Feature Scoping

### The Prodify Framework

**Step 1: Define success metrics BEFORE any work**
- "How will we know this feature succeeded?"
- Must be measurable: adoption rate, revenue impact, support ticket reduction
- Get stakeholder sign-off on metrics

**Step 2: Understand the release sequence**
- What's the MVP? What's phase 2?
- What can be de-scoped without losing the core value?
- Which parts have technical dependencies?

**Step 3: Risk assessment**
- Technical unknowns → spike/prototype first
- Design unknowns → user research first
- Market unknowns → competitive analysis first

**Step 4: T-shirt sizing before detailed estimation**
- S (< 1 week), M (1-2 weeks), L (2-4 weeks), XL (> 1 month)
- XL means "break it down further"

## Measuring Engineering Organizations

Will Larson's framework from *The Engineering Executive's Primer*:

### Core Principle

> "There is no one solution to engineering measurement. Becoming an effective engineering executive is adding more approaches to your toolkit."

### Measurement Approaches

**1. Quantitative Metrics (Use with Caution)**

| Metric | Useful For | Dangerous When |
|--------|-----------|----------------|
| Deploy frequency | CI/CD health | Used as team performance metric |
| Lead time | Process bottleneck detection | Compared across different-complexity teams |
| MTTR | Incident response capability | Used to blame individuals |
| Change failure rate | Release quality | Drives fear of shipping |

These are the **DORA metrics** — useful as diagnostics, dangerous as targets (Goodhart's Law).

**2. Qualitative Measures**

- **Engineering surveys** — developer satisfaction, tooling effectiveness
- **Architecture reviews** — is the system getting better or worse?
- **Incident reviews** — are we learning from failures?
- **Hiring pipeline health** — can we attract and retain talent?

**3. Business Impact**

The ultimate measure: does engineering output move business metrics?
- Feature X shipped → conversion rate changed by Y%
- Platform investment → deploy frequency improved by Z%
- Reliability work → customer churn reduced by W%

### What NOT to Measure

- **Lines of code** — incentivizes bloat
- **Story points completed** — incentivizes inflation
- **Hours worked** — incentivizes presenteeism
- **Number of PRs** — incentivizes small, meaningless changes

### The Measurement Stack

```
Layer 1: Business outcomes (revenue, retention, NPS)
   ↑ influenced by
Layer 2: Product metrics (adoption, engagement, conversion)
   ↑ influenced by
Layer 3: Engineering metrics (DORA, reliability, velocity)
   ↑ influenced by
Layer 4: Team health (satisfaction, retention, growth)
```

**Measure all four layers.** Over-indexing on any single layer creates blind spots.

## Key Takeaways

1. **Evidence > intuition** for scheduling — track velocity, use Monte Carlo
2. **Big Tech abandoned Scrum** for a reason — lighter-weight, engineering-driven approaches work better at scale
3. **Planning should be directional**, not granular — quarterly priorities, not sprint tasks
4. **TPMs are for cross-team coordination**, not for managing engineers
5. **Feature scoping starts with success metrics**, not technical requirements
6. **Measure outcomes, not output** — DORA metrics are diagnostics, not KPIs
7. **The "no" list is more important than the "yes" list** in planning
