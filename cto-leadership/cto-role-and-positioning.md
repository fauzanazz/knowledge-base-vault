---
title: "CTO Role and Positioning: Archetypes, Frameworks, and First 90 Days"
tags: [cto, leadership, engineering-leadership, vp-engineering, org-design, career, executive]
category: cto-leadership
---

# CTO Role and Positioning: Archetypes, Frameworks, and First 90 Days

> *"Hell if I know."* — Nathan Myhrvold, former Microsoft CTO, when asked to define his role

The Chief Technology Officer title is one of the most ambiguous in the industry. Unlike a CFO (who always owns finance) or a CPO (who always owns product), the CTO role changes shape depending on company stage, business model, and founder background. This article synthesizes perspectives from Werner Vogels (Amazon CTO), Fred Wilson (Union Square Ventures), Greg Brockman (Stripe CTO), Will Larson (author of *An Elegant Puzzle*), Lara Hogan, and Miguel Carranza (RevenueCat founder CTO) into a practical reference for engineers moving into the role.

---

## What Is a CTO?

The CTO title emerged in the late 1980s, when R&D lab directors were elevated to the executive team to translate technical capability into strategic advantage. From the start, the role was defined not by a fixed job description but by the **relationship between technology and business value** at a given company.

Werner Vogels's framing is the most durable: the CTO's job is to worry about technology in the future. *"If you want to have a great future, you have to start thinking about it in the present — because when the future's here, you won't have the time."* This forward-orientation distinguishes the CTO from every other technical role. Engineers solve today's problems. The CTO bets on tomorrow's.

Three properties are nearly universal across all CTO archetypes:

1. **Technical credibility** — The CTO must be respected as the strongest technologist at the executive level, even if not the best coder on the team.
2. **External orientation** — Customer relationships, ecosystem positioning, and market foresight are core functions, not side duties.
3. **Influence without direct authority** — Except in the smallest companies, the CTO typically does not manage all of engineering. They lead through vision, trust, and architecture rather than org chart power.

What the CTO does *not* own (at scale) is the day-to-day operational health of the engineering team — that belongs to the VP Engineering.

---

## CTO vs. VP Engineering: The Core Distinction

Fred Wilson's framing is the cleanest: *"The CTO makes sure the technical approach is correct and the VP Engineering makes sure the team is correct. They are yin and yang."*

Werner Vogels sharpened it to a single contrast: *"A VP of Engineering wakes up each morning concerned whether they have the absolutely best engineering team. The CTO wakes up concerned whether they have the absolutely best technology."*

| Dimension | VP Engineering | CTO |
|---|---|---|
| **Primary orientation** | People and process | Technology and vision |
| **Success metric** | Team health, velocity, retention | Technical correctness, strategic bets |
| **Authority type** | Organizational (by role) | Influential (by credibility) |
| **Time horizon** | Now → 6 months | 1 year → 5 years |
| **Typical output** | Shipped product, healthy team | Architecture decisions, external trust |
| **Sales involvement** | Rarely | Frequently (closes strategic deals) |
| **Personality** | Process/management orientation | Visionary, tinkerer, researcher |

Brad Feld's empirical rule: **one person can hold both roles until approximately 20 people**. Beyond that threshold, the person either becomes ineffective at both, defaults into the VP Eng role (because it's more urgent), or consciously hands one side off. The split is not about seniority — these are often peer roles. A VP Eng can report to a CTO; a CTO can report to a VP Eng. What matters is clarity about which person owns which problems.

### Who to Hire First?

There is a genuine debate:

- **Pro-CTO first (Fred Wilson):** Early on you don't need a manager of people and systems you don't have yet. You need an architect-implementor — someone who can build the technical foundation and make it credible to customers and investors.
- **Pro-VP Eng first (Erik Schwartz):** Early on you'll make much more progress with a VPE. CTOs tend to try to launch perfection. You'll get to users faster with someone focused on shipping.

The practical answer depends on whether your bottleneck is *technical credibility* (needs CTO) or *execution throughput* (needs VP Eng). Most technical co-founders who carry both roles will eventually evolve into one or the other — and then hire to fill the gap.

---

## CTO Archetypes

Tom Berray's four-model framework (cited by Vogels) provides the clearest taxonomy, organized by where technology sits in the business:

### Archetype 1: Infrastructure Manager
**Context:** Traditional businesses where technology is a pure support function (e.g., retail, manufacturing, financial services with legacy systems).

The CTO owns operational infrastructure: data centers, networks, application maintenance, security. The CIO retains ownership of *how* technology is used strategically. This is the least externally visible CTO role, and the one most junior engineers would not recognize as "the CTO job."

**Typical scope:** 500–1,000+ engineers in a direct line.

### Archetype 2: Technology Visionary & Operations Manager
**Context:** Tech-oriented companies and dot-coms where technology *is* the business strategy.

The CTO both determines how technology implements business strategy (visionary) and integrates and runs it (operations manager). This is often the technical co-founder or first engineering hire. Greg Brockman at Stripe and Miguel Carranza at RevenueCat both started here — IC plus cultural architect plus eventual executive.

**Typical scope:** Full ownership of engineering, often both CTO and VP Eng titles in one person early on.

### Archetype 3: External-Facing Technologist
**Context:** Companies selling technology products or APIs to customers and partners — platform businesses, enterprise software, cloud infrastructure.

This CTO acts as the primary technical interface with customers, partners, and the market. They influence the product portfolio, instill customer confidence, and close strategic deals. Werner Vogels himself functions largely in this archetype — public-facing, conference-speaking, customer-trusting. Some larger companies use multiple CTOs in this model (e.g., a per-division or per-product CTO).

**Typical scope:** Small team of 10–50 engineers functioning as an incubator; influence over the broader org without direct management.

### Archetype 4: Big Thinker / Internal Disruptor
**Context:** Large companies where technology can open new business models or preempt competitive disruption.

The CTO evaluates how technology can create new business lines internally, runs a prototyping lab, assesses competitive threats, sets architecture standards, manages external partnerships. This is R&D leadership rather than operational management.

**Typical scope:** Small advanced technology group; reports directly to CEO to guarantee cross-organizational influence.

### The Structural Divide

Archetypes 1 and 2 have the CTO managing a large engineering division directly. Archetypes 3 and 4 have the CTO **influencing** other divisions rather than directing them — which is why reporting to the CEO (rather than burying the role in an org hierarchy) is essential for the latter two to function.

---

## The Founder CTO Trajectory

Miguel Carranza's year-by-year account of RevenueCat's scaling adds a dimension the taxonomy frameworks miss: **how a single person's job changes as a company grows from 5 to 45 people**.

### Stage 0 → ~8 Engineers: Pure IC
The founder CTO is writing most of the code, making all architecture decisions, onboarding engineers manually, and handling technical customer conversations. Processes are minimal by design. Technical debt is acceptable — finding product-market fit takes priority. As Carranza puts it: *"Technical debt is actually good if taken responsibly while trying to find product-market fit."*

**Breaking point signals:** You are on-call 24/7 and carry your laptop everywhere. Onboarding is inconsistent. You are the single point of failure for every system.

### Stage ~8–20 Engineers: The Bottleneck Transition
This is the hardest phase — what Carranza calls "no man's land." The founder CTO becomes a bottleneck for shipping. They can no longer write code on time while also doing 1:1s, sprint planning, recruiting, and board prep. The common failure mode is trying to do all of it at below-par quality.

The correct move: **stop coding and delegate**. This feels deeply unnatural. *"Not coding felt really weird at first. It totally felt like I was not doing my job."* It isn't. Delegation unblocks projects and creates stronger ownership in engineers. The CTO's job becomes architectural guidance, knowledge sharing, and identifying who should own what.

**Breaking point signals:** You are blocking PRs. Engineers are waiting on you for decisions. You haven't shipped code in a sprint because you were in meetings.

### Stage ~20–50 Engineers: Org Design and Hiring
The dominant challenge shifts from technical to organizational. Hiring pipelines that worked on referrals break down. Time zones multiply coordination costs. Junior engineers need onboarding infrastructure. The CTO now spends significant time on team composition, career ladders, diversity initiatives, and working with a Head of People.

Carranza's framing is important for founder CTOs who feel identity anxiety about this shift: *"My founder identity should be more important than my CTO title. In the long run, as a major shareholder, I need to do whatever is best for the company."* The dopamine hits from coding must give way to the satisfaction of multiplied output through others.

---

## First 90 Days Playbook

Whether you are hired externally or promoted internally, the first 90 days as a CTO or VP Eng follow a consistent pattern across Will Larson, Lara Hogan, and Greg Brockman's experiences.

### The Foundational Rule: No Major Changes in the First 30 Days

Lara Hogan's core warning applies to every senior leader: *"No matter how well-intentioned you are, enacting change within your first 30 days could jeopardize your trust and standing."* The impulse to make a big move — to prove you were the right hire, to demonstrate momentum, to show responsiveness to complaints — is wrong. Trust is built by listening, not by acting.

The only exception: a genuine safety emergency.

### Days 1–30: Sponge Mode (Listening Tour)

**Your only job is to absorb.** Schedule 30-minute 1:1s with everyone on the team, all cross-functional peers, and key stakeholders. Use a structured format:

1. *"When you think about the team as a whole, what change do you most want to see?"*
2. *"What's working so well we shouldn't change it?"*

Follow up with: *"What's the risk if we don't make that change?"* and *"Who else should I talk to?"*

Include external listening: shadow customer support tickets, sit in on sales calls, find the data warehouse and query it yourself. Will Larson's principle: customers are a "reality reservoir" — they give you ground truth that internal narratives have smoothed over.

**What to avoid:**
- Making judgments before you have context (*"This codebase is terrible, what were they thinking?"*)
- "At my last job, we did it this way" — signals you haven't understood the new environment yet
- Rushing to make changes because you feel unproductive

### Days 30–60: Synthesize and Align

At the end of 30 days, share what you've learned — but do this carefully. Identify 1–2 overarching themes that emerged from your listening tour. Keep them **future-focused** (where we're going, not what went wrong in the past). Present them at an all-hands. The goal is to make people feel heard, not to announce changes.

Will Larson's seven discovery areas to have mapped by this point:

| Area | What to Have Understood |
|---|---|
| **Business model** | Where money comes in and goes out; runway; what creates step-function value |
| **Culture** | Real values (vs. stated), actual decision-making processes, who is rewarded and why |
| **Peer relationships** | What each stakeholder needs from engineering; relationship health before first conflict |
| **Execution** | How ideas become shipped work; how emergencies are handled |
| **Technical quality** | What technical limitations block "impossible" projects; key quality properties |
| **Team health** | Who thrives and who doesn't; inclusion gaps; energy sources and drains |
| **Pace** | What you personally need to stay sustainable long-term |

### Days 60–90: Begin Durable System Changes

Tactical fixes create the appearance of improvement. Durable improvements come from **systems** that change behavior over time. Greg Brockman's experience at Stripe illustrates the pitfall: delegation without a feedback mechanism means you lose situational awareness even as the org scales.

Brockman identified four mechanisms for staying informed without managing everything directly:
1. **Doing the work** — coding, prototyping, writing specs (most direct, hardest to scale)
2. **Talking to people** — skip-levels, informal conversations, aggregate sentiment
3. **Observing the work** — reading all diffs (he tried this for a month; found it ineffective — *"most of the work was reconstructing the author's state of mind"*)
4. **Planning the work** — being in the room where roadmaps are made

The best combination is (2) + (4): maintain meaningful 1:1 relationships and participate in planning, rather than monitoring output directly.

### Structural Setup for Success

**External support system:** Build a peer network outside your company (private Slack group, peer CTO breakfast). No one at your company is in your role. You need peers who understand your context without competing interests.

**Executive coach:** Will Larson recommends getting one immediately. They have seen dozens of people through the exact challenges you're facing and can accelerate your calibration dramatically.

**Operating rhythm with co-leaders:** Greg Brockman and VP Eng Marc Hedlund ran two hour-long 1:1s per week (Monday + Friday). Over time, they built enough shared context to serve as reliable "mental simulators" of each other — each could predict how the other would respond. Their communication pattern: *"I plan to do X; will proceed in 24 hours absent objection."* Asynchronous by default, synchronous for genuine disagreements.

---

## Key Frameworks

### Framework 1: The Delegation Spectrum
From Brockman at Stripe — three models for working with co-leaders:
- **Delegate completely:** Define principles, leave execution alone. Scales well; requires high trust.
- **Stay involved in all details:** Works only for *one* area — Zuckerberg with product, Bezos with customer experience. Sustainable only at that focus.
- **Sparse micromanagement:** Jump into a random issue, overturn a decision, disappear. *Always wrong.* Destroys trust and creates unpredictable decision environments.

### Framework 2: The Dopamine Test
From Bryan Helmig (Zapier CTO), surfaced by Miguel Carranza: **where do you get your dopamine hits?** If your energy comes from writing code and solving technical puzzles, you are wired for the CTO/IC path. If it comes from growing people and shipping teams, you are wired for the VP Eng path. This self-knowledge shapes every hiring and delegation decision you'll make.

### Framework 3: Influence vs. Authority
The structural reality for most senior CTOs: **you influence more than you direct**. The VP Eng has organizational authority by role. The CTO has credibility authority by demonstrated judgment. Engineering leaders who expect their title to do the work will fail. Those who invest in trust, context, and relationship — before they need them in a conflict — succeed.

### Framework 4: Stage-Appropriate Focus
| Company Stage | Primary CTO Lever |
|---|---|
| 1–10 people | Technical architecture + customer credibility |
| 10–30 people | Delegation + first management layer |
| 30–100 people | Org design + hiring systems + tech strategy |
| 100+ people | External representation + long-horizon bets + culture at scale |

---

## Key Takeaways

1. **There is no single CTO role.** Archetype determines job description more than title does. Know which archetype applies to your company before accepting or designing the role.

2. **CTO and VP Eng are complementary, not competitive.** When a strong CTO and VP Eng trust each other, you have a winning formula. When one person tries to hold both past ~20 people, they do both poorly.

3. **Founder CTOs must evolve or stagnate.** The hardest transition is stopping coding — not because coding is wrong, but because the multiplier effect of enabling others becomes dramatically larger at scale. Your identity as a founder should be durable; your job description should not be.

4. **The first 30 days are for listening, not doing.** Every hour you invest in understanding the system before changing it pays compound interest in trust. Changes made before trust is established cost more than the change itself.

5. **Build feedback loops before you need them.** Once you delegate, you lose visibility by default. Intentional skip-levels, planning participation, and a trusted co-leader relationship are how you stay calibrated without micromanaging.

6. **The CTO is externally oriented by nature.** Customer relationships, ecosystem positioning, and market foresight are not optional CTO duties — they are the core of the role in most archetypes. Technical credibility at the sales table closes deals no marketing can close.

7. **Influence without authority is the actual job.** If you need the org chart to get things done, your credibility foundation is weak. Invest in relationships with peers and skip-levels before you're in a conflict that depends on them.

---

## Sources

- Werner Vogels — [The Different CTO Roles](https://www.allthingsdistributed.com/2007/07/the_different_cto_roles.html) (2007)
- Fred Wilson — [VP Engineering vs CTO](https://avc.com/2011/10/vp-engineering-vs-cto/) (2011)
- Greg Brockman — [Figuring Out the CTO Role at Stripe](https://blog.gregbrockman.com/figuring-out-the-cto-role-at-stripe) (2014)
- Will Larson — [Your First 90 Days as CTO or VP Engineering](https://lethain.com/first-ninety-days-cto-vpe/) (2020)
- Lara Hogan — [How to Spend Your First 30 Days in a New Senior-Level Role](https://larahogan.me/blog/first-30-days-new-role/)
- Miguel Carranza — [Evolution of My Role as a Founder CTO](https://miguelcarranza.es/cto) (RevenueCat)
