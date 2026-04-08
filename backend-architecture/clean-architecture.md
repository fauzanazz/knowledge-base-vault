---
title: "Clean Architecture"
category: backend-architecture
summary: "A layered architectural pattern by Robert C. Martin that enforces strict dependency rules to isolate business logic from frameworks, databases, and external concerns."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Clean Architecture

> A layered architectural pattern by Robert C. Martin that enforces strict dependency rules to isolate business logic from frameworks, databases, and external concerns.

## Overview

Clean Architecture (popularized by Robert C. Martin, aka "Uncle Bob") organizes code into concentric layers where **dependencies only point inward**. The innermost layers contain business rules; outer layers contain I/O, frameworks, and delivery mechanisms. This makes core logic independently testable and swappable at the edges.

## The Four Layers

```
┌─────────────────────────────────────┐
│         Frameworks & Drivers        │  ← Web, DB, UI, External APIs
│  ┌───────────────────────────────┐  │
│  │    Interface Adapters         │  │  ← Controllers, Presenters, Gateways
│  │  ┌─────────────────────────┐  │  │
│  │  │   Application Use Cases │  │  │  ← Business workflows
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │   Entities        │  │  │  │  ← Enterprise business rules
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### 1. Entities
Pure domain objects encoding enterprise-wide business rules. No framework dependencies. Example: an `Order` entity that validates its own state.

### 2. Use Cases (Application Layer)
Orchestrates entities to fulfill a specific user story. Example: `PlaceOrderUseCase` — fetches inventory, validates payment, persists order. Depends only on Entities.

### 3. Interface Adapters
Translates data between use cases and the outside world. Includes Controllers (HTTP → Use Case input), Presenters (Use Case output → JSON/HTML), and Gateways (Use Case interface → DB query).

### 4. Frameworks & Drivers
The outermost ring: Express.js, Spring Boot, PostgreSQL, Redis, etc. Completely swappable without touching inner layers.

## The Dependency Rule

> *"Source code dependencies must point only inward, toward higher-level policies."*

Inner layers must **never** reference outer layers. Outer layers implement interfaces defined by inner layers (Dependency Inversion Principle). This is enforced via:
- **Repository interfaces** defined in use cases, implemented in the DB layer
- **Port interfaces** for external services (email, payments, etc.)

## Real-World Example: Uber's Fare Calculator

Uber isolates their pricing engine (entity layer) from delivery mechanisms. The same `CalculateFareUseCase` runs identically whether triggered by HTTP, an internal RPC call, or a batch job. Swapping surge pricing algorithms requires zero changes to controllers or DB adapters.

## Implementation Example (TypeScript)

```typescript
// Entity (innermost)
class Order {
  constructor(public id: string, public items: Item[], public status: OrderStatus) {}
  validate(): boolean { return this.items.length > 0; }
}

// Use Case Port (interface defined IN use case layer)
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

// Use Case
class PlaceOrderUseCase {
  constructor(private repo: OrderRepository) {}
  async execute(input: PlaceOrderInput): Promise<PlaceOrderOutput> {
    const order = new Order(uuid(), input.items, 'PENDING');
    if (!order.validate()) throw new Error('Invalid order');
    await this.repo.save(order);
    return { orderId: order.id };
  }
}

// Adapter (outermost) — implements the port
class PostgresOrderRepository implements OrderRepository {
  async save(order: Order) { /* SQL INSERT */ }
  async findById(id: string) { /* SQL SELECT */ }
}
```

## Trade-offs

| Pros | Cons |
|------|------|
| Highly testable — no DB/framework needed for unit tests | Significant boilerplate (DTOs, mappers, interfaces) |
| Framework-agnostic core | Over-engineering for simple CRUD apps |
| Clear separation of concerns | Steep learning curve for teams |
| Easy to swap databases or delivery mechanisms | Can fragment small features across many files |

## Anti-Patterns

- **Leaking frameworks into entities**: Annotating domain objects with `@Entity` (JPA) or `@Column` couples the core to the ORM
- **Fat controllers**: Placing business logic in HTTP handlers bypasses the use case layer
- **Anemic domain model**: Entities with no behavior (just getters/setters) — business rules end up scattered across use cases

## When to Use

✅ **Use when**: Long-lived applications with complex, evolving business rules; teams that need high unit test coverage; projects that may swap databases or frameworks over time.

❌ **Avoid when**: Simple CRUD APIs, prototypes, scripts, or small services where the overhead isn't justified.

## Key Principle Summary

- **Entities** know nothing about use cases or the outside world
- **Use cases** depend only on entities and interfaces they define
- **Adapters** depend on use cases; they never bleed into entities
- **Frameworks** plug into adapters — always replaceable

---
*Related: [[Hexagonal Architecture]], [[Domain-Driven Design]], [[Modular Monolith]], [[Backend for Frontend]]*
