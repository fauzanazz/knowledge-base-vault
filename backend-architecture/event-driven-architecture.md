---
title: "Event-Driven Architecture"
category: backend-architecture
summary: "An architectural paradigm where services communicate through the production, detection, and consumption of events — enabling loose coupling, temporal decoupling, and reactive system design at scale."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Event-Driven Architecture

> An architectural paradigm where services communicate through the production, detection, and consumption of events — enabling loose coupling, temporal decoupling, and reactive system design at scale.

## Overview

In Event-Driven Architecture (EDA), components communicate by emitting and reacting to **events** — immutable records of things that happened. Services are decoupled: producers don't know who consumes their events; consumers don't know who produced them. This enables independent scaling, deployment, and evolution of services.

EDA is foundational to Kafka-based architectures (LinkedIn, Uber, Netflix), event sourcing systems, and real-time data pipelines.

## Core Concepts

### Events
An event is an immutable record of something that occurred in the domain, named in the **past tense**:
```json
{
  "eventType": "OrderPlaced",
  "eventId": "evt_8x2kp9",
  "timestamp": "2026-04-08T11:23:45Z",
  "version": "1.0",
  "payload": {
    "orderId": "ord_xyz789",
    "customerId": "cust_abc123",
    "items": [{"productId": "p1", "qty": 2, "price": 29.99}],
    "totalAmount": 59.98
  }
}
```

**Event vs. Command**: A Command is an intent ("PlaceOrder"). An Event is a fact ("OrderPlaced"). Events cannot be rejected by consumers.

### Event Channels / Brokers
Infrastructure that stores and routes events:
- **Apache Kafka**: Persistent, ordered, replayable streams — the industry standard
- **AWS Kinesis**: Managed Kafka-like streams on AWS
- **Google Pub/Sub**: Managed, globally distributed
- **RabbitMQ**: Message broker (queues, not streams — events consumed once)
- **AWS EventBridge**: Serverless event bus with schema registry
- **NATS**: Lightweight, high-performance messaging

### Producers
Services that publish events without knowledge of consumers:
```java
@Service
public class OrderService {
    private final KafkaTemplate<String, OrderPlacedEvent> kafkaTemplate;

    public Order placeOrder(PlaceOrderCommand cmd) {
        Order order = createOrder(cmd);
        orderRepository.save(order);

        // Publish event — don't care who consumes it
        kafkaTemplate.send("orders.placed",
            order.getId().toString(),
            new OrderPlacedEvent(order));

        return order;
    }
}
```

### Consumers
Services that subscribe to event streams and react:
```java
@KafkaListener(topics = "orders.placed", groupId = "notification-service")
public void onOrderPlaced(OrderPlacedEvent event) {
    emailService.sendOrderConfirmation(event.getCustomerId(), event.getOrderId());
}

@KafkaListener(topics = "orders.placed", groupId = "inventory-service")
public void onOrderPlaced(OrderPlacedEvent event) {
    inventoryService.reserveItems(event.getItems());
}
```

Same event, different consumer groups — each consumer processes independently.

## Event Sourcing

Store the **full history of events** as the primary data, not current state. Current state is derived by replaying events.

```java
// Event store (append-only)
public class EventStore {
    void append(String streamId, List<DomainEvent> events, long expectedVersion);
    List<DomainEvent> load(String streamId, long fromVersion);
}

// Reconstruct aggregate from events
public class Order {
    private OrderStatus status;
    private List<OrderItem> items;

    public static Order reconstitute(List<DomainEvent> events) {
        Order order = new Order();
        events.forEach(order::apply);
        return order;
    }

    private void apply(DomainEvent event) {
        switch (event) {
            case OrderPlaced e -> { this.id = e.orderId(); this.status = PENDING; }
            case OrderConfirmed e -> this.status = CONFIRMED;
            case OrderShipped e -> this.status = SHIPPED;
            case OrderCancelled e -> this.status = CANCELLED;
        }
    }
}
```

**Benefits**: Complete audit trail, temporal queries ("what was the order state at 3pm?"), event replay for rebuilding projections, debugging.

## CQRS (Command Query Responsibility Segregation)

EDA pairs naturally with CQRS — commands modify state (publish events), queries read from projections:

```
Write Side:                    Read Side:
PlaceOrder Command             OrderSummaryQuery
    ↓                              ↑
Order Aggregate           Order Summary Projection
    ↓                              ↑
Events Published ──────► Event Handler updates projection DB
(Kafka/Event Store)       (Elasticsearch, Redis, Read DB)
```

```java
// Write model — event sourced
public class OrderCommandHandler {
    public void handle(PlaceOrderCommand cmd) {
        Order order = orderRepository.load(cmd.orderId()); // load from event store
        List<DomainEvent> events = order.place(cmd);
        orderRepository.save(events);  // append to event store
    }
}

// Read model — denormalized projection
@EventHandler
public void on(OrderPlaced event, OrderSummaryRepository repo) {
    repo.save(new OrderSummary(event.orderId(), event.customerName(),
        event.totalAmount(), event.timestamp()));
}
```

## Event Streaming at Scale: Apache Kafka

Kafka is the backbone of most large-scale EDA systems:

```
Topic: orders.events
Partitions: 32 (parallel processing by 32 consumers)
Retention: 7 days (replay last week of events)
Replication: 3 (fault tolerant)

Consumer Group: inventory-service (each instance gets some partitions)
Consumer Group: notification-service
Consumer Group: analytics-pipeline
```

**Kafka's guarantee**: Messages in a partition are ordered. Use the entity ID (order ID, user ID) as the partition key to ensure events for the same entity are processed in order.

## Real-World EDA at Scale

**LinkedIn**: Invented Kafka (2011). Processes 7 trillion messages/day. All activity events — profile views, job applications, connection requests — flow through Kafka topics consumed by multiple downstream systems.

**Uber**: Their "Fulfillment Platform" uses EDA — a trip lifecycle emits events (TripRequested, DriverMatched, TripStarted, TripCompleted) consumed by billing, analytics, notifications, and the driver earnings service independently.

**Netflix**: Uses Kafka for all viewing events. A user watching a show emits play/pause/stop events consumed by the recommendations engine, billing, analytics, and the "continue watching" feature — all decoupled.

**Shopify**: Processes flash sales by queuing order events in Kafka. During Black Friday, order events are enqueued when submitted and processed by downstream services (inventory, payment) asynchronously — allowing the storefront to accept orders faster than the processing pipeline.

## Event Schema Evolution

Events are a contract between producers and consumers. Use a **Schema Registry** (Confluent Schema Registry, AWS Glue) to manage evolution:

```
v1: { orderId, customerId, totalAmount }
v2: { orderId, customerId, totalAmount, discountCode }  ← added field
v3: { orderId, customerId, totalAmount, discountCode, taxAmount }

Rules:
✅ Adding optional fields: backward compatible
✅ Adding new event types: safe
❌ Removing fields: breaks old consumers
❌ Renaming fields: breaking change
```

## Trade-offs

| Pros | Cons |
|------|------|
| Loose coupling — producers don't know consumers | Eventual consistency — harder to reason about |
| Independent scaling of producers and consumers | Event ordering across partitions is tricky |
| Event replay for rebuilding projections | Schema evolution requires careful management |
| Natural audit trail | Debugging complex event chains is hard |
| High throughput (Kafka handles millions/sec) | Operational complexity (Kafka cluster management) |
| Temporal decoupling — consumers can be offline | Idempotency required in consumer logic |

## Anti-Patterns

- **Event-carried state transfer without versioning**: Consumers depend on specific event structure that silently changes
- **Massive events**: Putting entire object graphs in events — use event notifications + API call for large data
- **Synchronous response over events**: Using events for request/reply adds complexity; use REST/gRPC instead
- **No dead-letter queue**: Failed events silently lost — always configure DLQ for unprocessable messages

## When to Use

✅ **Use when**: Multiple consumers for the same domain events; audit logging requirements; event sourcing for complex domain history; high-throughput data pipelines; decoupled microservices evolution.

❌ **Avoid when**: Simple request/response patterns; strong consistency required; team lacks EDA/Kafka operational expertise; systems where event schema coordination is too costly.

---
*Related: [[Domain-Driven Design]], [[Database per Service]], [[Actor Model]], [[API Gateway Pattern]], [[Serverless Architecture]]*
