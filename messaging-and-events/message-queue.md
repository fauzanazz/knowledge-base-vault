---
title: "Message Queue"
category: system-design
summary: "A message queue is a durable component that enables asynchronous communication between system components through temporary message buffering. It provides various messaging patterns and delivery guarantees for decoupled system architectures."
sources:
  - raw/articles/scaling-system-design-interview-by-alex-xu-pagefy.md
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:12:09.754Z
---

# Message Queue

> A message queue is a durable component that enables asynchronous communication between system components through temporary message buffering. It provides various messaging patterns and delivery guarantees for decoupled system architectures.

# Message Queue

A message queue is a durable component that enables asynchronous communication between system components. It serves as a buffer that stores messages in memory and allows producers to send messages that consumers can process independently.

## Core Concepts

A **message channel** acts as a temporary buffer between sender and receiver. Messages have well-defined formats with headers and payloads (e.g., JSON) and can be:
- **Commands**: Meant for processing by workers
- **Events**: Notifications of interesting occurrences

Services use inbound and outbound adapters to send/receive messages from channels.

## Benefits of Decoupling

- **Availability**: Producers can send messages even when consumers are temporarily unavailable
- **Load Balancing**: Requests distribute across consumer instance pools for easy scaling
- **Load Smoothing**: Consumers read at their own pace, smoothing out traffic spikes
- **Batching**: Clients can submit single requests for multiple messages, optimizing throughput

## Messaging Patterns

### One-Way Messaging (Point-to-Point)
Messages are received and processed by exactly one consumer. Useful for job processing workflows.

### Request-Response Messaging
Similar to direct request-response but flows through channels. Uses response channels and response IDs to associate responses with original requests.

### Broadcast Messaging (Publish-Subscribe)
Producers write to publish-subscribe channels to broadcast messages to all consumers. Used for event notifications without tight coupling.

## Message Broker Guarantees

Different message brokers (Amazon SQS, Kafka) offer different trade-offs:

- **Message Ordering**: Kafka partitions messages and allows only single consumers per partition to guarantee order
- **Delivery Guarantees**: At-most-once vs at-least-once delivery
- **Message Durability**: Persistence guarantees
- **Latency**: Response time characteristics
- **Broker Limits**: Maximum message sizes and throughput

## Processing Guarantees

Consumers must delete messages after processing, but this creates timing challenges:
- Delete before processing: Risk of message loss if processing fails
- Delete after processing: Risk of duplicate processing if deletion fails

**Exactly-once processing** is impossible to guarantee in distributed systems. Systems typically implement **at-least-once** delivery with **idempotent** consumers that can safely process duplicate messages.

## Implementation Considerations

Message channels introduce complexity through additional services and non-typical control flow. However, they provide essential decoupling for scalable, resilient distributed systems.

---
*Related: [[Microservices Architecture]], [[API Gateway]], [[Distributed Systems]], [[Load Balancer]], [[Eventual Consistency]]*
