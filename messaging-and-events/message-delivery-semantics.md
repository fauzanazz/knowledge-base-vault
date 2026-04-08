---
title: "Message Delivery Semantics"
category: system-design
summary: "Message delivery semantics define guarantees about how many times messages are delivered in distributed systems, with three main types: at-most-once, at-least-once, and exactly-once, each offering different trade-offs between performance and reliability."
sources:
  - raw/articles/distributed-message-queue-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:15:21.490Z
---

# Message Delivery Semantics

> Message delivery semantics define guarantees about how many times messages are delivered in distributed systems, with three main types: at-most-once, at-least-once, and exactly-once, each offering different trade-offs between performance and reliability.

# Message Delivery Semantics

Message delivery semantics define the guarantees about how many times messages are delivered between producers and consumers in distributed messaging systems.

## At-Most-Once Delivery

Messages are delivered **zero or one time** - never more than once, but may not be delivered at all.

### Implementation
- **Producer**: Sends messages asynchronously without retries on failure
- **Consumer**: Commits offset immediately after fetching, before processing
- **Acknowledgment**: No confirmation required (`ACK=0`)

### Characteristics
- **Lowest latency**: Fastest message delivery
- **Lowest durability**: Messages may be lost
- **Use cases**: Metrics collection, logging where occasional loss is acceptable

## At-Least-Once Delivery

Messages are delivered **one or more times** - guaranteed delivery but possible duplicates.

### Implementation
- **Producer**: Uses `ACK=1` or `ACK=all` with retry logic on failures
- **Consumer**: Commits offset only after successful message processing
- **Failure handling**: Retries ensure message delivery

### Characteristics
- **Balanced approach**: Good durability with reasonable performance
- **Duplicate handling**: Applications must handle potential duplicates
- **Use cases**: Financial transactions, order processing with deduplication

## Exactly-Once Delivery

Messages are delivered **exactly one time** - no loss, no duplicates.

### Implementation Challenges
- **Distributed coordination**: Requires complex state management
- **Performance cost**: Significant overhead for coordination
- **Idempotency**: Consumers must be idempotent or use transactional processing

### Characteristics
- **Highest guarantee**: Perfect delivery semantics
- **Highest cost**: Most expensive to implement and operate
- **Use cases**: Critical financial systems, regulatory compliance scenarios

## Configuration Factors

### Producer Acknowledgment Settings
- **ACK=0**: Fire-and-forget (at-most-once)
- **ACK=1**: Leader acknowledgment (at-least-once)
- **ACK=all**: All replicas acknowledgment (strongest durability)

### Consumer Offset Management
- **Auto-commit**: Automatic offset updates (at-most-once tendency)
- **Manual commit**: Application-controlled offset updates (at-least-once)
- **Transactional**: Coordinated processing and offset commits (exactly-once)

## Trade-offs

### Performance vs. Reliability
- **At-most-once**: Highest performance, lowest reliability
- **At-least-once**: Balanced performance and reliability
- **Exactly-once**: Lowest performance, highest reliability

### Implementation Complexity
- **At-most-once**: Simplest implementation
- **At-least-once**: Moderate complexity with retry logic
- **Exactly-once**: Most complex with distributed coordination

## Choosing the Right Semantic

Consider these factors:
- **Data criticality**: How important is message loss vs. duplication?
- **Performance requirements**: What latency and throughput are needed?
- **Application design**: Can the system handle duplicates or missing messages?
- **Operational complexity**: What level of system complexity is acceptable?

Most [[Message Queue]] systems default to at-least-once delivery as it provides good balance between reliability and performance while being implementable with reasonable complexity.

---
*Related: [[Message Queue]], [[Database Replication]], [[System Scaling]]*
