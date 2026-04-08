---
title: "Modular Monolith"
category: backend-architecture
summary: "A single deployable unit partitioned into strictly bounded, loosely coupled modules with enforced internal APIs — capturing microservice clarity while avoiding distributed systems complexity."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Modular Monolith

> A single deployable unit partitioned into strictly bounded, loosely coupled modules with enforced internal APIs — capturing microservice clarity while avoiding distributed systems complexity.

## Overview

A Modular Monolith is an architectural style where a single deployable application is internally divided into well-defined, loosely coupled **modules** with explicit boundaries. Each module owns its code, data schema, and exposes a controlled API to other modules. It combines the operational simplicity of a monolith with the structural clarity of microservices.

Shopify, Stack Overflow, and Basecamp run significant production workloads as modular monoliths. Shopify's Rails codebase — serving millions of merchants — is organized into components (Billing, Identity, Shipping) with enforced boundaries rather than a distributed mesh of services.

## Why Not Just Microservices?

Microservices introduce: network latency, distributed transactions, operational overhead (12+ services to deploy/monitor), service discovery, eventual consistency challenges, and debugging complexity across service boundaries. For many teams and domains, these costs outweigh the benefits — especially early in a product's lifecycle.

A Modular Monolith delivers module isolation **without** the distribution tax.

## Module Structure

Each module is a self-contained vertical slice:

```
src/
├── orders/
│   ├── api/              ← Public API (interfaces/DTOs exposed to other modules)
│   │   └── OrderService.java
│   ├── domain/           ← Entities, value objects, domain events
│   ├── application/      ← Use cases, command/query handlers
│   ├── infrastructure/   ← DB adapters, external service clients
│   └── OrdersModule.java ← Module registration/wiring
├── inventory/
│   ├── api/
│   │   └── InventoryService.java
│   ├── domain/
│   ├── application/
│   └── infrastructure/
├── payments/
│   └── ...
└── shared/               ← MINIMAL shared kernel (IDs, Money, events)
```

## Enforcing Module Boundaries

Boundaries must be enforced by tooling, not convention. Several approaches:

### Java: ArchUnit
```java
@Test
void orders_should_not_depend_on_payments_internals() {
    noClasses()
        .that().resideInAPackage("..orders..")
        .should().dependOnClassesThat()
        .resideInAPackage("..payments.domain..")
        .check(importedClasses);
}
```

### Java: JPMS (Project Jigsaw)
```java
// module-info.java for orders module
module com.example.orders {
    requires com.example.inventory.api; // Only the API, not internals
    exports com.example.orders.api;
}
```

### .NET: Solution structure + Roslyn analyzers
Split modules into separate projects; internal types stay `internal`, only API surfaces are `public`.

## Internal Communication Patterns

### Direct Method Call (Synchronous)
The simplest approach — modules call each other's API interfaces:
```java
// Orders module calls Inventory via injected interface
public class PlaceOrderHandler {
    private final InventoryService inventoryService; // from inventory.api package

    public void handle(PlaceOrderCommand cmd) {
        inventoryService.reserve(cmd.productId(), cmd.quantity()); // in-process call
        // ...
    }
}
```

### In-Process Event Bus (Asynchronous)
Modules emit domain events; other modules subscribe — same process, no network:
```java
// Orders publishes
eventBus.publish(new OrderPlaced(orderId, items));

// Notifications subscribes
eventBus.on(OrderPlaced.class, event -> notificationService.sendConfirmation(event.orderId()));
```

This mirrors microservice event-driven communication without the broker complexity. Transitioning to a real broker (Kafka) later requires minimal changes.

## Database Strategy

Modular monoliths use a single database but with **schema-per-module** to enforce data ownership:

```sql
-- Each module owns its schema
CREATE SCHEMA orders;
CREATE SCHEMA inventory;
CREATE SCHEMA payments;

-- No cross-schema foreign keys (use IDs only)
CREATE TABLE orders.order_items (
    order_id UUID,
    product_id UUID,  -- reference by ID, not FK to inventory.products
    quantity INT
);
```

This makes future extraction to separate services much easier — just point the module at its own database.

## Migration Path to Microservices

A Modular Monolith is an excellent stepping stone:

```
Phase 1: Modular Monolith (single deploy, multiple modules)
Phase 2: Extract high-load/independent modules to services (Strangler Fig)
Phase 3: Full microservices (if genuinely needed)
```

Criteria for extracting a module:
- Independent scaling needs (e.g., payment processing spikes)
- Different tech requirements (e.g., ML model serving in Python)
- Separate team/release cadence
- Compliance isolation (e.g., PCI scope reduction)

## Real-World Examples

**Shopify**: Rails monolith organized into "components" with strict dependency rules enforced via Packwerk (custom tooling). Components communicate through explicit APIs; violations fail CI.

**Stack Overflow**: .NET monolith with modular design handling 1B+ page views/month. Teams concluded their monolith scales better for their access patterns than distributed alternatives.

**Basecamp (Hey)**: Rails modular monolith — DHH and team openly advocate against premature microservice adoption.

## Trade-offs

| Pros | Cons |
|------|------|
| Single deployment artifact | Harder to scale individual modules |
| In-process calls — no serialization/network overhead | Shared process — one bad module can crash all |
| Shared transaction support | Risk of boundary erosion without enforcement |
| Easier debugging (single process, single log) | Shared DB can become a coupling point |
| Natural migration path to microservices | Limits technology diversity per module |

## Anti-Patterns

- **The Big Ball of Mud**: No actual module boundaries — just folders without enforcement
- **Chatty cross-module calls**: Module A imports dozens of things from Module B's internals
- **Shared domain model**: One giant domain model shared across all modules — defeats purpose
- **Premature extraction**: Extracting modules to services before understanding boundaries well

## When to Use

✅ **Use when**: Early-stage products, teams under ~20 engineers, domains not yet fully understood, or when operational simplicity is a priority.

❌ **Avoid when**: Teams truly need independent deployment/scaling per domain, or when different modules require completely different tech stacks.

---
*Related: [[Clean Architecture]], [[Domain-Driven Design]], [[Strangler Fig Pattern]], [[Database per Service]]*
