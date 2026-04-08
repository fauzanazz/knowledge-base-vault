---
title: "Domain-Driven Design"
category: backend-architecture
summary: "A software philosophy by Eric Evans that aligns code structure with complex business domains through bounded contexts, aggregates, and a shared ubiquitous language between developers and domain experts."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Domain-Driven Design

> A software philosophy by Eric Evans that aligns code structure with complex business domains through bounded contexts, aggregates, and a shared ubiquitous language between developers and domain experts.

## Overview

Domain-Driven Design (DDD), introduced by Eric Evans in his 2003 "Blue Book" (*Domain-Driven Design: Tackling Complexity in the Heart of Software*), provides patterns and principles for modeling complex business logic. DDD is most valuable when the domain is inherently complex — not just CRUD. It treats the business domain as the primary driver of software design, with code structure reflecting real-world concepts.

DDD splits into **Strategic Design** (the big picture: how to divide the system) and **Tactical Design** (the details: how to model within a bounded context).

## Strategic Design

### Bounded Context
The most important DDD concept. A **Bounded Context** is an explicit boundary within which a domain model is defined and applicable. The same word can mean different things in different contexts.

**Example**: At a retail company:
- **Catalog Context**: A `Product` has a name, description, images, and SEO metadata
- **Inventory Context**: A `Product` has a SKU, warehouse location, and stock count
- **Billing Context**: A `Product` has a price, tax category, and discount eligibility

Each context has its own model. Trying to create a single unified `Product` becomes a nightmare.

### Context Map
Describes how Bounded Contexts relate to each other. Relationship patterns:
- **Shared Kernel**: Two teams share a small common model (high coordination cost)
- **Customer-Supplier**: Upstream context provides API; downstream consumes it
- **Anti-Corruption Layer (ACL)**: Translate between a legacy/external model and your domain model — prevents corruption of your clean domain
- **Published Language**: A well-documented, shared protocol (e.g., canonical event schema)

### Ubiquitous Language
A shared vocabulary used consistently in code, docs, conversations, and tests. If the business says "reservation," the code has `Reservation` — not `Booking`, `Hold`, or `TempOrder`. When language diverges, misunderstandings compound.

```java
// Bad — developer-centric naming
public class DataRecord { public void processItem() {} }

// Good — ubiquitous language
public class Reservation { public void confirm() {} }
```

## Tactical Design

### Entities
Objects with a distinct identity that persists over time. Two entities with the same attributes are still different if they have different IDs.

```java
public class Customer {
    private final CustomerId id;   // identity
    private String email;
    private String name;

    public void changeEmail(String newEmail) {
        // business rule: validate email format
        this.email = new Email(newEmail).value();
    }
}
```

### Value Objects
Immutable objects with no identity — defined entirely by their attributes. Two value objects with the same attributes are equal.

```java
public record Money(BigDecimal amount, Currency currency) {
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) throw new CurrencyMismatchException();
        return new Money(this.amount.add(other.amount), this.currency);
    }
}
```

### Aggregates
A cluster of related entities and value objects treated as a single unit for data changes. One entity is the **Aggregate Root** — all external interactions go through it. Enforces invariants (business rules) within the boundary.

```java
public class Order { // Aggregate Root
    private OrderId id;
    private List<OrderLine> lines;    // child entities/VOs inside aggregate
    private OrderStatus status;

    public void addItem(ProductId productId, int quantity, Money price) {
        if (status != OrderStatus.DRAFT) throw new OrderNotEditableException();
        lines.add(new OrderLine(productId, quantity, price));
    }

    public Money totalAmount() {
        return lines.stream().map(OrderLine::subtotal).reduce(Money.ZERO, Money::add);
    }
}
```

**Aggregate Rules**:
- Load/save the entire aggregate as a unit
- External objects reference aggregates by ID only (not direct object references)
- Keep aggregates small — large aggregates cause contention and slow performance

### Domain Events
Significant occurrences in the domain, named in past tense. Used to communicate between aggregates and bounded contexts.

```java
public record OrderPlaced(OrderId orderId, CustomerId customerId, Instant occurredAt) {}
```

### Domain Services
Logic that doesn't naturally belong to a single entity or value object.

```java
public class FraudDetectionService {
    public RiskScore assess(Order order, Customer customer) { /* cross-entity logic */ }
}
```

### Repositories
Abstractions for loading/saving aggregates. One repository per aggregate root.

## Real-World Adoption

**Amazon**: Teams are organized around bounded contexts (Orders, Payments, Fulfillment, Catalog). Each team owns its model independently — a key reason Amazon could decompose into microservices successfully.

**Netflix**: Their content metadata system uses DDD with separate contexts for editorial (what a movie "is"), licensing (where it can be shown), and streaming (how it's encoded/delivered).

**Uber**: The trip domain contains aggregates like `Trip`, `Driver`, `Rider` — each with strict invariants enforced at the aggregate root before persistence.

## Strategic DDD and Microservices

Bounded Contexts are the natural unit for microservice boundaries. Each service owns one or more bounded contexts. However, **not every bounded context needs to be a separate service** — a Modular Monolith can house multiple bounded contexts in separate modules.

## Trade-offs

| Pros | Cons |
|------|------|
| Models closely mirror real business processes | Requires close collaboration with domain experts |
| Reduces accidental complexity in complex domains | Heavy upfront investment to discover the domain |
| Natural fit for microservice decomposition | Overkill for simple CRUD applications |
| Explicit language reduces miscommunication | Learning curve for teams new to DDD |
| Events enable loose coupling between contexts | Eventual consistency is harder to reason about |

## Anti-Patterns

- **Anemic Domain Model**: Entities are just data bags; all logic lives in service classes — loses OO benefits
- **God Aggregate**: One aggregate that encompasses the entire domain — causes concurrency bottlenecks
- **Skipping the Ubiquitous Language**: Code uses different terms than the business; drift accumulates
- **Sharing a database across Bounded Contexts**: Undermines the context boundary entirely

## When to Use

✅ **Use when**: Complex, evolving business domains with rich rules; large teams needing clear ownership; systems being decomposed into microservices.

❌ **Avoid when**: Simple CRUD apps, data pipelines, or domains where business rules are minimal.

---
*Related: [[Clean Architecture]], [[Hexagonal Architecture]], [[Modular Monolith]], [[Event-Driven Architecture]], [[Database per Service]]*
