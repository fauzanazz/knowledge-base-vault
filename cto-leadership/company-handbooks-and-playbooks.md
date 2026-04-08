---
title: "Company Handbooks and Engineering Playbooks"
tags: [handbooks, playbooks, company-culture, remote-work, values, atlassian, valve, basecamp, gitlab, hashicorp]
category: cto-leadership
sources:
  - "Atlassian Team Playbook"
  - "Valve Employee Handbook"
  - "Basecamp Employee Handbook"
  - "GitLab Team Handbook"
  - "How HashiCorp Works"
last_updated: 2026-04-08
---

# Company Handbooks and Engineering Playbooks

## Why Handbooks Matter

A handbook is **your company's operating system documented**. It:
- Reduces "how do we do X?" questions
- Creates consistency across teams and offices
- Enables asynchronous decision-making
- Onboards new hires faster
- Forces clarity about culture and values

> "If it's not written down, it's not a rule — it's a preference." — GitLab

## Notable Company Handbooks

### Valve Employee Handbook

**Key philosophy:** Flat hierarchy, self-directed work.

**Standout concepts:**
- **No managers** — employees choose what to work on
- **Desks on wheels** — literally move to join a project
- **Hiring is the most important thing you do** — T-shaped people (broad knowledge + deep expertise)
- **Stack ranking for compensation** — peers rank each other (controversial but transparent)

**Lessons for CTOs:**
- Works only with exceptional hiring bar
- Doesn't scale past ~400 people without adaptation
- The "no manager" model still has informal hierarchy — acknowledge it
- Creative freedom requires extreme trust and accountability

**Caution:** Valve's model has been criticized for:
- Cliques forming around popular projects
- Important but "boring" work getting neglected
- Implicit power structures that are harder to address than explicit ones

### Basecamp Employee Handbook

**Key philosophy:** Calm work, sustainable pace.

**Standout concepts:**
- **No venture capital** — profitability from day one
- **6-week cycles** (Shape Up methodology)
- **No goals** — they've explicitly rejected OKRs and KPIs
- **4-day work weeks in summer**
- **No real-time chat for work** — asynchronous by default
- **Company-paid vacations** and sabbaticals
- **No tracking** — trust people, measure output

**Lessons for CTOs:**
- Sustainable pace > heroic sprints
- Async-first communication reduces interruptions
- Profitable bootstrapped companies have different dynamics than VC-funded ones
- "Calm" doesn't mean "slow" — it means intentional

### GitLab Team Handbook

**Key philosophy:** Radical transparency, all-remote.

**By the numbers:**
- 2,000+ pages, publicly available
- Covers EVERYTHING: values, hiring, security, benefits, decision-making, meeting norms
- Updated hundreds of times per month

**Standout concepts:**
- **Handbook-first** — if it's not in the handbook, it doesn't exist
- **All-remote** — no offices, 65+ countries
- **Values: CREDIT** — Collaboration, Results, Efficiency, Diversity, Iteration, Transparency
- **Low-context communication** — assume the reader has no context
- **No DMs for work** — everything in public channels
- **Boring solutions** — prefer simple, proven approaches
- **Two-way door decisions** — most decisions are reversible, so decide fast

**Lessons for CTOs:**
- Documentation is a competitive advantage
- Remote-first requires explicit communication norms
- Public handbooks attract talent aligned with your values
- "Boring solutions" prevents over-engineering

### Atlassian Team Playbook

**Key philosophy:** Structured practices for team health.

**Standout plays:**

| Play | Purpose | When to Use |
|------|---------|------------|
| **Health Monitor** | Assess team health across 8 dimensions | Quarterly |
| **Project Kickoff** | Align on goals, roles, and risks | Project start |
| **Retrospective** | Reflect and improve | End of sprint/project |
| **DACI** | Decision-making framework (Driver, Approver, Contributors, Informed) | Any decision |
| **Pre-mortem** | Imagine failure, prevent it | Before major launches |
| **Empathy Map** | Understand user feelings and motivations | Product discovery |
| **Sparring** | Get feedback on work-in-progress | During design/planning |

**DACI Framework (Decision-Making):**
```
D - Driver: Owns the process, gathers input, drives to decision
A - Approver: Has final say (usually one person)
C - Contributors: Provide input and expertise
I - Informed: Told after the decision is made
```

**Health Monitor dimensions:**
1. Shared understanding (of mission and goals)
2. Value and metrics (do we know what success looks like?)
3. Proof of concept (have we validated assumptions?)
4. Team cohesion (do we trust each other?)
5. Full-time owner (is someone accountable?)
6. Balanced team (right skills represented?)
7. Managed dependencies (external blockers identified?)
8. Appropriate velocity (are we going fast enough?)

### HashiCorp — How HashiCorp Works

**Key philosophy:** Writing culture, principled architecture.

**Standout concepts:**
- **Writing culture** — proposals, RFCs, and decisions are written, not discussed in meetings
- **Principles over rules** — teach thinking, not just processes
- **Open source core + enterprise** — business model reflected in engineering culture
- **"The Tao of HashiCorp"** — explicit technology philosophy document:
  - Workflows, not technologies
  - Simple, modular, composable
  - Communicate via APIs
  - DevOps is not a team

## Building Your Own Handbook

### Minimum Viable Handbook

Start with these sections:

```
1. Mission & Values
   - Why we exist
   - What we believe in
   - How we behave

2. How We Work
   - Communication norms (sync vs async, tools)
   - Meeting culture (types, frequency, norms)
   - Decision-making framework (DACI or similar)
   - Work hours and flexibility

3. Engineering Practices
   - Code review expectations
   - Testing requirements
   - Deployment process
   - On-call rotation
   - Tech stack decisions

4. People
   - Career ladder / growth framework
   - Performance review process
   - Feedback culture
   - Hiring process

5. Operations
   - Onboarding checklist
   - Offboarding checklist
   - Tools and access
   - Security practices
```

### Handbook Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| Written once, never updated | Becomes fiction | Assign owners, review quarterly |
| Too long, too detailed | Nobody reads it | Concise, link to details |
| Only HR content | Engineers ignore it | Include engineering practices |
| Top-down only | Feels imposed | Allow contributions via PR |
| Private/locked | Can't reference in conversations | Make it accessible (ideally public) |

### The GitLab Approach to Handbook Maintenance

1. **Anyone can edit** — via merge request
2. **DRI (Directly Responsible Individual)** for each section
3. **Regular audits** — stale content is flagged and updated or removed
4. **"Handbook first"** — if someone asks a question, answer it AND update the handbook
5. **No meetings without a handbook reference** — if the topic isn't documented, document it first

## Key Takeaways

1. **A handbook is your company's operating system** — invest in it early
2. **Handbook-first culture** prevents tribal knowledge and scales communication
3. **Valve's flat model** is aspirational but requires exceptional hiring and breaks at scale
4. **Basecamp's calm approach** proves sustainable pace beats heroic sprints
5. **GitLab's radical transparency** shows that public documentation attracts aligned talent
6. **DACI framework** solves "who decides?" — the most common organizational dysfunction
7. **Start minimal, iterate** — a living 10-page handbook beats a dead 200-page handbook
