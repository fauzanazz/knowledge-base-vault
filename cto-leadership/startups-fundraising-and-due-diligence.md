---
title: "Startups, Fundraising, and Due Diligence"
tags: [startups, fundraising, seed-round, due-diligence, ceo, vc, pitch-deck, lean-canvas]
category: cto-leadership
sources:
  - "85 Things I Learned Being a CEO - Sachin Gupta"
  - "Startup Playbook - Sam Altman (Y Combinator)"
  - "A Guide to Seed Fundraising - Geoff Ralston (YC)"
  - "YC Series A Diligence Checklist"
  - "Technology Due Diligence Checklist - AKF Partners"
  - "Technical Due Diligence Questions (GitHub)"
  - "A Guide to Surviving Tech Due Diligence - CircleCI"
  - "Lean Canvas"
last_updated: 2026-04-08
---

# Startups, Fundraising, and Due Diligence

## Startup Fundamentals

### Sam Altman's Startup Playbook (Y Combinator)

The core thesis: **Make something users love.**

> "It's much better to build a product that a small number of users love than a product a large number of users kind of like."

**The Four Things That Matter:**
1. **Great idea** (includes market size and growth rate)
2. **Great product** (users love it, not just use it)
3. **Great team** (can execute, are resilient)
4. **Great execution** (speed, focus, relentlessness)

### Idea Evaluation

**Good startup ideas have:**
- A **real problem** that people actively try to solve (even with bad solutions)
- A **why now** — what changed that makes this possible/necessary today?
- **Small initial market** that can grow — better to own a niche than compete in a crowd
- **Founder-market fit** — you understand this problem deeply from personal experience

**Red flags:**
- "I need to educate the market" → market doesn't want it
- "It's like X for Y" → derivative, not innovative
- "No competition" → either no market or you haven't looked hard enough
- "Just need to get 1% of this huge market" → no plan for customer acquisition

### 85 CEO Lessons (Distilled)

**On Leadership:**
- It's a lonely job — build a peer network of other founders
- Uncertainty is permanent — learn to operate without complete information
- Your job is to make yourself unnecessary — build systems, not dependencies
- Hire people smarter than you and get out of their way

**On Execution:**
- Speed wins — the fast eat the slow, not the big eat the small
- Focus on 1-3 things at a time — strategy is choosing what NOT to do
- Culture is what you tolerate — fire fast when values are violated
- Cash is oxygen — always know your runway

**On Product:**
- Talk to customers every week — never stop
- Measure what matters — one or two north star metrics
- Ship fast, iterate faster — perfect is the enemy of shipped

## Fundraising

### Seed Round Guide (Y Combinator)

**When to raise:**
- You have a prototype or early product
- You have some traction (users, revenue, or strong engagement)
- You need capital to reach a meaningful milestone (usually 12-18 months of runway)

**How much to raise:**
- Enough for **12-18 months** of runway
- Typical seed: $1M-$4M (2024 range, varies by market)
- Raise the minimum needed — dilution matters

**From whom:**

| Investor Type | Check Size | What They Offer |
|--------------|-----------|-----------------|
| **Angel investors** | $5K-$100K | Advice, intros, speed |
| **Seed funds** | $100K-$1M | Process, follow-on potential |
| **Accelerators** (YC, TechStars) | $125K-$500K | Network, brand, mentorship |
| **Micro VCs** | $250K-$2M | Thesis-driven, active support |

**The process:**
1. Build a target list (50-100 investors aligned to your space)
2. Get warm intros (cold emails work < 5% of the time)
3. Pitch (deck + demo + metrics)
4. Due diligence (see below)
5. Term sheet → negotiate → close

### Pitch Deck Structure

| Slide | Content | Time |
|-------|---------|------|
| 1. Cover | Company name, one-line description | 5 sec |
| 2. Problem | The pain point, who feels it | 30 sec |
| 3. Solution | Your product, how it solves the pain | 30 sec |
| 4. Demo/Product | Screenshots, video, or live demo | 60 sec |
| 5. Traction | Users, revenue, growth rate | 30 sec |
| 6. Market | TAM/SAM/SOM, why it's big enough | 30 sec |
| 7. Business Model | How you make money, unit economics | 30 sec |
| 8. Competition | Your advantage, positioning | 30 sec |
| 9. Team | Why you're the right people | 30 sec |
| 10. Ask | How much, what milestones | 15 sec |

### Lean Canvas (1-Page Business Model)

```
┌──────────────┬──────────────┬──────────────┐
│   Problem    │  Solution    │ Unique Value │
│  (top 3)     │  (top 3)     │  Proposition │
│              │              │              │
├──────────────┼──────────────┼──────────────┤
│ Key Metrics  │   Channels   │   Unfair     │
│ (what you    │ (how you     │  Advantage   │
│  measure)    │  reach them) │              │
├──────────────┼──────────────┼──────────────┤
│ Cost         │              │   Revenue    │
│ Structure    │              │   Streams    │
└──────────────┴──────────────┴──────────────┘
        Customer Segments (who pays)
```

## Technical Due Diligence

When investors, acquirers, or partners evaluate your technology.

### AKF Partners Framework (15+ Years of Tech DD)

**Six Areas of Assessment:**

#### 1. Organization & Talent
- Team size, structure, and skill distribution
- Key person dependencies (bus factor)
- Hiring pipeline and retention metrics
- Engineering culture (code review, testing, docs)

#### 2. Architecture & Technology
- System architecture diagram (current state)
- Technology choices and rationale
- Scalability plan (current capacity vs projected needs)
- Technical debt inventory
- Security posture

#### 3. Development Process
- CI/CD maturity
- Testing strategy (unit, integration, e2e)
- Deployment frequency and lead time
- Code review practices
- Documentation quality

#### 4. Operations & Reliability
- Uptime history (SLA/SLO)
- Monitoring and alerting
- Incident response process
- Disaster recovery plan
- On-call rotation

#### 5. Data & Compliance
- Data architecture and governance
- Privacy compliance (GDPR, CCPA)
- Data backup and recovery
- Third-party data dependencies
- Audit trail

#### 6. Product & Roadmap
- Product-market fit evidence
- Feature development velocity
- Customer feedback loop
- Competitive technical advantages
- Technical roadmap alignment with business goals

### Common DD Red Flags

| Red Flag | What It Means |
|----------|--------------|
| No automated tests | Fragile, risky to change |
| Single point of failure (person) | High key-person risk |
| No monitoring/alerting | Flying blind in production |
| Manual deployments | Error-prone, slow |
| No documentation | Knowledge is tribal |
| Massive tech debt with no plan | Velocity will only decrease |
| No security practices | Liability time bomb |
| Outdated dependencies | Security vulnerabilities |

### Preparing for Due Diligence (CTO Checklist)

**6 months before (proactive):**
- [ ] Architecture diagram up to date
- [ ] Technical debt inventory with prioritization
- [ ] Security audit completed
- [ ] CI/CD pipeline documented
- [ ] Key metrics dashboard (deploy freq, lead time, MTTR, error rates)

**1 month before:**
- [ ] Clean up documentation
- [ ] Prepare data room (access to repos, dashboards, docs)
- [ ] Brief the team (they'll be interviewed)
- [ ] Honest assessment of weaknesses (better you surface them than they discover them)

## Startup Finance for CTOs

### Equity and Compensation

**Dan Luu's Analysis — Options vs Cash:**

The expected value calculation most startups present:
> "Our options are worth $X based on our last valuation"

**Reality check:**
- Startups quote **post-money valuation** — your options are worth post-money × your %
- But 90% of startups fail → expected value = 0.1 × quoted value
- Liquidation preferences mean investors get paid first — your common shares may be worth $0 even in a "successful" exit
- Options have a **strike price** — you only profit if exit price > strike price
- **Exercise window** — you usually have 90 days after leaving to exercise (costs real money)

**Practical framework:**
| Stage | Discount to quoted value |
|-------|------------------------|
| Pre-seed/Seed | Options worth ~5-10% of quoted value |
| Series A | ~10-20% of quoted value |
| Series B+ | ~20-40% of quoted value |
| Pre-IPO | ~50-70% of quoted value |

**CTO equity benchmarks:**

| Stage | CTO Equity Range |
|-------|-----------------|
| Co-founder | 15-25% (pre-dilution) |
| First hire (pre-seed) | 2-5% |
| Post-seed hire | 0.5-2% |
| Post-Series A | 0.25-1% |

### Financial Planning for CTOs

What a CTO should know about company finances:

1. **Runway** — months of cash remaining at current burn rate
2. **Burn rate** — monthly cash outflow
3. **Unit economics** — LTV:CAC ratio (target > 3:1)
4. **Engineering cost per feature** — helps prioritize roadmap
5. **Infrastructure costs** — cloud bills, third-party services

### Budgeting

**Engineering budget rule of thumb:**
- **Pre-PMF:** 70% product, 20% infrastructure, 10% tech debt
- **Post-PMF scaling:** 50% product, 25% infrastructure, 15% tech debt, 10% R&D
- **Mature:** 40% product, 20% infrastructure, 20% tech debt, 20% platform

## Key Takeaways

1. **Make something users love** — that's the only startup advice that matters
2. **Raise for 12-18 months** of runway, no more
3. **Options are worth much less than quoted** — discount heavily
4. **Technical due diligence is about risk assessment** — prepare honestly
5. **Know your runway** — cash is oxygen
6. **Lean Canvas before pitch deck** — validate the model before selling it
7. **CTO equity: 2-5% as first hire, 15-25% as co-founder** — negotiate with data
