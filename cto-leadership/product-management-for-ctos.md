---
title: "Product Management for CTOs"
tags: [product-management, feature-teams, product-teams, shape-up, roadmap, north-star-metric, product-strategy]
category: cto-leadership
sources:
  - "What Makes a Great Product Manager - Lawrence Ripsher"
  - "How to Hire a Product Manager - Ken Norton"
  - "Product vs Feature Teams - Marty Cagan (SVPG)"
  - "Shape Up - Basecamp"
  - "Product North Star Metric - Amplitude"
  - "Most Startups Should Be Deer Hunters - Mark Suster"
  - "Execution at Facebook - Product Life"
  - "How to Write Your First Strategic Roadmap"
  - "Awesome Product Management (GitHub)"
last_updated: 2026-04-08
---

# Product Management for CTOs

## What Makes a Great Product Manager

### Ben Horowitz's Definition (Paraphrased)
> "A good product manager is the CEO of the product. A bad product manager makes excuses."

### Core Competencies (Lawrence Ripsher)

| Competency | What Great PMs Do | What Bad PMs Do |
|-----------|-------------------|-----------------|
| **Customer obsession** | Talk to users weekly, cite specific stories | Reference "the market" abstractly |
| **Data-driven** | Set metrics before building, analyze after | Cherry-pick data to support opinions |
| **Communication** | Write clearly, present concisely, listen actively | Death by slides, meeting-heavy |
| **Prioritization** | Say no to 90% of ideas with clear reasoning | Say yes to everything, build nothing well |
| **Technical fluency** | Understand trade-offs, speak engineer language | Treat engineering as a black box |
| **Strategic thinking** | Connect feature to business outcome | Build features because competitors have them |

### Hiring a PM (Ken Norton)

**What to look for:**
1. **Product sense** — Give them a product, ask them to critique it. Great PMs see what's missing, not just what's wrong.
2. **Analytical ability** — Can they structure ambiguous problems? Ask: "How would you figure out how many piano tuners are in Chicago?"
3. **Customer empathy** — Can they articulate a customer's problem without referencing the solution?
4. **Execution ability** — Have they shipped? What were the trade-offs? What would they do differently?
5. **Leadership without authority** — How do they influence when they can't direct?

**Red flags:**
- "I want to be a PM because I have ideas" → ideas are cheap, execution is hard
- Can't describe a product failure they owned → hasn't shipped enough or lacks accountability
- Only talks about features, never about problems → feature factory mindset

## Product Teams vs Feature Teams

### Marty Cagan's Framework (SVPG)

Three types of teams (most companies think they have #3 but actually have #1):

**1. Delivery Teams (Feature Teams)**
- Receive a prioritized backlog from stakeholders
- Job: build what's requested, on time
- PM = "backlog administrator"
- Success metric: output (features shipped, story points)
- **This is repackaged waterfall**, regardless of whether you use Scrum

**2. Feature Teams (Better)**
- Given a feature to build with some autonomy on implementation
- PM has more influence but still receives direction from above
- Some discovery work, but scope is largely predetermined

**3. Empowered Product Teams (The Goal)**
- Given a **problem to solve**, not a feature to build
- Team decides what to build based on discovery
- PM owns the "what" and "why," engineering owns the "how"
- Success metric: outcome (business metric movement)
- Design, engineering, and PM work as a true triad

### How to Know Which You Are

| Signal | Feature Team | Product Team |
|--------|-------------|--------------|
| Roadmap contains | Features and dates | Problems and metrics |
| PM's main activity | Writing tickets | Talking to customers |
| Success is measured by | Features shipped | Metrics moved |
| Engineers learn about "why" | After spec is written | During discovery |
| Design is involved | At the end (make it pretty) | At the beginning (shape the problem) |

### Transitioning from Feature to Product Teams

1. **Start with one team** — don't transform the whole org at once
2. **Assign a problem, not a feature** — "Reduce churn by 20%" not "Build retention dashboard"
3. **Give the team 6 weeks** to explore and propose solutions
4. **Measure outcome** — did the metric move? Not "did they ship?"
5. **Accept failure** — product teams will try things that don't work. That's the point.

## Shape Up (Basecamp)

An alternative to Scrum for small teams, created by Ryan Singer at Basecamp.

### Core Concepts

**6-week cycles** instead of 2-week sprints:
- Long enough to build something meaningful
- Short enough to maintain urgency
- No extensions — if it's not done in 6, it wasn't scoped correctly

**Shaping** (before building):
- Senior people "shape" the work — define the problem and rough solution
- Not a detailed spec — a "shaped pitch" with boundaries
- Includes **appetite** (how much time we're willing to spend) instead of estimates

**Betting table** (instead of backlog):
- No infinite backlog that grows forever
- Leadership "bets" on shaped pitches each cycle
- If a pitch isn't bet on, it's gone (can be re-pitched later)
- **Nothing is carried over automatically**

**Hill charts** (instead of burndown):
```
  Figuring out    Making it happen
      ╱╲              ╱╲
     ╱  ╲            ╱  ╲
    ╱    ╲          ╱    ╲
───╱──────╲────────╱──────╲───
  Uphill    Downhill
  (unknown) (execution)
```

Each scope moves from left (uphill/unknown) to right (downhill/execution). When everything is "over the hill," you're mostly done.

### When Shape Up Works
- Small teams (1-3 engineers + 1 designer)
- Products with clear ownership
- Teams that can work autonomously for 6 weeks
- Organizations willing to give up the illusion of long-term roadmaps

### When It Doesn't
- Large organizations with many dependencies
- Regulated environments requiring detailed planning
- Teams that can't be trusted with autonomy (hire better)

## North Star Metrics

### Concept (Amplitude)

The North Star Metric is the **single metric** that best captures the core value your product delivers to customers.

| Company | North Star Metric | Why |
|---------|------------------|-----|
| Airbnb | Nights booked | Core value = stays |
| Facebook | Daily active users | Core value = engagement |
| Spotify | Time spent listening | Core value = music enjoyment |
| Slack | Messages sent | Core value = team communication |
| Shopify | GMV (Gross Merchandise Value) | Core value = merchant sales |

### Choosing Your North Star

**Good north star metrics:**
- Reflect **customer value**, not just company revenue
- Are **leading indicators** of revenue (not lagging)
- Can be **influenced by the product team**
- Are **easy to understand** by everyone in the company

**Bad north star metrics:**
- Revenue (lagging, not directly influenced by product changes)
- Number of users (vanity, doesn't reflect engagement)
- NPS (too infrequent, too noisy)

### The Input Metric Tree

```
        North Star Metric
       /        |        \
  Input 1   Input 2   Input 3
   (team A)  (team B)  (team C)
```

Each team owns an input metric that feeds the north star. This creates alignment without top-down control.

## Strategic Roadmap

### Framework (Ganotnoa)

**A roadmap is NOT a Gantt chart.** It's a communication tool that answers:
1. **Where are we going?** (Vision)
2. **Why does it matter?** (Strategy)
3. **How will we get there?** (Themes + timeframes)
4. **How will we know we're on track?** (Metrics)

### Roadmap Levels

| Level | Audience | Timeframe | Detail |
|-------|----------|-----------|--------|
| **Vision** | Board, investors | 2-3 years | Directional themes |
| **Strategic** | Leadership | 1 year | Quarterly priorities |
| **Tactical** | Product team | 1 quarter | Specific initiatives |
| **Sprint** | Engineers | 2-6 weeks | Tasks |

**Only the sprint level should have specific dates.** Everything above is "this quarter" or "this half."

## Marketing for Technical Products

### Developer Marketing (Heavybit)

Developers are the hardest audience to market to because they:
- **Hate being sold to** — they can smell marketing a mile away
- **Value authenticity** — real examples > polished messaging
- **Trust peers** — community recommendations > ads
- **Evaluate technically** — they'll read your docs before your marketing site

### What Works

| Channel | Effectiveness | Why |
|---------|--------------|-----|
| **Technical blog** | Very high | Demonstrates expertise, SEO, trust |
| **Open source** | Very high | Builds community, goodwill, adoption |
| **Documentation** | Very high | Your docs ARE your marketing |
| **Conference talks** | High | Credibility, networking |
| **Developer relations** | High | Human connection, feedback loop |
| **Community (Discord/Slack)** | Medium-High | Direct relationship |
| **Social (X/Twitter)** | Medium | Thought leadership, reach |
| **Paid ads** | Low | Developers ignore/block ads |

### SaaS Email Marketing

Key principles for B2B SaaS:
1. **Behavioral triggers** > time-based campaigns (email when they DO something, not on a schedule)
2. **Value first** — every email should help, not just sell
3. **Segment aggressively** — free vs paid, active vs dormant, role-based
4. **Minimize frequency** — developers unsubscribe fast
5. **Plain text > HTML** — looks like a real person, not a marketing machine

## Key Takeaways

1. **Product teams > feature teams** — give problems, not features
2. **North Star Metric** aligns the whole org without top-down control
3. **Shape Up** is a compelling alternative to Scrum for small teams
4. **Hire PMs for product sense and execution**, not just domain knowledge
5. **Roadmaps are NOT Gantt charts** — communicate direction, not dates
6. **Developer marketing = content + community + docs** — not ads
7. **Strategic roadmap has 4 levels** — only the lowest has specific dates
