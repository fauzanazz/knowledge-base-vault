---
title: "Estimation Techniques"
category: software-house-ops
summary: "A practical guide to software estimation methods used in software house operations, covering why estimation fails, the most common sizing and forecasting techniques, and how to choose the right approach for fixed-price vs time-and-materials engagements."
sources:
  - web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Estimation Techniques

> A practical guide to software estimation methods used in software house operations, covering why estimation fails, the most common sizing and forecasting techniques, and how to choose the right approach for fixed-price vs time-and-materials engagements.

---

## Why Estimation Is Hard

Software estimation is notoriously difficult. Two well-studied phenomena explain most failures:

### Cone of Uncertainty
Early in a project — during concept or requirements gathering — estimates can be off by **4× in either direction**. Only as work progresses and unknowns shrink does the cone narrow toward actual cost and schedule. This means early fixed-price commitments carry significant risk unless scope is tightly controlled or large contingency is baked in.

| Phase | Accuracy Range |
|---|---|
| Initial concept | 0.25× – 4× |
| Approved product definition | 0.67× – 1.5× |
| User-story-complete backlog | 0.8× – 1.25× |
| Development complete | ~1× |

### Planning Fallacy
Coined by Kahneman & Tversky: individuals and teams systematically underestimate task duration, even when they have historical evidence of past overruns. Root causes include:
- **Optimism bias** — assuming best-case conditions.
- **Scope blindness** — missing hidden tasks (integration, testing, deployment, review cycles).
- **Anchoring** — locking to a first estimate and failing to revise sufficiently.
- **Student Syndrome** — work expands to fill the time, masking real pace until the deadline.

Awareness alone rarely cures the planning fallacy; the antidote is systematic use of historical data and structured techniques.

---

## Story Points vs. Time-Based Estimation

### Time-Based (Hours/Days)
- Directly maps to billing and scheduling.
- Feels concrete to clients; easy to compare against actuals.
- Prone to planning fallacy — engineers inflate or deflate based on social pressure.
- Requires re-estimation whenever velocity or team composition changes.

### Story Points
- Relative sizing: "this story is twice as complex as that one."
- Abstracts away individual differences in speed.
- Calibrated to a team's **velocity** (points completed per sprint).
- **Advantage**: velocity naturally adjusts as teams mature; no need to recalibrate every estimate.
- **Drawback**: meaningless to clients who need dates and budgets. Translation to calendar time requires a velocity baseline (at least 3–5 sprints of data).

**Rule of thumb for software houses**: use story points internally for sprint planning; convert to time/cost externally via velocity when quoting T&M work. For fixed-price, convert to hours with explicit risk buffers.

---

## Common Estimation Techniques

### 1. Planning Poker
A consensus-based relative estimation game for agile teams.

**Process:**
1. Product owner reads a user story.
2. Each estimator privately selects a card (Fibonacci: 1, 2, 3, 5, 8, 13, 21, ?, ∞).
3. Cards are revealed simultaneously.
4. Outliers (high/low) explain their reasoning.
5. Repeat until convergence.

**Why Fibonacci?** The gaps grow with story size, reflecting that larger stories carry proportionally more uncertainty. Using 6 or 7 would imply false precision.

**Software house use case**: best for sprint-level backlog refinement sessions (30–90 min). Not scalable for entire-project rough-order-of-magnitude (ROM) estimates.

---

### 2. T-Shirt Sizing
Quick relative bucketing: **XS / S / M / L / XL / XXL**.

- Used for epics, features, or early backlog sizing before stories are decomposed.
- Each size maps to a rough time or point range set by the team (e.g., M = 3–5 days of effort).
- Fast; enables roadmap-level capacity planning.
- Loses precision; use as a precursor to Planning Poker when stories are refined.

---

### 3. Three-Point Estimation (PERT)
For tasks where uncertainty is quantifiable, estimate three scenarios:

| Scenario | Symbol | Description |
|---|---|---|
| Optimistic | O | Everything goes right |
| Most Likely | M | Normal conditions |
| Pessimistic | P | Major problems arise |

**PERT Formula:**

```
Expected = (O + 4M + P) / 6
Standard Deviation = (P - O) / 6
```

The weighted average biases toward the most likely outcome while the standard deviation quantifies risk. Sum expected values across tasks for a project estimate; combine standard deviations in quadrature for total uncertainty.

**When to use**: fixed-price proposals, project bids, or any task where consequences of overrun are significant. Encourages explicit discussion of risk.

---

### 4. Reference Class Forecasting
Proposed by Bent Flyvbjerg: **ignore inside-view estimates; anchor to the outside view** — how long did similar projects actually take?

**Steps:**
1. Identify a reference class of comparable past projects (same type, size, domain).
2. Pull actuals: duration, cost, defect rate.
3. Adjust the new project's estimate to match the statistical distribution.
4. Apply an uplift if the new project has identifiable risk factors above the baseline.

**For software houses**: build an internal database of completed projects tagged by type (e.g., SaaS MVP, mobile app, data pipeline), team size, and tech stack. After ~20 projects, reference class forecasting can outperform expert gut-feel significantly.

---

### 5. Function Point Analysis (FPA)
A language-agnostic measure of functional size based on:
- **External Inputs** (forms, API calls processed)
- **External Outputs** (reports, API responses)
- **External Inquiries** (read-only queries)
- **Internal Logical Files** (data stores maintained)
- **External Interface Files** (data consumed from other systems)

Each is counted and weighted (simple/average/complex). Raw function points are adjusted by a **Value Adjustment Factor** (VAF) covering 14 general system characteristics.

**Use case**: requirements-complete bids on large enterprise projects; regulatory or contractual environments where a vendor-neutral size metric is required. High overhead; rarely used for small agile projects.

---

### 6. COCOMO II
**Constructive Cost Model** — an algorithmic estimation model:

```
Effort (person-months) = A × Size^B × ∏EM
Duration (months)      = C × Effort^D
```

Where:
- **Size** = thousands of source lines of code (KSLOC) or function points converted.
- **B** = scale factor (1.01–1.26) influenced by precedentedness, architecture flexibility, team cohesion, etc.
- **EM** = effort multipliers (17 cost drivers: reliability requirements, database size, platform volatility, analyst capability, etc.).

**Practical note**: COCOMO requires calibration to your organization's historical data to be accurate. Off-the-shelf defaults can easily be off by 2×. Use it as a sanity-check against expert estimates rather than a primary method for most software house work.

---

### 7. #NoEstimates
A movement (Ron Jeffries, Woody Zuill) arguing that estimation effort itself is waste and that small, continuously delivered slices eliminate the need for upfront estimates.

**Core idea**: break work into stories small enough to complete in ≤1 day; count stories completed per week; forecast remaining stories from count alone.

**Where it works**: long-running T&M products with high trust and mature DevOps.  
**Where it doesn't**: fixed-price contracts, regulatory bids, new client proposals — any context requiring a financial commitment before work starts.

**Software house verdict**: valuable as a _mindset_ (thin slices, frequent delivery), but impractical as a _billing model_ without client buy-in.

---

## Estimation by Contract Type

### Time & Materials (T&M)
- Estimate is a **forecast**, not a commitment.
- Use velocity-based forecasting: known velocity × estimated backlog size = projected sprints × sprint cost.
- Publish confidence intervals (e.g., "8–12 sprints at 85% confidence") rather than point estimates.
- Re-forecast every sprint as scope and velocity data accumulate.

### Fixed-Price
- Estimate becomes a **contract commitment** — errors come directly off margin.
- Requirements must be locked or a change-control mechanism must be agreed upfront.
- Recommended approach:
  1. Decompose to the lowest practical level (user stories or tasks).
  2. Apply three-point (PERT) estimates per task.
  3. Sum expected values for the baseline.
  4. Add explicit **contingency** (see below).
  5. Add a **margin** on top of contingency for profit.
- Consider using a **capped T&M** hybrid: T&M up to a ceiling, client absorbs scope creep.

---

## Padding & Contingency

Contingency is **not lying** — it is quantified risk. Best practice:

| Risk Level | Suggested Contingency |
|---|---|
| Well-understood domain, known stack | 15–20% |
| New domain or new client | 25–35% |
| Research-heavy / novel architecture | 40–60% |
| Integrations with undocumented third-party systems | Add 20% per integration |

**How to present contingency to clients:**
- Line-item it as "Project Risk Reserve" — clients respect transparency more than discovering hidden padding later.
- Define a formal change-request process so scope growth is captured as new line items, not absorbed into the reserve.

Avoid Parkinson's Law ("work expands to fill the time"): ring-fence contingency at the project level; don't distribute it to individual tasks where it will be consumed regardless of need.

---

## Velocity-Based Forecasting

Once a team has 3–5 sprints of data, probabilistic forecasting replaces guesswork:

1. Collect velocity samples (story points or story count per sprint).
2. Run a **Monte Carlo simulation** (or use a simple percentile approach) over the remaining backlog.
3. Report P50 (median), P85, and P95 completion dates.

**Tools**: Jira's built-in forecasting, LinearB, ActionableAgile, or a simple spreadsheet.

**Key insight**: velocity has a distribution, not a single value. Quoting a P85 date to clients ("85% likely to finish by X") sets honest expectations and reduces the adversarial dynamic when estimates slip.

---

## Tracking Estimation Accuracy

Improving estimation requires measuring it. Capture for every project:

| Metric | Formula |
|---|---|
| Estimation Error | `(Actual − Estimate) / Estimate × 100%` |
| Bias | Mean of signed errors (positive = chronic underestimation) |
| Spread | Standard deviation of errors |

**Review cadence**: per-project retrospective + quarterly aggregate review. Share anonymised project data across the delivery team to build shared calibration.

**Red flags**: consistent +30% overruns signal systemic underestimation (planning fallacy at org level). Consistent negative errors signal padding abuse or sandbag culture.

---

## Quick-Reference: Technique Selection Guide

| Situation | Recommended Technique |
|---|---|
| Sprint backlog refinement | Planning Poker |
| Early roadmap / feature sizing | T-shirt sizing |
| Fixed-price bid with clear requirements | Three-point (PERT) + contingency |
| Long T&M product, established team | Velocity-based Monte Carlo |
| Large enterprise / regulatory bid | Function Point Analysis + COCOMO sanity check |
| Organisation with 20+ comparable past projects | Reference Class Forecasting |
| High-trust T&M with mature delivery | #NoEstimates story counting |

---

## Key Takeaways

1. **No single technique is universally correct** — match method to contract type, team maturity, and available data.
2. **Estimation accuracy improves only with measurement** — track actuals religiously.
3. **The Cone of Uncertainty is real** — avoid fixed-price commitments on loosely defined scope.
4. **Contingency is a risk instrument, not dishonesty** — line-item it and define a change-control process.
5. **Velocity-based forecasting with confidence intervals** is the most honest way to communicate delivery timelines to clients.
6. **Reference class forecasting** is underused in software houses and consistently outperforms optimistic inside-view estimates once enough historical data exists.
