---
title: Engineering Ladder & Hiring
category: software-house-ops
summary: >
  A comprehensive guide to engineering career ladders and the full hiring lifecycle —
  covering IC vs management tracks, levelling rubrics, competency matrices, hiring pipeline
  design, structured interviews, onboarding, retention, and employer branding.
sources: web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Engineering Ladder & Hiring

## Why Career Ladders Matter

Without a clear career ladder, engineers don't know what "good" looks like or how to grow.
Promotions become political and arbitrary. High performers leave when they hit invisible
ceilings. A well-designed ladder gives engineers a map, gives managers a shared language for
feedback, and gives the organisation a consistent way to calibrate compensation fairly.

---

## IC Track vs Management Track

Most mature engineering organisations offer two distinct career paths:

### Individual Contributor (IC) Track
For engineers who want to grow through technical depth and scope — not by managing people.

```
Junior Engineer → Engineer → Senior Engineer → Staff Engineer → Principal Engineer → Distinguished Engineer
```

### Engineering Management Track
For engineers who want to grow through people leadership and organisational impact.

```
Senior Engineer → Engineering Manager → Senior Manager → Director → VP of Engineering → CTO
```

**Key principle:** The management track should not be the *only* path to seniority or higher
compensation. A Principal Engineer should be as respected and compensated as a Director of
Engineering. Organisations that fail to enforce this systematically drain their best
technical talent into management roles they don't want.

---

## IC Level Definitions

### Junior Engineer (L3 / E3)
- Executes well-defined tasks with guidance.
- Scope: individual features or bug fixes.
- Needs frequent code review and direction.
- Learning the team's tools, practices, and codebase.
- Typical experience: 0–2 years.

### Engineer (L4 / E4)
- Independently delivers features of moderate complexity.
- Scope: a feature end-to-end within one service.
- Proactively identifies bugs and technical debt.
- Gives useful code review feedback to peers.
- Typical experience: 2–5 years.

### Senior Engineer (L5 / E5)
- Delivers complex features with broad technical impact.
- Scope: cross-service features; significant architectural decisions within a team.
- Mentors junior engineers; improves team processes.
- Technical depth in ≥ 1 domain; productive breadth across the stack.
- Typical experience: 5–8 years (but not time-based).

### Staff Engineer (L6 / E6)
- Technical leader across multiple teams or a significant product area.
- Scope: sets technical direction for a squad/tribe; drives cross-team initiatives.
- Translates business strategy into technical strategy.
- Identifies and mitigates systemic risks before they become incidents.
- "Multiplier" — their impact is felt through others, not just their own code.

### Principal Engineer (L7 / E7)
- Company-wide technical authority.
- Scope: architectural decisions that affect the entire engineering organisation.
- Represents engineering in exec-level discussions.
- Produces artefacts (RFCs, ADRs, standards) that shape how hundreds of engineers work.
- Rare: typically 1–3 per organisation of 200 engineers.

---

## Competency Matrix

A competency matrix defines *what* is evaluated at each level across key dimensions.
Example dimensions and brief descriptors:

| Dimension | Junior | Engineer | Senior | Staff |
|-----------|--------|----------|--------|-------|
| **Technical Skill** | Applies known patterns | Selects appropriate patterns | Evaluates trade-offs independently | Defines patterns for org |
| **Delivery** | Completes tasks on time with guidance | Owns feature delivery | Owns complex cross-service delivery | Drives multi-team programs |
| **Code Quality** | Writes readable code with review | Consistently clean code; good tests | High-quality across services; reviews others | Sets quality standards org-wide |
| **Communication** | Communicates progress to manager | Clear written/verbal comms | Communicates technical concepts to non-engineers | Influences exec strategy via written proposals |
| **Mentorship** | Seeks mentorship | Informally helps peers | Actively mentors 1–2 juniors | Coaches senior engineers |
| **Scope & Impact** | Task-level | Feature-level | Team-level | Org-level |

Publish the competency matrix publicly (internally). Ambiguity about expectations breeds
resentment and attrition.

---

## Levelling Rubrics & Calibration

- **Calibration sessions** — quarterly cross-manager meetings where all engineers under review
  are discussed against the same matrix. Prevents individual manager bias.
- **Evidence-based levelling** — promotions require concrete examples for each dimension, not
  just manager assertion. Engineers maintain a **brag document** (running record of
  accomplishments, impact, and feedback).
- **Time-in-level guidance** — while promotions are not time-based, set *typical* ranges so
  engineers have pacing expectations. E.g., Junior → Engineer: 12–24 months.
- **Promotion vs. performance review** — separate these conversations. A performance review
  covers the past; a promotion discussion covers readiness for future scope.

---

## Career Conversations

1:1 career conversations should happen **at least quarterly**. Structure:

- **Where are you now?** Review the competency matrix together. Where is the engineer
  demonstrably at level? Where are the gaps?
- **Where do you want to go?** Aspirations: IC depth, management, specific domain expertise?
- **What's the plan?** Identify 1–2 specific growth areas and concrete opportunities
  (project ownership, conference talk, mentoring assignment).
- **What do you need from me?** Remove blockers; connect engineers with sponsors and mentors.

Document agreed growth areas in writing after each conversation.

---

## Hiring Pipeline

### 1. Sourcing

- **Employee referrals** — highest-quality channel; convert at 2–3× the rate of cold
  applicants. Run a structured referral programme with meaningful bonuses.
- **Job boards** — LinkedIn, Indeed, Stack Overflow Jobs, Wellfound (AngelList for startups).
- **GitHub / open source** — engage contributors to relevant OSS projects.
- **Community presence** — meetups, conferences, Discord/Slack communities, tech blogs.
- **Recruiting agencies** — last resort; expensive (15–25% of first-year salary). Use for
  hard-to-fill Staff+ roles or urgent hiring.

### 2. Screening (Résumé Review)

- Use a **scorecard** aligned to role requirements — evaluate every candidate against the same
  criteria to reduce unconscious bias.
- Screen for: relevant experience, trajectory (growth over time), specific skills in JD.
- **Anonymised screening** — remove names, universities, and graduation years to reduce bias.
- First-pass screen by recruiter; second pass by hiring manager.

### 3. Recruiter / Hiring Manager Phone Screen (30 min)

- Confirm basic qualifications, compensation expectations, and availability.
- Explain the role and the hiring process.
- Assess communication and enthusiasm.
- Go / no-go decision before investing engineering time.

### 4. Technical Screening (60–90 min)

Options:

#### Take-Home Assignment
- **Pros:** Candidates work at their own pace; no performance anxiety; reveals real workflow.
- **Cons:** Imposes significant time burden; unfair to candidates with caretaking
  responsibilities; risk of outsourcing.
- **Best practices:** Limit to 2–3 hours of actual work; pay candidates for their time (even
  a token amount signals respect); evaluate the process as much as the output.

#### Live Coding Interview
- **Pros:** Fast; observe thought process in real time; lower total time commitment.
- **Cons:** Performance anxiety disadvantages many qualified engineers; poor signal for
  experienced engineers.
- **Best practices:** Use collaborative tools (CodePair, CoderPad); allow internet access;
  prefer practical problems over algorithmic puzzles.

#### Portfolio Review
- For senior candidates: review existing open-source work, a system they built, or a design
  doc they've written. High-signal; zero extra time burden on the candidate.

### 5. Technical Interview (90 min)

- **System design** (for Senior+): "Design a URL shortener" / "Design a notification service."
  Evaluate: requirements gathering, trade-off analysis, scalability thinking, communication.
- **Code review exercise**: Give the candidate a PR diff with real bugs. Evaluate: what they
  catch, how they frame feedback.
- **Domain-specific depth**: 30–45 min deep-dive into their area of expertise.

### 6. Culture / Values Interview (45 min)

- Assess alignment with team values, not "culture fit" (which is a proxy for bias).
- Use **behavioural questions** (STAR format: Situation, Task, Action, Result):
  - "Tell me about a time you disagreed with a technical decision. How did you handle it?"
  - "Describe a project that failed. What was your role? What did you learn?"
  - "Tell me about a time you mentored someone. What was the outcome?"

### 7. Hiring Committee / Debrief

- All interviewers complete their **scorecards independently** before the debrief — prevents
  anchoring bias from the first voice in the room.
- Structured debrief: each interviewer summarises signal; discuss disagreements; make a
  decision against the scorecard, not gut feel.
- Document the decision and key reasoning — essential for candidate feedback and future
  calibration.

---

## Structured Interviews & Scorecards

A scorecard for each interview stage defines:
- The **competencies** being assessed (e.g., "system design thinking," "debugging skill").
- The **rating scale** (e.g., 1–4: Strong No Hire / No Hire / Hire / Strong Hire).
- **Example evidence** for each rating.

Sample scorecard entry:
```
Competency: Communication of Technical Concepts
4 - Strong Hire: Explained complex design clearly to a non-expert; adapted explanation when
    questioned; used diagrams effectively.
3 - Hire: Explained the design with minor gaps; responded well to clarifying questions.
2 - No Hire: Explanation was unclear; struggled to answer follow-up questions.
1 - Strong No Hire: Unable to communicate the design or reasoning.
```

---

## Offer Negotiation

- **Move quickly** — a strong candidate receiving a slow offer process interprets it as
  lack of interest and moves to a competitor.
- **Make the first offer strong** — low-ball offers damage trust; many candidates disengage
  rather than negotiate.
- **Compensation bands** — every level should have a transparent salary band. Candidates who
  ask for a number deserve one.
- **Equity explanation** — ensure candidates understand the value and vesting schedule of any
  equity component. Many candidates incorrectly discount equity.
- **Competing offers** — if a candidate has competing offers, engage transparently. Ask what
  would make your offer the right choice; address the gap if possible.

---

## Onboarding: 30/60/90 Day Plan

### Day 1–30 (Orient)
- Complete account provisioning, tool setup, security training.
- Read architecture primer; meet the team.
- Pair with a buddy (peer engineer who owns onboarding support).
- Complete first small PR (a real task, not a "hello world" ticket).
- 1:1 with manager: goals, expectations, how you like to work.

### Day 31–60 (Contribute)
- Independently own a feature end-to-end (scoped for the level).
- Participate in code reviews — giving and receiving.
- Shadow an on-call rotation shift.
- Identify one improvement to the local dev setup or documentation.

### Day 61–90 (Accelerate)
- Deliver a feature with minimal guidance.
- Present a technical decision or retrospective to the team.
- Have a 90-day review with the manager: what's going well, what needs adjustment.
- Set 6-month goals against the competency matrix.

---

## Retention Strategies

The cost of replacing an engineer (recruiting, onboarding, productivity ramp) is typically
**50–200% of annual salary**. Retention is a financial priority.

- **Clear growth paths** — engineers who can't see their next level leave for companies
  where they can.
- **Competitive compensation** — benchmark salaries annually (Levels.fyi, Radford, Mercer).
  Underpaying signals that talent is not valued.
- **Autonomy** — engineers want ownership over technical decisions. Excessive top-down
  mandates drive Senior+ engineers out.
- **Interesting problems** — engineers accept lower compensation for technically challenging
  work. Deliberately assign engineers to problems that stretch their skills.
- **Psychological safety** — engineers leave managers, not companies. Train managers; act on
  engagement survey results.
- **Flexible work** — remote/hybrid flexibility is now a retention baseline, not a perk.
- **Exit interviews** — conduct structured exit interviews; analyse themes; act on findings.

---

## Employer Branding

- **Engineering blog** — publish technical content written by engineers. Top engineers read
  engineering blogs before applying.
- **Open source contributions** — release internal tools; contribute to OSS. Demonstrates
  engineering quality and attracts aligned candidates.
- **Conference talks** — speaking at QCon, Strange Loop, local meetups builds name recognition
  in the talent pool.
- **Glassdoor / Blind** — monitor and respond to reviews. Negative reviews about process or
  management surface in candidate research.
- **Transparent hiring process** — publish the interview process on the careers page.
  Candidates trust organisations that demystify the process.

---

## Further Reading

- *An Elegant Puzzle: Systems of Engineering Management* — Will Larson (Stripe Press, 2019)
- *The Staff Engineer's Path* — Tanya Reilly (O'Reilly, 2022)
- *Radical Candor* — Kim Scott (career conversations)
- Levels.fyi — compensation benchmarking
- Engineering Ladders open-source project — github.com/jorgef/engineeringladders
- Progression.fyi — public engineering ladders from tech companies
