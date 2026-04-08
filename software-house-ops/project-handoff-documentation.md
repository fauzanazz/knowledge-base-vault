---
title: Project Handoff & Documentation
category: software-house-ops
summary: >
  A structured guide to project handoffs and technical documentation — covering handoff types,
  checklists, documentation standards (ADRs, runbooks, API docs), the Diátaxis framework,
  knowledge transfer sessions, docs-as-code, and preventing documentation rot.
sources: web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Project Handoff & Documentation

## Why Handoffs Fail

Handoffs are one of the highest-risk moments in a project's lifecycle. Knowledge that lives
only in people's heads evaporates when those people leave. The most common failure modes:

- **Tribal knowledge** — critical context never written down.
- **Outdated docs** — README says one thing; code does another.
- **Scope gaps** — the receiving team doesn't know what they don't know.
- **No runbook** — the first production incident after handoff becomes a five-alarm fire.
- **Insufficient time** — handoffs are treated as a final task rather than an ongoing process.

The cure is treating documentation as a first-class engineering deliverable, not an afterthought.

---

## Types of Handoffs

### Dev-to-Ops (DevOps Handoff)
The development team hands a running service to an SRE/platform team. Requires: architecture
overview, deployment runbooks, alerting documentation, incident response playbooks, and on-call
context.

### Agency-to-Client
A software agency delivers a product to a client's internal team. Requires: source code
transfer, credential rotation, hosting/DNS migration, access provisioning, training sessions,
and a support transition period.

### Team-to-Team (Internal)
One squad takes ownership of a service from another (reorg, scaling, team dissolution).
Requires: domain knowledge transfer, codebase walkthroughs, ownership transfer in CODEOWNERS
and on-call rosters, and a formal "shadow on-call" period.

### Individual Offboarding
An engineer leaves and hands their responsibilities to colleagues. Requires: documentation of
ongoing work, system knowledge dump, pairing sessions, and transition of direct client
relationships.

---

## Handoff Checklist

### Code & Repository
```
[ ] Repository access granted to receiving team
[ ] CODEOWNERS updated
[ ] Branch protection rules documented
[ ] CI/CD pipeline documented; receiving team can trigger deployments
[ ] All secrets rotated and stored in approved secrets manager
[ ] Dependency inventory current (no abandoned libraries)
[ ] Open PRs reviewed and either merged or explicitly deferred
[ ] Technical debt register updated
```

### Infrastructure & Operations
```
[ ] Cloud account access transferred / receiving team added
[ ] Infrastructure-as-code (Terraform/Pulumi) in version control; state file location documented
[ ] Monitoring dashboards reviewed and handed over (Datadog, Grafana, etc.)
[ ] Alert routing updated to receiving team's on-call schedule
[ ] Runbooks exist for all SEV-1/SEV-2 failure scenarios
[ ] Backup and restore procedures documented and tested
[ ] SSL certificate renewal schedule documented
[ ] Disaster recovery playbook exists
```

### Documentation
```
[ ] README is accurate and up to date
[ ] Architecture Decision Records (ADRs) are current
[ ] API documentation is published and versioned
[ ] Data model / entity relationship diagram current
[ ] Environment variable reference complete
[ ] Third-party integrations documented (webhooks, OAuth apps, SaaS accounts)
[ ] Licence and open source compliance reviewed
```

### Knowledge Transfer
```
[ ] Live walkthrough sessions scheduled and recorded
[ ] Codebase tour video created
[ ] Q&A session held with receiving team
[ ] Shadow on-call / shadow deployment period completed
[ ] Escalation paths defined for post-handoff questions
[ ] Handoff sign-off document signed by both parties
```

---

## Documentation Standards

### README

Every repository must have a README covering:
1. **What** this service does (one paragraph).
2. **Who** owns it and how to contact them.
3. **How to run it locally** (exact commands; no assumptions).
4. **How to run tests**.
5. **How to deploy** (or where the deployment docs live).
6. **Links** to architecture docs, ADRs, and runbooks.

### Architecture Decision Records (ADRs)

ADRs capture *why* architectural decisions were made, not just *what* was decided. Future
engineers who question a design choice need the context.

```markdown
# ADR-0012: Use PostgreSQL for Event Storage

**Date:** 2025-11-03
**Status:** Accepted
**Deciders:** @alice, @bob, @carol

## Context
We need durable, queryable storage for domain events. Options considered: Kafka + Kafka Streams,
EventStoreDB, PostgreSQL with JSONB.

## Decision
Use PostgreSQL with a JSONB `payload` column and a GIN index.

## Consequences
- Positive: No new infrastructure; team already has PostgreSQL expertise.
- Positive: ACID guarantees simplify exactly-once processing.
- Negative: Not ideal for > 100 M events/day; revisit if volume grows beyond this threshold.
```

Store ADRs in the repository at `/docs/adr/` using the Nygard format or MADR (Markdown
Architectural Decision Records).

### Runbooks

A runbook is a step-by-step operational guide for a specific scenario. Format:

```markdown
## Runbook: Database Connection Pool Exhaustion

**Symptoms:** p99 latency spike; 503 errors; "too many connections" in DB logs.
**Severity:** SEV-2

### Diagnosis
1. Check DB connection count: `SELECT count(*) FROM pg_stat_activity;`
2. Identify top connection consumers: `SELECT usename, count(*) FROM pg_stat_activity GROUP BY usename;`

### Mitigation
1. Reduce pool size in app config: `DB_POOL_SIZE=10` (default 50), then rolling restart.
2. Kill idle connections: `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle' AND query_start < now() - interval '10 minutes';`

### Resolution
Open ticket to review pool sizing and add PgBouncer as a connection proxy.
```

### API Documentation

- Use **OpenAPI 3.x** (Swagger) for REST APIs; maintain the spec as code in the repository.
- Use **GraphQL Introspection + GraphDoc** for GraphQL APIs.
- Document: authentication mechanism, rate limits, error codes, pagination, versioning policy.
- Publish docs to a stable URL (Redoc, Stoplight, or GitHub Pages).
- Version the API docs alongside the API — breaking changes must update docs simultaneously.

---

## The Diátaxis Framework

Diátaxis (from Greek *diataxis* — arrangement) defines four distinct documentation types,
each serving a different user need:

| Type | User Need | Analogy | Example |
|------|-----------|---------|---------|
| **Tutorial** | Learning | A cooking class | "Build your first API in 15 minutes" |
| **How-To Guide** | Accomplishing a goal | A recipe | "How to add OAuth login" |
| **Reference** | Information look-up | A dictionary | API endpoint reference |
| **Explanation** | Understanding | An essay | "Why we use event sourcing" |

**Why this matters for handoffs:** Most engineering teams produce only Reference docs. Without
Tutorials (onboarding) and Explanation (ADRs, architecture docs), new team members are lost.
Build all four types deliberately.

---

## Knowledge Transfer Sessions

### Live Walkthrough Sessions

Schedule three types of sessions during handoff:

1. **Architecture Overview (60 min)** — high-level design, data flows, key technical decisions.
2. **Codebase Tour (90 min)** — repo structure, key modules, gotchas, where bodies are buried.
3. **Operational Walkthrough (60 min)** — how to deploy, how to debug, how to respond to alerts.

**Record all sessions.** Future engineers who join after the handoff are the real beneficiaries.

### Video Walkthroughs (Async)

For client handoffs or distributed teams, create short (5–15 min) Loom/Vidyard videos:

- "How the deployment pipeline works"
- "How to add a new feature flag"
- "How to run the database migration locally"

Store videos in the project's documentation portal alongside written docs.

### Shadow On-Call / Shadow Deployment

Before handoff is complete, the receiving team should:
- **Shadow 2–3 deployments** with the handing-off team present.
- **Shadow on-call for 1–2 weeks** — receiving team handles incidents with the original team
  available as escalation backup.

This surfaces runbook gaps and tribal knowledge before the safety net is removed.

---

## Documentation-as-Code

Treat documentation with the same discipline as application code:

- **Docs live in the repository** alongside the code they describe (not in a separate wiki that
  drifts out of sync).
- **PRs update docs** — add "documentation updated" to the PR review checklist; block merge if
  new features lack docs.
- **Automated checks** — use Vale or markdownlint to enforce style guides; check for broken
  links in CI (markdown-link-check).
- **Versioned with the code** — docs for v2.x live on the v2 branch; docs for v3.x on main.
- **Published automatically** — CI deploys docs to GitHub Pages or an internal portal on merge
  to main.

---

## Preventing Documentation Rot

Documentation rot is the gradual divergence of docs from reality. It's inevitable without
active countermeasures:

- **Doc reviews on the PR checklist** — every code PR must verify that affected docs are still
  accurate.
- **Quarterly docs audit** — assign an engineer to review the top 20 most-read documentation
  pages each quarter. Label stale pages with a `⚠️ Needs Review` banner.
- **"Last verified" timestamps** — display when a runbook or guide was last manually verified
  against live systems.
- **Documentation owner** — each major service should have a named documentation owner (can
  be the same as CODEOWNERS) responsible for accuracy.
- **Delete docs you don't maintain** — a deleted doc is better than a wrong one. Wrong docs
  cause incidents.

---

## Client Handoff Package

For agency-to-client handoffs, deliver a formal handoff package:

```
handoff-package/
├── README.md                   # Overview and how to use this package
├── credentials-inventory.md    # Where credentials are stored (NOT the credentials themselves)
├── architecture-overview.md    # System design with diagrams
├── infrastructure-guide.md     # Cloud resources, costs, scaling
├── deployment-guide.md         # Step-by-step deployment instructions
├── runbooks/                   # Operational runbooks
│   ├── incident-response.md
│   ├── backup-restore.md
│   └── ssl-renewal.md
├── api-docs/                   # OpenAPI spec + generated HTML
├── adrs/                       # Architecture Decision Records
├── third-party-integrations.md # Every external service and account
├── known-issues.md             # Technical debt and deferred work
├── training-recordings/        # Links to walkthrough videos
└── sign-off.md                 # Formal acceptance document
```

Hold a **formal acceptance review** where the client team confirms they can perform core
operational tasks independently before the handoff is declared complete.

---

## Onboarding Documentation

New engineers (internal or client-side) need their own documentation set:

- **Day 1 checklist** — accounts, tools, access, Slack channels, repo access.
- **Architecture primer** — the 30-minute read that explains the system landscape.
- **Local dev setup guide** — exact steps from a clean machine to a running local environment.
  Validate this guide with every new hire; it rots fastest of all docs.
- **Contribution guide** — branch naming, PR templates, commit conventions, review process.
- **Glossary** — domain terms and internal acronyms. Every team has them; almost none
  document them.

---

## Further Reading

- *Diátaxis Documentation Framework* — diataxis.dev
- *Docs for Developers* — Bhatti, Corleissen, et al. (Apress, 2021)
- *MADR* — Markdown Architectural Decision Records — adr.github.io/madr
- Google's Technical Writing courses — developers.google.com/tech-writing
- Vale prose linter — vale.sh
