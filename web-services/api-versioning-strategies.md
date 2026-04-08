---
title: "API Versioning Strategies"
category: web-services
summary: "Taxonomy of API versioning approaches, breaking vs non-breaking changes, deprecation policies, and real-world patterns from Stripe, GitHub, and Twilio."
sources:
  - web-research-2026
updated: 2026-04-08
---
# API Versioning Strategies
> Taxonomy of API versioning approaches, breaking vs non-breaking changes, deprecation policies, and real-world patterns from Stripe, GitHub, and Twilio.

## Versioning Approaches

### 1. URI Path Versioning (`/v1/`)
The most common and least controversial approach. The version is explicit and visible in every request.

```
GET /v1/orders/123
GET /v2/orders/123
```

**Pros:**
- Immediately visible in logs, browser history, and documentation
- Easy to route at the load balancer / API gateway level
- Clients can hardcode the version — no negotiation complexity
- Cache-friendly (version is part of the URL key)

**Cons:**
- "Breaks" REST purity (version is not a resource property)
- Clients must update all URLs when migrating
- Encourages "version per feature" sprawl if discipline is lacking

**Used by:** Stripe, Twilio, PayPal, most REST APIs in the wild.

### 2. Header Versioning (Accept Header / Custom Header)
Version is expressed via HTTP headers, keeping URLs "clean."

```http
# Content negotiation style (RFC 6838)
GET /orders/123
Accept: application/vnd.myapi.v2+json

# Custom header style
GET /orders/123
API-Version: 2024-01-01
```

**Pros:**
- URLs remain stable across versions
- Enables fine-grained content negotiation per resource
- "Correct" from a REST academic perspective

**Cons:**
- Not visible in browser address bar or basic logging
- Harder to test with curl/browser without tooling
- Caches must be keyed on `Vary: Accept` or the custom header — many CDNs handle this poorly
- Client libraries must remember to set the header on every call

**Used by:** GitHub (`X-GitHub-Api-Version`), some enterprise APIs.

### 3. Query Parameter Versioning
Version is a query parameter, visible in URLs but not in the path.

```
GET /orders/123?api_version=2
GET /orders/123?version=2024-01-01
```

**Pros:**
- Easy to add to existing URLs
- Visible and loggable
- Simple fallback: omit param → get default (usually latest or oldest stable)

**Cons:**
- Clutters URLs for clients who always need the same version
- Query params are often stripped by some proxies/caches
- Ambiguity: is it a filter or a version?

**Used by:** AWS (some services), Google Cloud APIs.

---

## Semantic Versioning for APIs

SemVer (`MAJOR.MINOR.PATCH`) needs adaptation for HTTP APIs since clients may not control which version they call.

| Component | API meaning |
|---|---|
| **MAJOR** | Breaking change — clients must opt in |
| **MINOR** | Non-breaking addition — clients safe to ignore |
| **PATCH** | Bug fix — behavior correction, no contract change |

**Practical recommendation:** Surface only the major version in URLs (`/v1/`, `/v2/`). Track minor/patch internally for changelogs and documentation. Avoid publishing minor versions in URLs — it creates an explosion of version strings (`/v1.2.3/`) and confuses clients about compatibility guarantees.

**Date-based versioning** (Stripe's model): `2023-10-16` style. No semantic encoding — every date is potentially breaking, and clients pin to a date. Requires excellent changelogs but avoids SemVer philosophy debates.

---

## Breaking vs Non-Breaking Changes

Knowing what *is* and *isn't* breaking is essential before deciding when to bump the major version.

### Non-Breaking Changes (Safe to Ship Anytime)
- Adding a new optional request field
- Adding a new response field (if clients use tolerant readers)
- Adding a new endpoint or resource type
- Adding a new enum value (⚠️ risky if clients switch exhaustively)
- Expanding string length limits
- Relaxing validation rules
- Adding a new optional HTTP header

### Breaking Changes (Require New Major Version)
- Removing or renaming a field
- Changing a field's type (e.g., `string` → `integer`)
- Changing URL structure
- Removing an endpoint
- Changing authentication requirements
- Tightening validation (e.g., now requiring a formerly optional field)
- Changing error response format
- Changing pagination mechanics
- Making a formerly optional field required

> **Opinionated take:** Treat new required enum values in requests as breaking. Clients that don't know the value can't construct valid requests.

---

## Sunset Headers and Deprecation Policies

### The `Sunset` HTTP Header (RFC 8594)
Tells clients when a version will be decommissioned.

```http
HTTP/1.1 200 OK
Sunset: Sat, 01 Jun 2026 00:00:00 GMT
Deprecation: Mon, 01 Jan 2024 00:00:00 GMT
Link: <https://docs.api.example.com/migration/v1-to-v2>; rel="successor-version"
```

Clients that monitor headers can surface deprecation warnings automatically. Implement this in your SDK/client library.

### Deprecation Policy Best Practices
- **Minimum notice period:** 6–12 months for public APIs; 2–4 weeks for internal.
- **Communicate via:** Sunset header, email to registered developers, changelog, status page.
- **Deprecation ≠ removal:** Mark as deprecated immediately when replacement is available; remove only after the sunset date.
- **Track usage:** Log which API keys are still hitting deprecated endpoints. Contact high-traffic consumers directly before removal.
- **Run both versions in parallel** during the sunset window — never force a big-bang migration.

### Deprecation Response Pattern
```http
HTTP/1.1 299 Miscellaneous Persistent Warning
Warning: 299 - "This endpoint is deprecated. Migrate to /v2/orders by 2026-06-01."
```

The `299` warning code is passed through by most proxies and can be surfaced by client libraries.

---

## Version Negotiation and Content Negotiation

### Fallback Chain
```
Explicit version header/param
  → Default version (usually oldest stable for safety)
  → Reject with 400 if unknown version requested
```

Never silently upgrade a client to a newer version — this breaks the contract. Never silently downgrade — this hides errors.

### Content Negotiation Details
When using `Accept` header versioning, respond with `Content-Type` echoing the negotiated version:

```http
# Request
Accept: application/vnd.myapi.v2+json, application/vnd.myapi.v1+json;q=0.9

# Response — server picked v2
Content-Type: application/vnd.myapi.v2+json
Vary: Accept
```

The `Vary: Accept` header is critical for cache correctness; without it, a cached v1 response may be served to a v2 client.

---

## Migration Strategies

### Parallel Run (Strangler Fig)
Run v1 and v2 simultaneously. Route traffic to both and compare responses (shadow mode). Migrate clients incrementally. Decommission v1 only when traffic drops to zero (or after sunset date).

```
Client A (old)  → /v1/orders  → v1 handler
Client B (new)  → /v2/orders  → v2 handler
Migration proxy → /v1/orders  → v1 handler + shadow call to v2 (response discarded)
```

### Adapter / Shim Pattern
Implement v2 as the canonical version. v1 is a thin adapter that translates requests to v2 format and transforms responses back to v1 format.

```python
# v1 adapter — translates old field names to v2 schema
def get_order_v1(order_id: str) -> dict:
    v2_response = v2_handler.get_order(order_id)
    return {
        "id": v2_response["order_id"],        # renamed field
        "user": v2_response["customer_id"],   # renamed field
        "status": map_status_v2_to_v1(v2_response["status"]),
    }
```

**Pro:** Single source of truth in v2 logic.
**Con:** Adapter rot — complex transformations accumulate technical debt.

### Version Negotiation at the Gateway
Use an API gateway (Kong, Apigee, AWS API Gateway) to route based on version and strip version headers before passing to backends.

---

## Real-World Approaches

### Stripe: Date-Based Versioning
Stripe pins each API key to a **version date** (e.g., `2023-10-16`). New API keys get the latest version. Old keys continue to get the behavior of their pinned date. Stripe maintains years of version compatibility in a single codebase using feature flags and conditional logic per version.

- **Lesson:** This works at scale but requires extreme discipline — every code path must be version-aware.
- **Migration UX:** Stripe dashboard shows your current version and a changelog diff to the latest. You can explicitly upgrade.

### GitHub: Header-Based with Date Versions
GitHub uses `X-GitHub-Api-Version` header with date-based versions. Deprecated versions emit `Sunset` headers. GitHub's GraphQL API sidesteps REST versioning entirely — schema evolution via optional fields and deprecation annotations.

- **Lesson:** GraphQL naturally handles many non-breaking changes; consider it for complex, evolving schemas.

### Twilio: URI Versioning with Long Deprecation Windows
Twilio uses `/v1/`, `/v2/` URI versioning with very long deprecation windows (sometimes years). Core APIs like SMS have been `/2010-04-01/` since 2010 — never versioned again, only extended.

- **Lesson:** For foundational APIs, extreme stability beats clean versioning. If you get v1 right, you never need v2.

---

## Decision Framework

```
Is this change breaking?
├── No → Ship it; update changelog; done.
└── Yes
    ├── Internal API (team controls all clients)?
    │   └── Coordinate migration; update in lock-step or use adapter.
    └── Public API
        ├── Create v(N+1) route/handler
        ├── Publish migration guide
        ├── Add Sunset header to v(N)
        ├── Set deprecation date (minimum 6 months out)
        └── Monitor and contact high-usage clients
```
