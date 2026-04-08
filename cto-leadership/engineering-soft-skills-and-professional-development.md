---
title: "Engineering Soft Skills and Professional Development"
tags: [soft-skills, negotiation, communication, writing, public-speaking, senior-engineer]
category: cto-leadership
sources:
  - "Knowledge-Sharing Architects As An Alternative to Coding Architects"
  - "Ten Rules for Negotiating a Job Offer - Haseeb Qureshi"
  - "Salary Negotiation - Patrick McKenzie (patio11)"
  - "Undervalued Software Engineering Skills: Writing Well - Gergely Orosz"
  - "The Pyramid Principle (McKinsey)"
  - "Senior Engineer's Checklist"
  - "How to Prepare a Talk - Deconstruct Conf"
  - "Falsehoods Programmers Believe About Names"
last_updated: 2026-04-08
---

# Engineering Soft Skills and Professional Development

## Writing Well (The Most Undervalued Skill)

### Why Writing Matters for Engineers (Gergely Orosz)

At every level above mid-career, **writing becomes more important than coding:**

| Level | % Time Writing | What You Write |
|-------|---------------|----------------|
| Junior | 10% | Code comments, ticket descriptions |
| Mid | 20% | Design docs, PR descriptions |
| Senior | 40% | RFCs, architecture proposals, incident reports |
| Staff+ | 60%+ | Strategy docs, cross-org communications, specs |

### The Pyramid Principle (McKinsey)

Structure all written communication top-down:

```
         Main Point (answer first)
        /          |           \
   Support 1   Support 2   Support 3
   /    \       /    \       /    \
 Detail Detail Detail Detail Detail Detail
```

**Rules:**
1. **Start with the answer** — not the analysis that led to it
2. **Group and summarize** — each level summarizes the level below
3. **Logically order** — time, structure, or importance
4. **MECE** — Mutually Exclusive, Collectively Exhaustive

**Example:**
- ❌ "We analyzed three options. Option A has these pros and cons... Option B... Option C... We recommend Option B."
- ✅ "We recommend Option B (migrate to PostgreSQL). Here's why, and why the alternatives don't work."

### Writing Tips for Engineers

1. **One idea per paragraph** — if a paragraph covers two topics, split it
2. **Active voice** — "The service processes requests" not "Requests are processed by the service"
3. **Concrete > abstract** — "Response time increased from 200ms to 2s" not "Performance degraded significantly"
4. **Cut 50%** — your first draft is always too long
5. **Read it aloud** — if it sounds awkward spoken, it reads awkward too

## Salary Negotiation

### Haseeb Qureshi's Ten Rules

1. **Get everything in writing** — verbal offers are worthless
2. **Always negotiate** — the initial offer is never the best they can do
3. **Have alternatives** — the best negotiation leverage is another offer
4. **Give reasons** — "I'd like $X because..." not just "I want more"
5. **Be enthusiastic** — show you want the job, negotiate the terms
6. **Don't reveal your current salary** — it anchors the negotiation down
7. **Don't give a number first** — let them anchor, then counter
8. **Negotiate the whole package** — base, equity, signing bonus, title, start date, remote
9. **Be willing to walk away** — not as a tactic, as a genuine option
10. **Be kind** — aggressive negotiation burns bridges you might need

### Patrick McKenzie (patio11) — The Classic

> "Salary negotiation is the highest ROI activity most people will ever engage in."

**Key insights:**
- Companies have **salary bands** — your job is to find the top of the band
- HR is **not your friend** in this process — they're optimizing for the company
- **Equity is complicated** — apply heavy discounts (see startup finance section)
- **Title matters** — it affects your next negotiation, not just ego
- **Everything is negotiable** — PTO, equipment, conference budget, remote policy

### What CTOs Should Know About Compensation

When you're the one making offers:
- **Pay at the top of band** for critical hires — underpaying costs more in turnover
- **Be transparent about bands** — reduces negotiation anxiety and bias
- **Equity should vest over 4 years** with a 1-year cliff (standard)
- **Refresh grants** — annual equity refresh prevents tenure-based compensation decay
- **Don't penalize non-negotiators** — women and minorities negotiate less, creating pay gaps

## The Senior Engineer's Checklist

25 behaviors that distinguish senior from mid-level engineers:

### Technical
- [ ] Understand the business context of your work
- [ ] Can explain technical decisions to non-technical stakeholders
- [ ] Consider operational aspects (monitoring, alerting, rollback) during design
- [ ] Write code that others can understand and maintain
- [ ] Know when NOT to optimize

### Process
- [ ] Write clear, actionable ticket descriptions
- [ ] Break large tasks into reviewable chunks
- [ ] Estimate with historical accuracy (track your estimates)
- [ ] Document decisions and their rationale (ADRs)
- [ ] Review code constructively (suggest, don't dictate)

### People
- [ ] Mentor junior developers without condescension
- [ ] Give feedback that's specific, actionable, and kind
- [ ] Share knowledge proactively (brown bags, docs, pairing)
- [ ] Manage up — communicate status, risks, and blockers before being asked
- [ ] Say "I don't know" without shame

### Organizational
- [ ] Identify and reduce key-person dependencies (including yourself)
- [ ] Think about team velocity, not just personal productivity
- [ ] Balance new features with maintenance and tech debt
- [ ] Advocate for process improvements with data
- [ ] Build relationships across teams

## Public Speaking for Engineers

### Preparing a Talk (Deconstruct Conf)

**The process:**

1. **Start with one idea** — what's the ONE thing the audience should remember?
2. **Know your audience** — conference attendees ≠ team meeting ≠ executive briefing
3. **Outline before slides** — structure the narrative, then add visuals
4. **Practice out loud** — 3x minimum before presenting
5. **Time it** — most talks run 20% longer than practiced

**Structure for a 30-minute conference talk:**
```
0:00 - Hook (personal story or surprising fact)
0:02 - The problem (why should anyone care?)
0:05 - Your approach (what you tried)
0:12 - The details (technical meat)
0:22 - Results and lessons
0:26 - Key takeaways (repeat the one idea)
0:28 - Q&A
```

**Common mistakes:**
- Too much code on slides (nobody reads it)
- No story arc (just a list of facts)
- Apologizing ("this might be boring" — then why present it?)
- Not enough practice (the #1 mistake by far)

## Falsehoods Programmers Believe

### About Names (Patrick McKenzie)

40+ false assumptions, including:
- People have exactly one name ❌
- People have at most one family name ❌
- Names are unique ❌
- Names don't change ❌
- Names are written in ASCII ❌
- Names have a maximum length ❌
- People have names ❌ (yes, some legal systems allow no name)

**Why this matters:** Every assumption about user data is wrong somewhere. Design systems that are **permissive**, not prescriptive.

### The General Principle

Whenever you think "surely all X have Y," you're wrong. This applies to:
- Addresses (not all countries have zip codes, street numbers, or even streets)
- Time (not all days have 24 hours, not all minutes have 60 seconds)
- Currency (precision varies, some currencies have 3 decimal places)
- Phone numbers (variable length, different formatting rules per country)
- Gender (not binary, not static)

**Engineering principle:** Always validate assumptions about the real world. If your schema enforces a constraint, make sure that constraint is actually universal.

## Knowledge-Sharing Architecture

### The Alternative to Coding Architects

Traditional architect: designs systems, doesn't code, becomes disconnected from reality.

**Knowledge-sharing architect:**
- Spends 50% of time **teaching**, not designing
- Creates **reference implementations**, not abstract diagrams
- Reviews code to **transfer knowledge**, not enforce standards
- Writes **decision records**, not decrees
- Rotates through teams to **spread expertise**, not gatekeep

**Key behaviors:**
1. Write ADRs (Architecture Decision Records) for every significant choice
2. Run regular "architecture clinics" (team brings problems, architect facilitates solutions)
3. Create coding dojos and katas for new patterns
4. Pair program with developers implementing architectural patterns
5. Maintain a "tech radar" that the whole team contributes to

## Key Takeaways

1. **Writing is the most undervalued engineering skill** — invest in it at every level
2. **Pyramid Principle: answer first**, then support — don't bury the lead
3. **Always negotiate compensation** — it's the highest ROI conversation of your career
4. **Senior engineering is more about people and process** than technical brilliance
5. **Practice talks 3x minimum** — under-rehearsal is the #1 speaking mistake
6. **Question all assumptions about user data** — the real world is messier than your schema
7. **Knowledge-sharing > gatekeeping** — the best architects teach, not decree
