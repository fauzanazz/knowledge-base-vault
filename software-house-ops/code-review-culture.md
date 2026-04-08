---
title: Code Review Culture
category: software-house-ops
summary: >
  A comprehensive guide to building an effective code review culture in a software house —
  covering PR workflows, checklists, SLAs, automated gates, feedback patterns, ownership
  models, and metrics that keep teams shipping quality code fast.
sources: web-research-2026
updated: 2026-04-08T19:00:00.000Z
---

# Code Review Culture

## Why Code Review Matters

Code review is the highest-leverage quality gate a software team has. Beyond catching bugs, it
spreads knowledge, enforces consistency, mentors junior engineers, and creates a shared sense of
ownership. A healthy review culture balances rigour with speed — reviews that are too shallow
miss problems; reviews that are too slow become a release bottleneck.

---

## PR Review Process

### 1. Author Responsibilities (Before Opening a PR)

- Run all automated checks locally (`lint`, `test`, `build`).
- Keep PRs **small and focused** — target < 400 lines changed. Large PRs receive lower-quality
  reviews and take longer to merge.
- Write a clear PR description: **what** changed, **why** it changed, and **how** to test it.
- Link the PR to the relevant issue/ticket.
- Self-review the diff before requesting others; resolve obvious issues first.
- Tag the correct reviewers using `CODEOWNERS` or manually assign domain experts.

### 2. Automated Gates (Before Human Review)

Human time is expensive. Block the review queue with automated checks:

| Gate | Tool Examples |
|------|--------------|
| Linting / formatting | ESLint, Prettier, Ruff, golangci-lint |
| Unit & integration tests | Jest, pytest, Go test, JUnit |
| Static Application Security Testing (SAST) | Semgrep, Snyk Code, CodeQL |
| Dependency vulnerability scan | Dependabot, OWASP Dependency-Check |
| Type checking | TypeScript `tsc --noEmit`, mypy |
| Code coverage gate | Codecov (fail if coverage drops > 2%) |

**Policy:** PRs must be green on all automated gates before a human reviewer is assigned.
This removes trivial nits from human reviews and keeps feedback focused on logic and design.

### 3. The Review Itself

Reviewers should address four dimensions:

#### Correctness
- Does the code do what the ticket/spec requires?
- Are edge cases handled (nulls, empty collections, overflow, concurrency)?
- Are error paths tested?

#### Security
- Is user input validated and sanitised before use?
- Are secrets committed? (check for API keys, passwords, tokens)
- Are SQL queries parameterised? Are there XSS vectors?
- Are new dependencies audited? Do they have known CVEs?

#### Performance
- Are there N+1 queries?
- Are large collections streamed or paginated rather than loaded into memory?
- Are expensive operations cached where appropriate?
- Is the algorithmic complexity acceptable for expected input sizes?

#### Readability & Maintainability
- Are names descriptive and consistent with project conventions?
- Is the logic easy to follow without deep domain knowledge?
- Is there appropriate inline documentation for non-obvious decisions?
- Does the code respect SOLID principles / project architecture?

---

## Review Checklist (Quick Reference)

```
[ ] PR description is clear; ticket is linked
[ ] Automated gates are green
[ ] Correctness: logic matches requirements + edge cases handled
[ ] Security: input validation, no secrets, parameterised queries
[ ] Performance: no obvious N+1 or memory bombs
[ ] Readability: names make sense, complex logic is commented
[ ] Tests added / updated for new behaviour
[ ] No dead code / debugging artefacts left in
[ ] Breaking changes documented (migration guide, changelog entry)
[ ] Database migrations are reversible
```

---

## Google's Code Review Guidelines (Summary)

Google's internal guide (now public via *How Google Does Code Reviews*) offers several
battle-tested principles:

1. **The reviewer's job is not to make the code perfect** — it is to verify that the overall
   code health of the codebase improves. Don't block on minor style issues that a formatter
   can fix.
2. **Approve with comments** — if changes are truly minor, approve and let the author address
   them without a re-review cycle.
3. **Explain the *why***, not just the *what*. "Move this to a service layer because…" is more
   useful than "move this to a service layer."
4. **Be kind.** Comment on the code, never the person.
5. **Resolve conflicts by escalation, not silence.** If reviewer and author disagree after two
   rounds, escalate to a third engineer rather than letting the PR stall.

---

## Review Turnaround SLAs

Slow reviews kill developer flow. Establish explicit SLAs:

| PR Size | First Review Response |
|---------|----------------------|
| < 50 lines | 4 business hours |
| 50–400 lines | 1 business day |
| > 400 lines | 2 business days (consider breaking up the PR) |

- **Reviewers** must acknowledge within the SLA — even if the full review takes longer, an
  "I'll review this tomorrow morning" comment resets the clock and reduces anxiety.
- Post reminders automatically via CI bot or Slack integration if the SLA is breached.
- **Authors** should address review feedback within 1 business day to keep the cycle tight.

---

## LGTM / Approval Policies

- **Minimum 1 approval** required before merge on all repositories.
- **Minimum 2 approvals** for repositories tagged `critical` (payments, auth, data pipelines).
- Enable **branch protection rules** in GitHub/GitLab — prevent force-push and require status
  checks to pass.
- Use **CODEOWNERS** files to auto-assign reviewers for specific directories:
  ```
  # .github/CODEOWNERS
  /src/payments/   @payments-team
  /infra/          @devops-team
  /docs/           @tech-writers
  ```
- Consider **required review dismissal** — if new commits are pushed after approval, approvals
  are dismissed and must be re-granted.

---

## Pair Programming as an Alternative

For complex or high-risk changes, pair programming can replace async code review entirely:

- **Driver/Navigator** model: one types, one thinks about the bigger picture.
- The pair naturally performs real-time review — bugs and design issues surface immediately.
- Particularly effective for: onboarding new engineers, legacy code archaeology, security-critical
  features, and architectural spikes.
- **Mob programming** (3+ engineers on one task) is useful for critical incidents or deliberate
  team knowledge sharing sessions.

Pairs still benefit from a lightweight async review by a third engineer to catch blind spots.

---

## Trunk-Based vs Feature-Branch Review

| Approach | Review Model | Best For |
|----------|-------------|----------|
| **Trunk-based development** | Very small PRs (hours of work); feature flags hide incomplete features | High-frequency CI/CD, mature teams |
| **Short-lived feature branches** | Standard PR review; merge within 1–2 days | Most teams |
| **Long-lived feature branches** | Larger PRs, higher merge conflict risk | Not recommended; break down features |

Trunk-based development reduces merge hell but requires strong automated test coverage and
feature-flag discipline before it becomes safe.

---

## Review Metrics

Track these to diagnose bottlenecks and improve:

| Metric | Target |
|--------|--------|
| **Time-to-first-review** | < 4 h (business hours) |
| **Cycle time** (PR open → merge) | < 2 business days |
| **Review rounds** per PR | ≤ 2 |
| **PR size** (median lines changed) | < 200 |
| **Review coverage** (% of merges with ≥ 1 approval) | 100% |

Surface these in your engineering dashboard (LinearB, Jellyfish, or a custom Grafana dashboard
querying the GitHub/GitLab API).

---

## Constructive Feedback Patterns

- **Use "we" not "you"**: "We usually extract this into a helper" vs "You should extract this."
- **Prefix with intent**: Use labels like `nit:`, `suggestion:`, `blocker:`, `question:` to
  signal severity and whether a reply is required.
- **Explain trade-offs**: If you suggest a rewrite, acknowledge the cost: "This refactor adds
  complexity but makes the caching strategy much clearer."
- **Praise good code**: A `// This is elegant, nice use of the strategy pattern!` costs nothing
  and builds psychological safety.
- **Ask, don't tell** (for non-blockers): "What do you think about extracting this into a
  constant?" opens dialogue rather than issuing commands.

---

## Common Reviewer Mistakes

1. **Reviewing style issues** that a linter should enforce — configure the linter and stop
   spending human attention on it.
2. **Approving without reading** — rubber-stamp approvals erode the entire process.
3. **Blocking on opinion** — if the code is correct and readable, personal preference is not a
   blocker. Use `suggestion:` prefix.
4. **Ghosting** — not responding to a PR for days creates developer frustration and context loss.
5. **Re-reviewing the same issue repeatedly** — if you raised it before and it was resolved
   differently, either accept the resolution or escalate; don't loop.
6. **Failing to read the ticket** — reviewing code without understanding the requirements leads
   to incorrect correctness judgements.

---

## Code Ownership (CODEOWNERS)

- Maintain a `.github/CODEOWNERS` (GitHub) or `CODEOWNERS` (GitLab) file at the repo root.
- Assign ownership at the **directory level**, not the file level, to avoid gaps.
- Rotate ownership quarterly to prevent knowledge silos.
- Owners are not gatekeepers — they are **stewards** responsible for quality in their domain
  and the first responders to issues in that code.

---

## Further Reading

- *Code Review Best Practices* — Google Engineering Practices (google.github.io)
- *Accelerate* — Forsgren, Humble, Kim (DORA metrics including review cycle time)
- *The Art of Readable Code* — Boswell & Foucher
- Conventional Comments spec — conventionalcomments.org
