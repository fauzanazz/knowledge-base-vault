---
title: "Team Topologies"
category: software-house-ops
summary: "Team Topologies is an organisational design framework that defines four team types and three interaction modes to help software houses reduce cognitive load, align team boundaries with system architecture, and accelerate delivery flow."
sources:
  - web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Team Topologies

> Team Topologies is an organisational design framework that defines four team types and three interaction modes to help software houses reduce cognitive load, align team boundaries with system architecture, and accelerate delivery flow.

---

## Why This Matters for a Software House

Software houses juggle multiple client engagements, shifting stacks, and variable headcount. Ad-hoc org charts breed miscommunication, duplicated platforms, and hidden dependencies that slow every project. Team Topologies gives you a shared vocabulary and a repeatable structural playbook so that teams, clients, and leadership all reason about organisation the same way.

---

## Core Principle: Conway's Law & the Inverse Conway Maneuver

> *"Organisations which design systems are constrained to produce designs which are copies of the communication structures of those organisations."*
> — Melvin Conway, 1968

**The implication:** If your org chart is tangled, your architecture will be too — layered monoliths, chaotic microservices, or API seams that match political rather than technical boundaries.

**Inverse Conway Maneuver:** Deliberately reshape your team structure to match the *target* architecture you want, then let the code follow. In a software house this means designing client-engagement teams around bounded contexts *before* writing the first service, not retrofitting ownership after the fact.

---

## Cognitive Load Theory

Every team has a finite cognitive budget. Overload it and quality, velocity, and morale all drop. Cognitive load comes in three flavours:

| Type | Description | Example |
|------|-------------|---------|
| **Intrinsic** | The complexity inherent to the domain | Pricing engine business rules |
| **Extraneous** | Accidental complexity from tooling, process | Fragile CI, obscure deploy scripts |
| **Germane** | Productive learning that builds capability | Architecture decisions, code review |

**Goal:** Minimise extraneous load (good platforms, clear APIs) so teams can maximise germane load for the client's problem.

---

## The Four Team Types

### 1. Stream-Aligned Team *(primary type)*

- **Mission:** Deliver a continuous flow of value for a specific product or business stream.
- **Owns:** End-to-end slice of the system — code, tests, deployment, on-call.
- **In a software house:** Most client-facing delivery squads. One team per client workstream or product domain.
- **Size:** 5–9 engineers (see [Two-Pizza Rule](#team-sizing--dunbars-number)).
- **Key property:** Minimise hand-offs; the team can deploy independently without waiting on others.

### 2. Enabling Team

- **Mission:** Help stream-aligned teams acquire missing skills or adopt new practices. They are a temporary scaffolding, not a permanent dependency.
- **Owns:** A capability gap (e.g. observability, accessibility, security hardening).
- **In a software house:** A small chapter of senior engineers or specialists who embed in a delivery team for a sprint or two, upskill them, then move on.
- **Anti-pattern:** Becoming a gating authority or permanent "centre of excellence" that squads must queue for.

### 3. Complicated-Subsystem Team

- **Mission:** Own a component whose domain requires deep specialist knowledge that most engineers cannot reasonably maintain.
- **Owns:** The subsystem; exposes a well-defined API or library to stream-aligned teams.
- **In a software house:** Rare — examples: ML model serving layer, a custom DSP/audio engine, a cryptography module, or a legacy ERP adapter.
- **Key property:** Used only when the knowledge barrier is real and persistent, not as a political escape hatch.

### 4. Platform Team

- **Mission:** Provide a compelling internal product (the platform) that reduces cognitive load for stream-aligned teams.
- **Owns:** Shared infrastructure, tooling, and services consumed as self-service.
- **In a software house:** Internal DevOps/platform squad managing CI/CD pipelines, cloud baseline, shared observability stack, secrets management, and scaffolding templates.
- **Golden rule:** Platform teams must treat stream-aligned teams as customers. If nobody uses the platform voluntarily, it is not a platform — it is a bottleneck with a logo.

---

## The Three Interaction Modes

How teams work together is just as important as what type they are. Interaction modes should be *explicit* and *time-bounded*.

### 1. Collaboration

- Two teams work closely together, sharing ownership, for a defined period.
- **Use:** Discovering unknowns, rapid innovation, building a new service together.
- **Cost:** High communication overhead — not sustainable long-term.
- **Signal to end it:** When the unknowns are resolved and an API boundary is clear.

### 2. X-as-a-Service

- One team consumes another's capability via a stable interface (API, CLI, npm package, Terraform module).
- **Use:** Steady-state operation. Stream-aligned teams pull platform services; complicated-subsystem teams expose libraries.
- **Requirement:** The providing team maintains a genuine developer experience: docs, versioning, SLAs, and a clear upgrade path.

### 3. Facilitating

- An enabling team works *with* a stream-aligned team to coach and upskill, without doing the work for them.
- **Use:** Introducing a new testing discipline, migrating to a new framework, embedding security practices.
- **End state:** The facilitated team is self-sufficient; the enabling team moves on.

---

## Team Sizing & Dunbar's Number

Robin Dunbar's research identifies cognitive limits on stable social relationships:

| Dunbar Layer | Size | Meaning |
|---|---|---|
| Intimate | ~5 | High-trust inner circle |
| Squad | ~15 | Working team limit |
| Tribe | ~50 | Departmental familiarity |
| Organisation | ~150 | Everyone knows everyone |

**Two-pizza rule (Bezos):** A team is too large if two pizzas cannot feed it — roughly 5–8 people.

**Practical guidance for a software house:**
- Keep delivery squads at **5–8**; beyond 9 communication paths explode (n(n−1)/2).
- Keep the whole engineering org under **~50** before introducing formal platform separation.
- Once you breach **~150**, informal coordination breaks down — formalise Team Topologies structures explicitly.

---

## Fracture Planes

A *fracture plane* is a natural seam along which you split a system or team. Good fracture planes reduce coupling and cognitive load. Common ones:

| Fracture Plane | Description | Example |
|---|---|---|
| **Business domain** | Bounded context from DDD | Billing vs. Fulfilment |
| **Regulatory compliance** | Different compliance regimes | PCI-DSS scope isolation |
| **Change cadence** | Fast-change vs. stable core | UI shell vs. payment engine |
| **Risk profile** | Mission-critical vs. experimental | Trading engine vs. A/B test feature |
| **User persona** | Different end-user groups | Consumer app vs. ops dashboard |
| **Technology** | Distinct tech stacks | Mobile native vs. web front-end |

In a software house, fracture planes also align with *client boundaries* — never let one client's domain bleed into another's team or codebase.

---

## Team API

Every team should document its **Team API** — the contract it presents to the rest of the organisation:

```yaml
team: platform
version: "2.1"
responsibilities:
  - CI/CD pipelines (GitHub Actions)
  - Cloud baseline (AWS, Terraform modules)
  - Observability stack (Grafana / Loki / Tempo)
  - Secrets management (Vault)
consumed_as: x-as-a-service
interaction_mode: x-as-a-service
office_hours: Tuesday & Thursday 14:00–15:00 UTC
on_call: #platform-oncall (PagerDuty rotation)
docs: https://internal.wiki/platform
sla:
  pipeline_build_time_p95: 8 min
  incident_response: 15 min (P1)
```

A Team API answers: *Who are you? What do you own? How do I work with you? What can I expect?* It reduces the "who do I ask?" cognitive tax that plagues growing software houses.

---

## Platform Engineering in a Software House

A mature platform is a **golden path** — the well-lit road that teams take by default, not by mandate.

**Golden path components (typical):**

1. **Scaffolding CLI** — `generate service --lang=ts --type=api` produces a repo with CI, Dockerfile, IaC, and observability baked in.
2. **CI/CD templates** — Reusable GitHub Actions / GitLab CI templates; teams opt-in rather than copy-paste.
3. **Cloud baseline module** — Terraform module for VPC, IAM, logging, alerting defaults.
4. **Observability defaults** — Every generated service ships structured logging, traces (OpenTelemetry), and a default dashboard.
5. **Inner-source catalogue** — Internal registry of approved libraries, ADRs, and runbooks.

**Avoid:** Forcing teams onto the platform via policy before it earns trust. Adoption metrics are a leading indicator of platform health.

---

## DevOps Topologies Anti-Patterns

The Team Topologies book (Skelton & Pais) explicitly names organisational anti-patterns common in software houses:

| Anti-Pattern | Symptom | Fix |
|---|---|---|
| **Dev vs. Ops silo** | "Throw it over the wall" to Ops | Embed ops capability in stream-aligned teams (you build it, you run it) |
| **DevOps team as gatekeeper** | All deployments queue through one team | Platform team enables self-service; DevOps practices spread via enabling team |
| **Shared nothing** | Every team reinvents CI, logging, IaC | Platform team with golden-path templates |
| **Shared everything** | One mega-platform team owns all tooling | Platform is too broad; split by domain (observability platform vs. deploy platform) |
| **Ivory-tower architect** | Architecture mandates from above, no team input | Architect embedded or enabled within stream-aligned teams; decisions via ADR process |
| **Hero engineer** | One person knows the critical subsystem | Complicated-subsystem team with shared ownership and runbooks |

---

## Applying Team Topologies in Practice

### Starting State (< 20 engineers)

```
┌─────────────────────────────────┐
│  Stream-Aligned Team A          │  Client / Product A
│  Stream-Aligned Team B          │  Client / Product B
│  (Platform duties shared)       │
└─────────────────────────────────┘
```

At this scale a dedicated platform team is premature. Designate a rotating **platform champion** within each squad to own shared tooling and feed a growing internal wiki.

### Scaling State (20–60 engineers)

```
┌─────────────────────────────────┐
│  Stream-Aligned Team A          │
│  Stream-Aligned Team B          │
│  Stream-Aligned Team C          │
├─────────────────────────────────┤
│  Platform Team (3–5 engineers)  │  CI/CD, cloud baseline, observability
│  Enabling Team (2–3 engineers)  │  Security, testing, architecture coaching
└─────────────────────────────────┘
```

Interaction modes: delivery squads ↔ platform = **X-as-a-Service**; enabling team ↔ delivery squads = **Facilitating**.

### Mature State (60–150 engineers)

```
┌────────────────────────────────────────────┐
│  Stream-Aligned Teams (multiple, per domain)│
├────────────────────────────────────────────┤
│  Platform Team(s)  (may split by concern)  │
│  Enabling Team(s)  (may be domain-specific)│
│  Complicated-Subsystem Team (if warranted) │
└────────────────────────────────────────────┘
```

Formalise Team APIs, run quarterly topology reviews, track cognitive load via team surveys.

---

## Key Metrics to Track

| Metric | Target signal |
|---|---|
| **Deployment frequency** | Stream-aligned teams deploy independently, multiple times/week |
| **Lead time for change** | Hours, not weeks — indicates low hand-off overhead |
| **Platform adoption rate** | % of services using golden path (voluntary) |
| **Team cognitive load score** | Self-reported survey; flag teams consistently > 7/10 |
| **Inter-team dependency count** | Decreasing over time as APIs stabilise |
| **Enabling team graduation rate** | Facilitated teams become self-sufficient |

---

## Quick Reference

```
TEAM TYPES          INTERACTION MODES
─────────────────   ──────────────────────────
Stream-Aligned  →   Collaboration      (short-term, high-bandwidth)
Enabling        →   X-as-a-Service     (steady-state, low-friction)
Complicated-Sub →   Facilitating       (coaching, then exit)
Platform
```

---

## Further Reading

- Skelton, M. & Pais, M. — *Team Topologies* (IT Revolution, 2019)
- Conway, M. — *How Do Committees Invent?* (Datamation, 1968)
- DORA — *Accelerate* (Forsgren, Humble, Kim) — delivery metrics
- Dunbar, R. — *Grooming, Gossip, and the Evolution of Language* — social layer limits
- TeamTopologies.com — remote interaction patterns, topology examples
