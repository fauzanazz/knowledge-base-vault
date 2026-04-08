---
title: "Strangler Fig Pattern"
category: backend-architecture
summary: "An incremental migration strategy where new functionality is built around an existing legacy system, gradually replacing it piece by piece until the legacy system can be safely retired."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Strangler Fig Pattern

> An incremental migration strategy where new functionality is built around an existing legacy system, gradually replacing it piece by piece until the legacy system can be safely retired.

## Overview

Named after the strangler fig tree (Ficus aurea) that grows around a host tree, eventually replacing it, this pattern was coined by Martin Fowler in 2004. Instead of a high-risk "big bang" rewrite, you incrementally build new services alongside the legacy system. A **facade/proxy** routes traffic — gradually shifting calls from old to new until the legacy system handles nothing and can be decommissioned.

This is the dominant pattern for legacy-to-microservices migrations in production.

## The Strangler Fig Lifecycle

```
Phase 1: Facade introduced
  Client → [Facade] → Legacy System (100% traffic)

Phase 2: New service built for one domain
  Client → [Facade] → Legacy System (80% functionality)
                   → New Auth Service (20% functionality)

Phase 3: More domains migrated
  Client → [Facade] → Legacy System (30% functionality)
                   → Auth Service
                   → Order Service
                   → Inventory Service

Phase 4: Legacy retired
  Client → [Facade] → Auth Service
                   → Order Service
                   → Inventory Service
                   → (Legacy: decomissioned)
```

## Core Components

### The Facade (Strangler Facade)
A proxy/router that sits in front of both old and new systems. It's the migration control plane:
- Initially passes all traffic to the legacy system
- Progressively routes specific paths/domains to new services
- Provides a stable external interface during migration

**Implementations**: NGINX, Envoy, API Gateway, AWS Application Load Balancer, or a dedicated reverse proxy service.

```nginx
# NGINX facade configuration
# Old routes go to legacy
location /api/v1/products {
    proxy_pass http://legacy-system:8080;
}

# Migrated routes go to new service
location /api/v1/auth {
    proxy_pass http://new-auth-service:3000;
}

location /api/v1/orders {
    proxy_pass http://new-order-service:4000;
}
```

### Feature Flags for Gradual Traffic Shifting
```python
# Canary migration with feature flags
def route_user_request(user_id: str, endpoint: str):
    if endpoint == '/orders' and feature_flags.is_enabled('new_order_service', user_id):
        return new_order_service.handle(endpoint)
    return legacy_system.handle(endpoint)
```

This allows: test with 1% of users → 10% → 50% → 100% → decommission.

## Migration Strategy

### Step 1: Identify Seams
Find natural decomposition points in the legacy system. Domains with:
- High change frequency (hotspots)
- Clear ownership by one team
- Relatively clean interfaces to the rest

### Step 2: Build the Facade
Don't modify the legacy system yet. Insert a reverse proxy that forwards everything to legacy. Verify parity.

### Step 3: Anti-Corruption Layer (ACL)
Build a translation layer that maps between legacy data models and new domain models. The legacy system uses `CUST_ID` and a flat record; the new service uses a rich `Customer` aggregate.

```java
// Anti-Corruption Layer
public class LegacyCustomerAdapter {
    public Customer fromLegacy(LegacyCustomerRecord legacy) {
        return Customer.builder()
            .id(CustomerId.of(legacy.getCUST_ID()))
            .email(new Email(legacy.getEMAIL_ADDR()))
            .name(new Name(legacy.getFIRST_NM(), legacy.getLAST_NM()))
            .build();
    }
}
```

### Step 4: Extract and Migrate
Build the new service, run it in shadow mode (receives traffic but results discarded), then gradually shift real traffic.

### Step 5: Data Migration
Strategies for moving data from legacy DB to new service DB:
- **Dual write**: New service writes to both legacy and new DB during transition
- **Change Data Capture (CDC)**: Debezium reads legacy DB change log, syncs to new DB
- **Backfill + cutover**: Migrate historical data in batch, then cut over with CDC for live sync

## Real-World Examples

**Netflix** (2008–2016): Migrated from a monolithic Oracle DB and Java EE app to microservices using Strangler Fig over 8 years. Each domain (streaming, billing, content metadata) was extracted incrementally with zero downtime to members.

**Uber**: Migrated from a monolithic Python/Node.js service to domain-specific microservices. The dispatch system, payment processing, and driver management were extracted one at a time behind routing facades.

**Amazon**: Their move from a monolithic Obidos system to service-oriented architecture in the early 2000s is the canonical Strangler Fig example — teams extracted services domain by domain over years.

**Shopify**: Uses Strangler Fig within their Rails monolith → Storefront Renderer migration. New read-optimized paths handle storefront rendering while the legacy monolith still handles writes.

## Shadow Mode (Dark Launch)

Before shifting real traffic, run the new service in shadow mode:
```
Request → Facade → Legacy (response returned to client)
               └→ New Service (async, response discarded — compare in background)
```
This validates correctness with zero user impact. Tools: **Scientist** (GitHub's Ruby library), **Diffy** (GoLang).

## Trade-offs

| Pros | Cons |
|------|------|
| Zero downtime migration | Complexity of running two systems in parallel |
| Incremental risk — each step is independently reversible | Facade becomes a critical single point of failure |
| Teams can work on new services without touching legacy | Data synchronization between old and new systems is hard |
| Allows iterative learning about the domain | Migration takes months to years |
| No "big bang" rewrite risk | Temporary dual-write increases latency |

## Anti-Patterns

- **Skipping the facade**: Trying to migrate by modifying the legacy system — destabilizes running production
- **Migrating too many things at once**: Parallel migrations create coordination chaos
- **No shadow mode**: Discovering correctness issues after shifting real users
- **Leaving the facade forever**: The facade should be retired once legacy is gone

## When to Use

✅ **Use when**: Migrating legacy monoliths to microservices; replacing legacy systems that cannot be taken down; incremental modernization with risk containment.

❌ **Avoid when**: The legacy system is so tightly coupled that no seams exist (consider a domain model refactor first); or when the legacy system must be replaced completely for licensing/security reasons.

---
*Related: [[Modular Monolith]], [[API Gateway Pattern]], [[Domain-Driven Design]], [[Database per Service]]*
