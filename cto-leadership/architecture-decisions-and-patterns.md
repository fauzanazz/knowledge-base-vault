---
title: "Architecture Decisions and Patterns for CTOs"
tags: [architecture, twelve-factor, microservices, monolith, rest-api, serverless, over-engineering, technical-decisions]
category: cto-leadership
sources:
  - "The Twelve-Factor App - Adam Wiggins (Heroku)"
  - "Reactive Manifesto"
  - "Microservices Please Don't - Sean Kelly"
  - "GitHub CTO on Microservices (2022)"
  - "The Death of Microservice Madness - Dave Kerr"
  - "10 Modern Software Over-Engineering Mistakes - RDX"
  - "Best Practices for a Pragmatic RESTful API - Vinay Sahni"
  - "Bezos API Mandate"
last_updated: 2026-04-08
---

# Architecture Decisions and Patterns for CTOs

## The Twelve-Factor App

The foundational methodology for building modern SaaS applications, created by Adam Wiggins (Heroku co-founder):

| Factor | Principle | Why It Matters |
|--------|-----------|---------------|
| **I. Codebase** | One codebase per app, many deploys | Prevents drift between environments |
| **II. Dependencies** | Explicitly declare and isolate dependencies | No "works on my machine" |
| **III. Config** | Store config in environment variables | Secrets never in code |
| **IV. Backing Services** | Treat backing services as attached resources | Database, cache, queue are all swappable |
| **V. Build, Release, Run** | Strictly separate build and run stages | Reproducible deployments |
| **VI. Processes** | Execute the app as stateless processes | Horizontal scaling, crash resilience |
| **VII. Port Binding** | Export services via port binding | Self-contained, no app server dependency |
| **VIII. Concurrency** | Scale out via the process model | Add instances, not threads |
| **IX. Disposability** | Maximize robustness with fast startup and graceful shutdown | Cloud-native, container-friendly |
| **X. Dev/Prod Parity** | Keep development, staging, and production as similar as possible | Reduces "works in staging" failures |
| **XI. Logs** | Treat logs as event streams | Centralized logging, not file-based |
| **XII. Admin Processes** | Run admin/management tasks as one-off processes | Migrations, scripts run in same environment |

### Modern Relevance (2026)

Most of the 12 factors are now **infrastructure defaults** in containerized environments (Docker, K8s). The ones teams still struggle with:
- **III. Config** — secrets management is still messy
- **X. Dev/Prod Parity** — local dev environments drift from production
- **VI. Processes** — stateful services (WebSockets, file uploads) need careful design

## The Microservices Debate

### The Case Against (Critical Perspective)

Three influential articles argue the same point from different angles:

**Sean Kelly — "Microservices, Please Don't"**
> The industry conflates "microservices" with "good architecture." They're not the same thing.

Key arguments:
1. **Premature decomposition** — you need to understand domain boundaries before splitting. Split too early → wrong boundaries → constant cross-service refactoring
2. **Network is not free** — every service call adds latency, failure modes, and debugging complexity
3. **Distributed transactions are hard** — two-phase commit, sagas, eventual consistency are not free
4. **Operational overhead** — each service needs monitoring, logging, deployment, on-call

**Dave Kerr — "The Death of Microservice Madness"**
> Netflix are great at devops. Netflix do microservices. Therefore: If I do microservices, I am great at devops. (This is a logical fallacy.)

His critique of common justifications:

| Claim | Reality |
|-------|---------|
| "Better scalability" | You can scale a monolith horizontally too |
| "Technology diversity" | Cool in theory, maintenance nightmare in practice |
| "Better fault isolation" | A cascading failure in microservices is WORSE than a monolith crash |
| "Easier to understand" | One service is simpler, but the SYSTEM is far more complex |
| "Faster deployments" | True, but at the cost of distributed system complexity |

**Jason Warner (GitHub CTO, 2022):**
> "The biggest architectural mistake we made at GitHub was going full microservice."

### When Microservices Make Sense

Despite the criticism, microservices ARE appropriate when:
1. **Different scaling requirements** — search needs 100x the compute of user profiles
2. **Different team ownership** — clear organizational boundaries (Conway's Law)
3. **Different deployment cadences** — billing deploys monthly, frontend deploys hourly
4. **Regulatory isolation** — PCI-compliant payment service isolated from everything else
5. **Technology requirements** — ML inference in Python, API in Go, real-time in Rust

### The Pragmatic Approach

```
Start Monolith → Identify boundaries → Extract when pain is clear
      ↓                                         ↑
   "Modular monolith"                    Never extract speculatively
   (well-structured, module boundaries)
```

**The Modular Monolith** is the sweet spot for most startups (< 50 engineers):
- Module boundaries enforced at code level (packages, namespaces)
- Single deployment, single database
- When a module genuinely needs independence, THEN extract

## Over-Engineering: The 10 Mistakes

RDX's taxonomy of modern software over-engineering:

### 1. Small Generalizations
Building for hypothetical future requirements instead of current ones.
- **Anti-pattern:** Generic plugin system for a feature with one plugin
- **Fix:** YAGNI — You Ain't Gonna Need It

### 2. Premature Optimization
Optimizing code that isn't a bottleneck.
- **Anti-pattern:** Custom caching layer before you have 100 users
- **Fix:** Measure first, optimize second

### 3. Over-Abstracting
Too many layers of indirection.
- **Anti-pattern:** `UserService → UserRepository → UserDataSource → UserMapper → UserEntity`
- **Fix:** If removing a layer doesn't reduce understanding, remove it

### 4. Planning for Hypothetical Scale
Designing for 1M users when you have 100.
- **Anti-pattern:** Kubernetes cluster for a prototype
- **Fix:** Design for 10x current load, not 1000x

### 5-10: Pattern Abuse
Over-using design patterns, dependency injection, microservices, event-driven architecture, CQRS, and reactive programming — all good tools that become over-engineering when applied without specific need.

**The Golden Rule:** Every abstraction, pattern, or architecture decision should be justified by a **current, concrete problem** — not a hypothetical future one.

## RESTful API Design (Pragmatic)

Vinay Sahni's principles — focused on developer experience:

### URL Design
```
GET    /tickets          # List
POST   /tickets          # Create
GET    /tickets/12       # Read
PUT    /tickets/12       # Update (full)
PATCH  /tickets/12       # Update (partial)
DELETE /tickets/12       # Delete

GET    /tickets/12/messages    # Sub-resource
```

### Key Decisions

| Decision | Recommendation | Why |
|----------|---------------|-----|
| Versioning | URL path (`/v1/`) | Simple, visible, cacheable |
| Filtering | Query params (`?status=open&sort=-created`) | RESTful, composable |
| Pagination | `Link` header + `?page=2&per_page=20` | Standard, discoverable |
| Error format | `{error: {message, code, details}}` | Consistent, machine-readable |
| Auth | OAuth 2.0 Bearer token | Industry standard |
| Rate limiting | `X-RateLimit-*` headers | Transparent to consumers |
| Envelope | No envelope, use HTTP status codes | RESTful, not `{success: true, data: {...}}` |

### HATEOAS — Skip It
In theory, APIs should include links to related resources. In practice, almost nobody implements it properly, and clients hardcode URLs anyway. Focus on good documentation (OpenAPI/Swagger) instead.

## The Bezos API Mandate

Jeff Bezos's internal email (circa 2002) that transformed Amazon:

> 1. All teams will henceforth expose their data and functionality through service interfaces.
> 2. Teams must communicate with each other through these interfaces.
> 3. There will be no other form of interprocess communication allowed.
> 4. It doesn't matter what technology they use.
> 5. All service interfaces, without exception, must be designed from the ground up to be externalizable.
> 6. Anyone who doesn't do this will be fired.

**Why this matters:**
- Forced API-first thinking 20 years before it was trendy
- Led directly to AWS (if you build internal APIs, you can sell them externally)
- Eliminated back-door data access, making services truly independent
- Proved that architecture decisions are **organizational decisions**

## Technical Decision-Making

### Accentuate the Negative

When evaluating technology choices, teams tend to focus on benefits. Better approach: **systematically evaluate downsides**.

**Framework:**
1. List all options
2. For each option, ask: "What's the WORST thing that happens if we choose this?"
3. Rank by **downside severity**, not upside potential
4. Choose the option with the most tolerable worst case

This naturally leads to:
- **Boring technology choices** (proven, well-understood)
- **Reversible decisions** (prefer options you can undo)
- **Lower operational burden** (fewer new things to learn/maintain)

## Key Takeaways

1. **Start with a modular monolith** — extract microservices only when pain is concrete
2. **The Twelve-Factor App is still relevant** — especially config, dev/prod parity, and stateless processes
3. **Over-engineering is the #1 architectural risk** for startups — YAGNI over SOLID
4. **API design is developer UX** — invest in it like you would user-facing UI
5. **Evaluate decisions by their worst case**, not their best case
6. **Architecture is organization** — Conway's Law is real, design for your team structure
7. **The Bezos Mandate** — API-first thinking enables optionality at every level
