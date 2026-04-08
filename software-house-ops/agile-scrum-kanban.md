---
title: "Agile, Scrum & Kanban Deep Dive"
category: software-house-ops
summary: "A comprehensive reference covering Agile principles, Scrum ceremonies and artifacts, Kanban flow mechanics, hybrid Scrumban, scaling frameworks (SAFe/LeSS), and the metrics and anti-patterns every software house must know."
sources:
  - web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Agile, Scrum & Kanban Deep Dive

> A comprehensive reference covering Agile principles, Scrum ceremonies and artifacts, Kanban flow mechanics, hybrid Scrumban, scaling frameworks (SAFe/LeSS), and the metrics and anti-patterns every software house must know.

---

## 1. The Agile Manifesto — Core Principles

Published in 2001, the Manifesto distills 12 principles into 4 value statements:

| We value **more**               | Over                              |
|---------------------------------|-----------------------------------|
| Individuals and interactions    | Processes and tools               |
| Working software                | Comprehensive documentation       |
| Customer collaboration          | Contract negotiation              |
| Responding to change            | Following a plan                  |

**Key operating principles for a software house:**
- Deliver working software frequently (weeks, not months).
- Welcome changing requirements — even late in development.
- Maintain a sustainable pace indefinitely (no death marches).
- Continuous attention to technical excellence enhances agility.
- Simplicity — maximising the work *not* done — is essential.

---

## 2. Scrum

### 2.1 Roles

| Role               | Responsibility                                                                 |
|--------------------|--------------------------------------------------------------------------------|
| **Product Owner**  | Owns the Product Backlog; prioritises items by value; single point of authority for *what* gets built |
| **Scrum Master**   | Servant-leader; removes impediments; coaches team on Scrum; shields team from external disruption |
| **Dev Team**       | Cross-functional, self-organising; 3–9 people; owns *how* work gets done       |

### 2.2 Ceremonies

| Ceremony                  | Frequency       | Timebox (2-wk sprint) | Purpose                                               |
|---------------------------|-----------------|------------------------|-------------------------------------------------------|
| **Sprint Planning**       | Start of sprint | ≤ 4 hours              | Select backlog items; define Sprint Goal; create Sprint Backlog |
| **Daily Scrum (Standup)** | Daily           | 15 min                 | Inspect progress toward Sprint Goal; adapt plan       |
| **Sprint Review**         | End of sprint   | ≤ 2 hours              | Demo to stakeholders; inspect increment; update backlog |
| **Sprint Retrospective**  | End of sprint   | ≤ 1.5 hours            | Inspect team process; create improvement action items |
| **Backlog Refinement**    | Mid-sprint      | ~10% of sprint capacity | Groom, estimate, and split upcoming stories           |

### 2.3 Artifacts

- **Product Backlog** — Ordered list of everything that *might* be needed in the product. Single source of truth for requirements.
- **Sprint Backlog** — Subset of Product Backlog selected for the sprint + the plan for delivering the Sprint Goal.
- **Increment** — Sum of all completed PBIs; must meet the **Definition of Done (DoD)**.

### 2.4 Story Points & Estimation

Story points measure *relative complexity + effort + uncertainty*, not hours.

| Points (Fibonacci) | Meaning                                      |
|--------------------|----------------------------------------------|
| 1                  | Trivial; well-understood change              |
| 2–3                | Small, low uncertainty                       |
| 5                  | Medium complexity or some unknowns           |
| 8                  | Large; consider splitting                    |
| 13+                | Too large; must be decomposed                |

**Planning Poker** is the standard elicitation technique: everyone votes simultaneously, outliers discuss, re-vote until consensus.

### 2.5 Velocity

```
Velocity = Σ story points of accepted PBIs per sprint
```

- Stabilises after 3–5 sprints.
- Use a **rolling 3-sprint average** for forecasting.
- Never use velocity as a performance target — it will be gamed.
- Capacity adjusts for holidays, leave, and onboarding overhead.

**Example forecast:**

| Sprint | Completed pts | 3-sprint avg | Remaining backlog | Estimated sprints left |
|--------|---------------|--------------|-------------------|------------------------|
| S6     | 34            |              |                   |                        |
| S7     | 38            |              |                   |                        |
| S8     | 30            | **34**       | 204 pts           | ~6 sprints             |

---

## 3. Kanban

Kanban is a *flow* system, not an iterative one. Work items flow continuously through stages; there are no sprints or time-boxes.

### 3.1 Core Practices

1. **Visualise the workflow** — Board columns map to real process stages (Backlog → Analysis → Dev → Review → QA → Done).
2. **Limit WIP (Work-In-Progress)** — Each stage has a WIP cap. Exceeding it blocks new work from entering that stage.
3. **Manage flow** — Optimise for smooth, fast delivery; surface blockers immediately.
4. **Make policies explicit** — Entry/exit criteria for each column are written and visible.
5. **Implement feedback loops** — Kanban cadences: daily standup, replenishment, delivery planning, service delivery review.
6. **Improve collaboratively** — Use models (e.g., Theory of Constraints) to guide evolutionary change.

### 3.2 WIP Limits

WIP limits expose bottlenecks (a backed-up column with items at its cap signals a downstream constraint). The correct response is to **swarm** the bottleneck, not pull new work.

```
Recommended starting WIP limit per stage = (team members assigned) × 1.5
```

Tighten over time until flow is smooth.

### 3.3 Lead Time vs. Cycle Time

| Metric         | Starts when…               | Ends when…     | Meaning                        |
|----------------|----------------------------|----------------|--------------------------------|
| **Lead Time**  | Item enters the system     | Item is done   | Customer-facing delivery speed |
| **Cycle Time** | Work *actively begins*     | Item is done   | Internal process efficiency    |

> A large gap between Lead Time and Cycle Time indicates long queue/wait times — a sign of over-WIP or poor prioritisation.

### 3.4 Cumulative Flow Diagram (CFD)

The CFD plots count of items in each stage over time. Healthy patterns:
- Bands grow steadily and in parallel.
- Approximate parallelism → consistent throughput.

Warning patterns:
- **Bulging band** → work accumulating in one stage (bottleneck).
- **Flattening "Done" band** → throughput has dropped.
- **Widening gap between "Entered" and "Done"** → lead time increasing.

---

## 4. Scrumban — The Hybrid

Scrumban blends Scrum's cadences with Kanban's flow principles. Useful when:
- Teams need structure (Scrum) but have highly variable work sizes or maintenance load.
- Transitioning from Scrum toward pure Kanban.

| Feature                  | Scrum     | Kanban    | Scrumban           |
|--------------------------|-----------|-----------|---------------------|
| Iterations               | Fixed sprint | None   | Optional / loose    |
| WIP limits               | Implicit  | Explicit  | Explicit            |
| Estimation               | Required  | Optional  | Optional            |
| Roles                    | Required  | None      | Lightweight         |
| Planning cadence         | Per sprint| On-demand | Hybrid (trigger-based) |

**Trigger-based replenishment:** When the "Ready" column drops below a threshold (e.g., 3 items), a planning meeting fires automatically.

---

## 5. Scaling: SAFe & LeSS

### 5.1 SAFe (Scaled Agile Framework)

Designed for large enterprises (50–thousands of engineers). Key constructs:

- **Agile Release Train (ART)** — 50–125 people aligned to a common Program Increment (PI) of 8–12 weeks.
- **PI Planning** — 2-day all-hands event to align teams on objectives for the next PI.
- **System Demo** — End-of-sprint cross-team demo of integrated features.
- **Inspect & Adapt (I&A)** — PI-level retrospective + problem-solving workshop.

> ⚠️ SAFe is heavy. Software houses should only adopt it for multi-team programs ≥ 5 squads with hard interdependencies.

### 5.2 LeSS (Large-Scale Scrum)

Scales Scrum with minimal additional process. Rules:
- One Product Owner, one Product Backlog, one Increment.
- 2–8 feature teams share a single Sprint.
- Joint Sprint Planning Part 1 (what); separate Part 2 per team (how).
- Overall Retrospective after team retros.

LeSS is preferred when teams can be truly cross-functional and independent. SAFe is chosen when portfolio/governance alignment is unavoidable.

---

## 6. When to Use What

| Situation                                              | Best Fit         |
|--------------------------------------------------------|------------------|
| New product; greenfield; clear roadmap                 | **Scrum**        |
| Maintenance/support with unpredictable, varied tickets | **Kanban**       |
| Mixed product dev + ops/support in one team            | **Scrumban**     |
| High engineering discipline; TDD/CI/CD focus           | **XP** (+ Scrum) |
| 5+ teams; coordinated releases; enterprise client      | **SAFe / LeSS**  |
| Startup; ≤ 2 engineers; no dedicated PO                | Lightweight Scrum or Kanban |

---

## 7. Common Anti-Patterns

### Scrum Anti-Patterns

| Anti-Pattern                        | Symptom                                              | Fix                                              |
|-------------------------------------|------------------------------------------------------|--------------------------------------------------|
| **Zombie Scrum**                    | Ceremonies happen but no real inspect-and-adapt      | Make Retro actions mandatory with owners         |
| **Sprint Scope Creep**              | PO adds work mid-sprint                              | Protect Sprint Backlog; renegotiate Sprint Goal  |
| **Velocity as KPI**                 | Teams inflate points or rush quality                 | Use velocity for forecasting only; track DoD     |
| **Absent PO**                       | Stories are vague; blockers pile up                  | PO must be available ≥ 50% of sprint             |
| **No DoD**                          | "Done" means different things to different people    | Define and post DoD; make it a team agreement    |
| **Oversized sprints (4 wk)**        | Feedback lag; late risk discovery                    | Default to 2-week sprints                        |

### Kanban Anti-Patterns

| Anti-Pattern                        | Symptom                                              | Fix                                              |
|-------------------------------------|------------------------------------------------------|--------------------------------------------------|
| **Ignored WIP limits**              | Cards pile past the cap                              | Treat WIP limit as a hard stop; swarm bottleneck |
| **Stale board**                     | Cards don't move; board doesn't reflect reality      | Daily board hygiene; make updates mandatory      |
| **No explicit policies**            | "Ready" means nothing consistent                     | Write entry criteria on each column header       |
| **Endless backlog**                 | Hundreds of items; no prioritisation                 | Limit backlog size; use WSJF or CoD scoring      |

---

## 8. Metrics That Matter

| Metric                  | Framework    | Target / Benchmark                          |
|-------------------------|--------------|---------------------------------------------|
| **Velocity (rolling 3)** | Scrum       | Stable ± 15% sprint-over-sprint             |
| **Sprint Goal hit rate** | Scrum       | ≥ 80% (lower = planning or dependency issue) |
| **Cycle Time**          | Kanban       | Trending down; P85 < agreed SLA             |
| **Throughput**          | Kanban       | Items/week; more meaningful than story pts  |
| **WIP Age**             | Kanban       | Flag items older than 2× average cycle time |
| **Escaped Defects**     | Both         | Bugs found in prod / total stories shipped  |
| **Deployment Frequency**| Both (DevOps)| Elite: on-demand / multiple per day         |
| **MTTR**                | Both (DevOps)| Elite: < 1 hour                             |

> Use the **DORA metrics** (Deployment Frequency, Lead Time for Changes, Change Failure Rate, MTTR) alongside Agile flow metrics for a complete health picture.

---

## 9. Quick-Reference Cheat Sheet

```
SCRUM SPRINT CYCLE (2 weeks)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Day 1:  Sprint Planning (≤ 4h) → Sprint Backlog locked
Day 1–9: Dev + Daily Scrum (15 min each morning)
Day 8:  Backlog Refinement (1–2h)
Day 10: Sprint Review (≤ 2h) → Demo to stakeholders
Day 10: Sprint Retrospective (≤ 1.5h) → Improvement actions

KANBAN PULL SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Backlog] → [Analysis/2] → [Dev/4] → [Review/2] → [QA/2] → [Done]
              ↑ WIP cap        ↑            ↑           ↑
         Pull only when downstream has capacity
```

---

## 10. Software House Context

- **Client-facing projects** → Scrum (sprint demos build client trust; velocity enables deadline forecasting).
- **Internal platform/tooling** → Kanban (continuous improvement, no artificial sprints).
- **Dedicated support squads** → Kanban with SLA tiers (P1 < 4h, P2 < 1 day, P3 < 1 week).
- **Multi-client parallel work** → Scrumban; explicit capacity allocation per client per week.
- **Onboarding new teams** → Start with Scrum (structure), graduate to Scrumban once the team is self-organising.
- **Tooling** — Jira (Scrum + Kanban), Linear (modern UX), Shortcut, or GitHub Projects; pair with Confluence/Notion for DoD and wiki.
