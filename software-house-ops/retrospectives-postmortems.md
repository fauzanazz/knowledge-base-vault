---
title: Sprint Retrospectives & Postmortems
category: software-house-ops
summary: >
  A practical guide to running effective sprint retrospectives and blameless postmortems —
  covering retro formats, facilitation techniques, action item tracking, RCA methods,
  incident severity levels, anti-patterns, and remote tooling.
sources: web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Sprint Retrospectives & Postmortems

## Overview

**Retrospectives** are regular team rituals for continuous improvement — what went well, what
didn't, and what to change. **Postmortems** are structured incident reviews triggered by
production failures or significant misses. Both serve the same underlying goal: turning
experience into durable organisational learning. Neither works without psychological safety.

---

## Sprint Retrospective Formats

### 1. Start / Stop / Continue

The simplest and most widely used format.

- **Start** — things the team should begin doing.
- **Stop** — practices that are hurting the team.
- **Continue** — things working well that should be preserved.

*Best for:* New teams; quick cadence (< 60 min); teams that want low facilitation overhead.

---

### 2. 4Ls — Liked, Learned, Lacked, Longed For

- **Liked** — what was enjoyable or effective.
- **Learned** — new knowledge or insights gained.
- **Lacked** — what was missing (skills, tools, clarity).
- **Longed For** — what the team wishes it had.

*Best for:* Teams that want richer reflection than Start/Stop/Continue; post-milestone retros.

---

### 3. Sailboat (aka Speed Boat)

Visual metaphor drawn on a whiteboard or Miro board:

- **Wind in the sails** — what's propelling the team forward.
- **Anchors** — what's slowing the team down.
- **Rocks ahead** — upcoming risks.
- **Sun/Island** — the team's goal or vision.

*Best for:* Visual thinkers; longer-term planning retros; teams feeling stuck.

---

### 4. Mad / Sad / Glad

Emotional format that encourages honesty about feelings:

- **Mad** — things that frustrated or angered the team.
- **Sad** — things that disappointed or demoralised.
- **Glad** — things that made the team happy or proud.

*Best for:* Teams after a difficult sprint; building psychological safety; surfacing morale issues.

---

### Other Formats Worth Knowing

| Format | One-liner |
|--------|-----------|
| **KALM** (Keep/Add/Less/More) | Nuanced version of Start/Stop/Continue |
| **Timeline Retro** | Team maps sprint events on a timeline, adds emotions |
| **Lean Coffee** | Democratic agenda: team votes on discussion topics |
| **Futurespective** | Future-focused: "It's 6 months from now and we're thriving — what did we do?" |

---

## Facilitation Tips

1. **Set the stage** — open with a short icebreaker or check-in to prime psychological safety.
   The Agile retrospectives *ESVP* check-in (Explorer/Shopper/Vacationer/Prisoner) helps gauge
   engagement.
2. **Timebox strictly** — a 2-week sprint retro should run 60–90 minutes max. Use a visible
   timer.
3. **Silent writing first** — give 5–7 minutes for individuals to write sticky notes silently
   before sharing. This prevents anchoring bias (loudest voice dominating).
4. **Affinity mapping** — cluster similar sticky notes into themes before discussing.
5. **Dot voting** — give each participant 3–5 votes to prioritise which themes to discuss.
   Focus discussion on the top 2–3 themes only.
6. **Rotate the facilitator** — avoid the Scrum Master always running retros; rotation builds
   facilitation skills across the team.
7. **Read back previous action items first** — start every retro by reviewing last sprint's
   commitments. If items weren't done, discuss why before generating new ones.
8. **Close the loop** — end with explicit commitments: who owns each action item, by when.

---

## Action Item Tracking

The most common retro failure is generating action items that disappear. Prevent this with:

- **Max 3 action items per sprint** — focus over quantity. Teams that commit to 10 items
  complete zero.
- Every item needs an **owner** (a named person, not "the team") and a **due date**.
- Add action items as tickets in your project tracker (Jira, Linear, GitHub Issues) immediately
  during the retro — not after.
- Dedicate the first 5 minutes of every subsequent retro to reviewing previous items:
  *Done / In Progress / Dropped (with reason)*.
- Track a **retro completion rate** metric: % of action items completed by the next retro.
  Target ≥ 70%.

---

## Blameless Postmortems (SRE Model)

The blameless postmortem, popularised by Google's Site Reliability Engineering (SRE) book, is
the gold standard for incident reviews. The core principle: **systems fail, not people**. When
individuals fear punishment, they hide information, leading to repeated incidents.

### Incident Severity Levels

Define severity levels consistently across the organisation:

| Severity | Definition | Response SLA |
|----------|-----------|-------------|
| **SEV-1** | Total service outage; all users affected | Immediate response; postmortem required |
| **SEV-2** | Major feature unavailable; significant user impact | Response within 30 min; postmortem required |
| **SEV-3** | Degraded performance or minor feature broken | Response within 4 h; postmortem optional |
| **SEV-4** | Cosmetic / low-impact issue | Normal sprint cycle; no postmortem |

Postmortems are mandatory for SEV-1 and SEV-2 incidents.

---

## Postmortem Template

```markdown
## Incident Postmortem — [Service Name] [Date]

**Severity:** SEV-1 / SEV-2
**Duration:** HH:MM (detection time → resolution time)
**Impact:** X% of users affected; Y requests failed; $Z revenue impact (if known)
**Author(s):**
**Review Date:**

---

### Timeline
| Time (UTC) | Event |
|------------|-------|
| 14:32 | Alert fired — p99 latency > 5 s |
| 14:35 | On-call engineer acknowledges |
| ... | ... |

---

### Root Cause
One sentence: "A misconfigured connection pool limit caused the database to reject new
connections under load, resulting in cascading 503 errors."

---

### Contributing Factors
- Factor 1
- Factor 2

---

### What Went Well
- Monitoring caught the issue within 3 minutes.
- Runbook for DB connection issues was accurate and up to date.

---

### What Went Poorly
- On-call engineer lacked context on the new connection pool config.
- Rollback procedure was undocumented.

---

### Action Items
| Action | Owner | Due Date | Ticket |
|--------|-------|----------|--------|
| Add connection pool limits to deployment checklist | @alice | 2026-04-15 | ENG-1234 |
| Write runbook for connection pool exhaustion | @bob | 2026-04-22 | ENG-1235 |
```

---

## RCA Techniques

### 5 Whys

Iteratively ask "why?" to peel back symptom layers and reach a root cause:

1. **Why** did users see 503 errors? → DB rejected connections.
2. **Why** did DB reject connections? → Connection pool was exhausted.
3. **Why** was the pool exhausted? → Pool limit was set too low after a config change.
4. **Why** was it set too low? → The config change wasn't reviewed for load impact.
5. **Why** wasn't it reviewed? → There's no checklist for config changes that affect limits.

**Root cause:** No process for reviewing load-sensitive config changes. Fix the process.

Stop when you reach something actionable, not when you reach a person.

### Fishbone Diagram (Ishikawa)

Map contributing causes across six categories: **People, Process, Tools, Environment,
Materials, Measurement**. Useful when an incident has many contributing factors and the
5 Whys feels too linear.

---

## Sharing Learnings Across Teams

- **Postmortem digest** — monthly Slack post or newsletter summarising recent postmortems and
  their action items. Strip sensitive details; keep the technical learnings.
- **Engineering all-hands** — dedicate 10 minutes to a "postmortem spotlight" once a month.
- **Searchable postmortem database** — store all postmortems in Confluence, Notion, or a Git
  repository. Tag by service, severity, and root cause category.
- **Cross-team retro** — quarterly retro across multiple squads for systemic issues that
  transcend individual team boundaries.

---

## Retro Anti-Patterns

| Anti-Pattern | Why It's Harmful | Fix |
|-------------|-----------------|-----|
| **The blame game** | Individuals become defensive; root causes stay hidden | Establish blameless norms explicitly |
| **No action items** | Retro feels pointless; morale drops | Require ≥ 1 action item per retro |
| **Same format every sprint** | Teams disengage; produces the same themes | Rotate formats every 3–4 sprints |
| **Skipping retro when busy** | The sprints you most need a retro are the hard ones | Treat retro as non-negotiable |
| **HiPPO effect** (Highest Paid Person's Opinion) | Lower seniority team members self-censor | Use anonymous sticky notes; facilitate actively |
| **Action item graveyard** | Items accumulate; nobody checks them | Track and review at start of next retro |
| **Complaining without solutions** | Demoralising; no improvement | Reframe: "What would success look like?" |

---

## Measuring Retro Effectiveness

- **Action item completion rate** — % of items closed by next retro. Target ≥ 70%.
- **Team health survey** — quarterly NPS-style pulse check: "How much do you feel our retros
  improve the team?" (1–10 scale). Track trend over time.
- **Recurring theme reduction** — if the same theme appears in 3+ consecutive retros, it's a
  systemic issue requiring escalation beyond the team.
- **Participation rate** — are all team members contributing sticky notes? Silent participants
  indicate psychological safety or facilitation issues.

---

## Remote Retro Tools

| Tool | Strengths |
|------|-----------|
| **Miro** | Rich whiteboard; supports all retro formats; real-time collaboration |
| **FunRetro / EasyRetro** | Purpose-built retro tool; simple UI; anonymous voting |
| **Parabol** | Agile-native; integrates with Jira/GitHub; tracks action items |
| **Retrium** | Multiple formats; built-in facilitation guides; analytics |
| **Metro Retro** | Playful templates; anonymous mode; action item tracking |
| **Notion / Confluence** | Low-tech; good for async retros with distributed timezones |

**Remote facilitation tips:**
- Use **breakout rooms** for small-group silent writing before full-group discussion.
- **Video on** for retros — facial expressions matter for emotional formats.
- Use a **parking lot** area on the board for off-topic items.
- Share the board link 24 h before the retro so remote participants can pre-populate stickies.

---

## Further Reading

- *Agile Retrospectives: Making Good Teams Great* — Derby & Larsen
- *Site Reliability Engineering* — Google (sre.google) — Chapter 15: Postmortem Culture
- retromat.org — 100+ retro activity ideas
- Atlassian's Incident Management Handbook
