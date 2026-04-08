---
title: Client Management & Statement of Work (SOW)
category: software-house-ops
summary: >
  End-to-end guide for software houses covering SOW structure, proposal writing,
  discovery & scoping, requirements gathering, scope-creep management, change
  requests, client communication, RACI, risk register, billing models, legal
  clauses, and handling difficult clients.
sources: web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Client Management & Statement of Work (SOW)

## 1. Statement of Work — Structure

A well-formed SOW is the single source of truth for every engagement. Missing
sections are the root cause of most scope disputes.

| Section | What to Include |
|---|---|
| **Scope** | What is IN scope (features, platforms, integrations) and explicitly what is OUT of scope |
| **Deliverables** | Tangible artifacts: source code, designs, documentation, staging/prod deployments |
| **Timeline** | Phase breakdown with start/end dates, milestone gates, buffer allowances |
| **Acceptance Criteria** | Measurable pass/fail conditions per deliverable (not "done when client is happy") |
| **Change Order Process** | How changes are requested, costed, approved, and documented |
| **Assumptions** | Environment access, third-party APIs, client-supplied content, team availability |
| **Dependencies** | Client-side work that unblocks your team (e.g., design assets, credentials) |
| **Payment Schedule** | Milestone or retainer billing tied to deliverable acceptance |
| **Governance** | Primary contacts, escalation path, meeting cadence |

> **Rule of thumb:** If a reasonable person could interpret it two different ways,
> write it out explicitly. Ambiguity always costs money.

---

## 2. Proposal Writing

A proposal is a sales document and a scoping contract rolled into one. Keep it
concise but complete.

### Structure
1. **Executive Summary** — one page; tie your solution to the client's business goal.
2. **Problem Statement** — reflect their pain points back to them; shows you listened.
3. **Proposed Solution** — high-level architecture and approach; avoid over-engineering.
4. **Scope & Deliverables** — borrow directly from the SOW template above.
5. **Timeline & Milestones** — visual Gantt or phase table.
6. **Team & Roles** — named or role-based; seniority signals matter.
7. **Pricing** — itemised or by phase; always include a change-order rate card.
8. **Terms** — payment schedule, IP ownership, NDA reference, termination clause.
9. **Appendices** — case studies, tech stack rationale, references.

### Pricing Models
- **Fixed-price** — low client risk, high agency risk; requires airtight scope.
- **Time & Materials (T&M)** — fair for exploratory projects; needs trust and transparency.
- **Milestone-based** — hybrid; fixed price per milestone, renegotiable between phases.
- **Retainer** — recurring capacity reservation; ideal for ongoing product work.

---

## 3. Discovery & Scoping Phase

Discovery is an investment, not overhead. Charge for it. A paid discovery reduces
risk for both parties and produces artefacts that anchor the SOW.

### Discovery Deliverables
- Stakeholder interview notes (business goals, pain points, success metrics)
- As-is / to-be process maps
- High-level system architecture diagram
- Prioritised feature list (MoSCoW: Must / Should / Could / Won't)
- Technical risk log (pre-build)
- Rough effort estimate with confidence interval (±20 % at discovery stage is honest)

### Discovery Checklist
- [ ] Stakeholder alignment — who has final say?
- [ ] Budget validated against scope
- [ ] Non-functional requirements captured (performance, uptime, compliance, scale)
- [ ] Third-party integrations listed with API documentation available?
- [ ] Existing codebase / tech debt assessed if brownfield
- [ ] Data migration scope defined
- [ ] Legal / regulatory constraints identified (GDPR, HIPAA, PCI-DSS, etc.)

---

## 4. Requirements Gathering

### User Stories
Format: _As a [persona], I want to [action], so that [outcome]._

Good user stories are **INVEST**:
- **I**ndependent — can be developed without blocking others
- **N**egotiable — not a fixed contract; open to discussion
- **V**aluable — tied to a user or business outcome
- **E**stimable — team can size it in story points / hours
- **S**mall — completable in a single sprint
- **T**estable — has clear acceptance criteria

### Acceptance Criteria (Definition of Done)
Each story should have **Given / When / Then** scenarios:

```
Given: A logged-in user is on the checkout page
When: They click "Pay Now" with a valid card
Then: The order is created, a confirmation email is sent within 60 s,
      and the user is redirected to /orders/{id}
```

Tie acceptance criteria directly to QA test cases. This eliminates "but I
thought it would also do X" at UAT.

---

## 5. Managing Scope Creep

Scope creep is not the client's fault — it is an agency governance failure.

### Prevention
- Reference the SOW explicitly in all written communication.
- Include a concise "what is out of scope" list in the SOW.
- Price discovery and scoping phases separately from build phases.
- Send a written summary after every meeting that captures decisions made.

### Detection Signals
- Client emails requesting new features with "while you're at it…"
- "Small tweaks" accumulating across multiple tickets
- Sprint velocity dropping without team changes
- QA findings requiring significant new build work

### Response Protocol
1. Acknowledge the request positively — never refuse outright.
2. Log it in the change request register.
3. Estimate impact on timeline and cost.
4. Present options: defer to phase 2, include as change order, or swap for existing scope.
5. Get written sign-off before any work begins.

---

## 6. Change Request (CR) Process

Every change, no matter how small, must go through a formal CR.

### CR Workflow
```
Client Request → PM logs CR → Estimate (effort + cost + timeline impact)
→ Present to client in writing → Client approves/rejects in writing
→ Approved: update SOW, invoice accordingly → Rejected: log and close
```

### CR Template Fields
- CR number & date
- Requested by (client contact)
- Description of change
- Reason / business justification
- Effort estimate (hours)
- Cost impact
- Timeline impact
- Affected deliverables / milestones
- Approval signature (client) & date

---

## 7. Client Communication Cadence

| Touchpoint | Frequency | Audience | Purpose |
|---|---|---|---|
| Sprint Review / Demo | Every 1–2 weeks | Stakeholders + PM | Show working software, gather feedback |
| Status Report | Weekly (async) | Client PM | RAG status, progress, blockers, next steps |
| Steering Committee | Monthly | Executives | Budget, strategic direction, risk review |
| Ad-hoc Slack / email | As needed | Day-to-day contacts | Quick questions, decisions |
| Retrospective | End of phase | Joint team | Process improvements, lessons learned |

### Communication Rules
- All decisions confirmed in writing (email or ticket comment).
- Never communicate bad news verbally only — follow up in writing with context and mitigation plan.
- Response SLA: acknowledge within 4 business hours, full response within 1 business day.
- Keep a shared project log (e.g., Notion, Confluence) as the single source of truth.

---

## 8. RACI Matrix

Define responsibility at the start of every project to prevent "I thought you
were handling that."

| Activity | Agency PM | Agency Dev Lead | Client PM | Client SME |
|---|---|---|---|---|
| Requirements sign-off | C | I | A | R |
| Architecture decisions | I | R | C | I |
| UAT execution | I | C | A | R |
| Production deployment approval | C | R | A | I |
| Invoice approval | I | I | A | R |
| Change order approval | C | I | A | R |

**R** = Responsible · **A** = Accountable · **C** = Consulted · **I** = Informed

---

## 9. Risk Register

Maintain a living risk register updated at every sprint review.

| # | Risk | Likelihood | Impact | Severity | Mitigation | Owner |
|---|---|---|---|---|---|---|
| R-01 | Key client contact unavailable during UAT | Medium | High | High | Agree UAT schedule 2 sprints ahead; name backup contact | Client PM |
| R-02 | Third-party API changes breaking integration | Low | High | Medium | Pin API version; monitor changelog; contract SLA with vendor | Dev Lead |
| R-03 | Scope underestimated during discovery | Medium | High | High | 20% contingency in estimates; CR process enforced | Agency PM |
| R-04 | Client delays content/asset delivery | High | Medium | High | Dependency mapped in SOW; blocker escalation SLA 48 h | Client PM |
| R-05 | Key developer leaves during project | Low | Very High | High | Cross-training, documented code, notice period in contract | Agency PM |

---

## 10. Milestone-Based Billing

Tie payments to measurable deliverables, not calendar dates.

### Typical Milestone Structure
| Milestone | % of Project Fee | Trigger |
|---|---|---|
| Project kickoff | 25–30% | SOW signed, deposit received |
| Design / Architecture sign-off | 15–20% | Client approves wireframes & system design |
| Beta / UAT release | 25–30% | Feature-complete build delivered to staging |
| Production launch | 15–20% | Go-live, client acceptance signed |
| Warranty / Stabilisation end | 5–10% | 30-day post-launch support period closed |

> **Tip:** Never start the next phase without receiving payment for the previous
> milestone. Cash flow is project health.

---

## 11. Retainer Agreements

Retainers provide predictable revenue for the agency and priority access for the
client.

### Key Clauses
- **Monthly capacity** — hours reserved per month (e.g., 80 h/month).
- **Rollover policy** — unused hours expire or roll over (cap at 1 month to avoid debt accumulation).
- **Rate card** — hourly rate for hours used beyond retainer cap.
- **Notice period** — minimum 30–60 days to terminate; prevents abrupt revenue loss.
- **Scope of work** — broad enough for flexibility, narrow enough to avoid unplanned work.
- **Reporting** — monthly hour utilisation report sent to client.

---

## 12. NDA & IP Clauses

### NDA Essentials
- Mutual vs. one-way — default to mutual for software house engagements.
- Definition of confidential information — be explicit; "all technical and business information" is too vague in disputes.
- Carve-outs — information already public, independently developed, received from third parties.
- Duration — 2–3 years post-project is standard; perpetual NDAs are rarely enforceable.
- Permitted disclosures — employees, contractors, legal counsel on a need-to-know basis.

### IP Ownership
- **Work-for-hire** (client owns all IP) — appropriate for bespoke client projects; charge a premium.
- **Dual licence** — client gets a perpetual licence; agency retains the right to reuse patterns/frameworks.
- **Agency retains IP** — rarely acceptable to clients; only viable for SaaS with seat-based licences.
- **Open-source components** — list all OSS dependencies and their licences in the SOW appendix.
- **Pre-existing IP** — explicitly carve out your internal frameworks, tools, and libraries.

---

## 13. Project Kickoff Checklist

- [ ] SOW and MSA signed by both parties
- [ ] Deposit invoice paid
- [ ] NDA executed
- [ ] Access provisioned: repo, project management tool, staging environment, third-party APIs
- [ ] Communication channels set up (Slack workspace, email DLs, calendar invites)
- [ ] RACI matrix shared and accepted
- [ ] Initial backlog created and prioritised
- [ ] Definition of Done agreed and documented
- [ ] CI/CD pipeline and branching strategy agreed
- [ ] Security requirements documented (auth method, data handling, OWASP standards)
- [ ] First sprint planned and story points estimated
- [ ] Escalation path and decision-maker contacts documented
- [ ] Client onboarding deck / project handbook shared

---

## 14. Client Red Flags

Watch for these signals during sales and early engagement. They predict future
conflict.

| Red Flag | Risk | Response |
|---|---|---|
| "We just need a rough estimate, we'll figure out details later" | Scope explosion | Require discovery phase before any estimate |
| Multiple stakeholders with conflicting priorities | Decision paralysis, rework | Insist on single accountable client-side PM |
| "The last agency was terrible" (no specifics) | Unrealistic expectations or difficult client | Ask for post-mortem; probe why relationship ended |
| Pressure to skip discovery / start coding immediately | Misaligned expectations | Explain risk; offer a 1-week mini-discovery |
| Unusual payment resistance or delays | Cash flow risk, bad faith | Enforce milestone billing strictly; consider walking away |
| Scope defined verbally only, resistance to documentation | Liability exposure | No SOW = no project; non-negotiable |
| "We'll handle the legal stuff later" | IP and liability exposure | MSA and NDA before any work or sharing of confidential info |
| Demanding 24/7 availability from day one | Burnout, poor boundaries | Define availability windows in writing at kickoff |

---

## 15. Handling Difficult Clients

### Principles
1. **Stay factual, not emotional.** Reference the SOW, meeting notes, and ticket history — not feelings.
2. **Escalate early.** A brewing conflict addressed at week 2 is a 30-minute conversation; at week 8 it is a legal dispute.
3. **Document everything.** Every verbal conversation followed by a written summary. Every.
4. **Offer choices, not ultimatums.** "We can do X within scope or Y as a change order — which would you prefer?"

### Escalation Ladder
```
1. PM ↔ Client PM (resolve at working level)
2. Agency Delivery Lead ↔ Client Sponsor (if unresolved in 48 h)
3. Agency Principal ↔ Client Executive (strategic / relationship level)
4. Formal dispute resolution per MSA (mediation → arbitration → litigation)
```

### When to Fire a Client
- Persistent late payment beyond agreed terms despite escalation.
- Bad-faith scope manipulation after multiple documented CRs.
- Abusive behaviour toward agency team members.
- Requests to do something unethical or illegal.
- Project economics make continuation financially ruinous.

Document the termination basis, follow the contractual notice period, ensure IP
hand-off is clean, and protect your team's wellbeing above retention revenue.

---

## References & Further Reading

- *Project Management Institute — PMBOK Guide (7th Ed.)*
- *The Business of Software* — Michael Cusumano
- SFIA Framework — skills-based role definitions for RACI
- OWASP Top 10 — security requirements baseline for SOW appendix
- ISO/IEC 25010 — software quality model for acceptance criteria framing
