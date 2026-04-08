---
title: "Database per Service"
category: backend-architecture
summary: "A microservices data pattern where each service exclusively owns and manages its own database, eliminating shared-schema coupling and enabling independent scaling, technology choice, and deployment."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Database per Service

> A microservices data pattern where each service exclusively owns and manages its own database, eliminating shared-schema coupling and enabling independent scaling, technology choice, and deployment.

## Overview

In a microservices architecture, sharing a database between services creates tight coupling — schema changes in one service break others, performance issues in one service degrade all, and services can't be deployed independently. **Database per Service** mandates that each service has exclusive access to its own data store. Other services cannot query it directly — they must go through the service's API.

This is a prerequisite for true microservice independence, not an optional enhancement.

## The Problem with Shared Databases

```
❌ Shared DB Anti-Pattern:
   Order Service  ─────┐
   Payment Service ─────┤──► Shared PostgreSQL
   Inventory Service ───┘

Problems:
  - Schema migration for Order Service can break Inventory queries
  - Payment Service can JOIN directly into Order tables (hidden coupling)
  - Can't scale Order Service DB independently
  - Can't swap Inventory to a different DB type without affecting others
```

## Database Per Service in Practice

```
✅ Database Per Service:
   Order Service → OrderDB (PostgreSQL — transactional)
   Payment Service → PaymentDB (PostgreSQL — PCI-isolated)
   Inventory Service → InventoryDB (Redis + PostgreSQL — fast reads)
   Search Service → SearchDB (Elasticsearch — full-text search)
   Analytics Service → AnalyticsDB (Columnar — BigQuery/Redshift)
```

Each service picks the right tool for its data access patterns — **polyglot persistence**.

## Data Ownership Rules

1. **Only the owning service reads/writes its database** — no exceptions
2. **Other services access data through APIs** (REST, gRPC, GraphQL, events)
3. **No cross-service database JOINs** — ever
4. **No shared ORM entities** across service boundaries

## Cross-Service Data Access Patterns

### API Composition
The most straightforward approach — aggregate data by calling multiple services:

```python
# API Gateway / BFF aggregates data
async def get_order_details(order_id: str):
    order = await order_service.get_order(order_id)           # Order DB
    customer = await customer_service.get(order.customer_id)  # Customer DB
    items = await inventory_service.get_items(order.item_ids) # Inventory DB
    return OrderDetailsResponse(order, customer, items)
```

### Event-Driven Data Replication
Services publish events; consuming services maintain their own read-optimized copy:

```
Order Service publishes: OrderPlaced { orderId, customerId, items, total }
                            ↓
Search Service consumes: Updates its Elasticsearch index with order data
Reporting Service consumes: Appends to its columnar store
```

**Trade-off**: Eventual consistency — the replica may be slightly behind the source of truth.

### CQRS with Read Models
Command goes to the write service; queries are served from locally replicated read models:

```java
// Order Service publishes events
public class Order {
    public List<DomainEvent> place(OrderCommand cmd) {
        // ... validate ...
        return List.of(new OrderPlaced(id, cmd.customerId(), cmd.items()));
    }
}

// Reporting Service maintains its own denormalized read model
@EventHandler
public void on(OrderPlaced event) {
    reportingRepository.save(OrderSummary.builder()
        .orderId(event.orderId())
        .customerName(event.customerName())  // denormalized — no join needed
        .totalAmount(event.total())
        .build());
}
```

### Saga Pattern (Distributed Transactions)
For operations spanning multiple services, use Sagas — a sequence of local transactions with compensating actions:

```
Place Order Saga (Choreography):
  1. Order Service: Creates order (PENDING) → publishes OrderCreated
  2. Inventory Service: Reserves stock → publishes StockReserved
  3. Payment Service: Charges card → publishes PaymentCompleted
  4. Order Service: Confirms order (CONFIRMED)

Compensation (if payment fails):
  4. Payment Service publishes PaymentFailed
  3. Inventory Service receives PaymentFailed → releases stock → publishes StockReleased
  2. Order Service receives StockReleased → cancels order
```

## Eventual Consistency

With separate databases, cross-service data is **eventually consistent** (not immediately consistent). Design implications:

```
User sees order as CONFIRMED in Order Service
At the same millisecond, Reporting Service may still show it as PENDING
→ This is acceptable for most read models

NOT acceptable for: financial transfers, inventory reservation (use Sagas instead)
```

**Design for eventual consistency**:
- Communicate to users when data may lag ("processing...")
- Use idempotent event handlers (replay safety)
- Include timestamps; consumers can detect stale data

## Schema Migration Strategy

Each service owns its migration independently:

```
Order Service:
  migrations/
    V1__create_orders.sql
    V2__add_shipping_address.sql
    V3__add_discount_column.sql  ← only affects Order Service
```

Tools: Flyway, Liquibase, Alembic. Run as part of service deployment. No coordination with other teams.

## Real-World Examples

**Netflix**: Each of their 700+ microservices owns its data. Content metadata in Cassandra, user preferences in EVCache, billing in MySQL, recommendations in specialized stores. Schema evolution happens per service without cross-team coordination.

**Uber**: Trip service owns trip data (Schemaless/MySQL), payment service owns payment records (isolated PCI-compliant DB), driver service owns driver profiles. Cross-service queries are replaced by event-driven read models.

**Amazon**: Services own their data. To get a product price, the Order service calls the Catalog service API — it never JOINs into the catalog database.

**Shopify**: In their modular monolith, each module uses a separate PostgreSQL schema. When extracting to microservices, the module gets its own DB server — schema ownership was already established.

## Database Technology Choices by Service Type

| Service Type | Recommended DB | Why |
|-------------|---------------|-----|
| Transactional (orders, payments) | PostgreSQL, MySQL | ACID transactions |
| User sessions | Redis | Fast TTL-based access |
| Product search | Elasticsearch | Full-text, faceted search |
| Time-series metrics | InfluxDB, TimescaleDB | Optimized time-range queries |
| Social graph | Neo4j, Amazon Neptune | Relationship traversal |
| Document storage | MongoDB, DynamoDB | Flexible schema, horizontal scale |
| Analytics/reporting | BigQuery, Redshift | Columnar scan performance |

## Trade-offs

| Pros | Cons |
|------|------|
| True service independence | No cross-service JOINs — complex queries require composition |
| Polyglot persistence — right tool per service | Eventual consistency complexity |
| Independent schema evolution | Data duplication across services |
| Independent scaling per service | Distributed transactions require Sagas |
| Security isolation (PCI, HIPAA boundaries) | Higher operational overhead (N databases to manage) |

## Anti-Patterns

- **Shared database schema with different apps** — the worst of both worlds
- **Direct DB access via a "shared library"** — bypasses API contract
- **Synchronous cross-service DB calls** — creates distributed monolith
- **Overly granular services with chatty API calls** — 10 service calls to render one page

## When to Use

✅ **Use when**: True microservices architecture; services with different scaling/storage requirements; strong team ownership boundaries; compliance isolation requirements (PCI, HIPAA).

❌ **Avoid when**: Modular monolith (use schema-per-module instead); services that are truly always deployed/scaled together; small teams where operational overhead is prohibitive.

---
*Related: [[Domain-Driven Design]], [[Event-Driven Architecture]], [[Modular Monolith]], [[Strangler Fig Pattern]]*
