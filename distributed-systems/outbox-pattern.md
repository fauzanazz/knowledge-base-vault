---
title: "Outbox Pattern"
category: distributed-systems
summary: "The Outbox pattern ensures consistency between multiple data stores by using a local outbox table and asynchronous message processing to guarantee reliable data synchronization."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.481Z
---

# Outbox Pattern

> The Outbox pattern ensures consistency between multiple data stores by using a local outbox table and asynchronous message processing to guarantee reliable data synchronization.

# Outbox Pattern

The Outbox pattern solves the common problem of maintaining consistency when persisting data across multiple data stores, such as a primary database and a search index like Elasticsearch.

## Problem Statement

When applications need to update multiple unrelated services or data stores, ensuring consistency becomes challenging:
- Direct updates to multiple systems can fail partially
- Network failures can leave systems in inconsistent states
- Traditional distributed transactions may be too heavyweight or unavailable

## Solution Approach

**Atomic Local Write**: Instead of directly updating multiple systems, the application performs a single atomic transaction within its primary database:
1. Save the main data to the designated table
2. Save a corresponding message to a special "outbox" table
3. Both operations occur within the same local transaction

**Asynchronous Processing**: A separate process periodically:
1. Queries the outbox table for pending messages
2. Sends messages to the secondary data store (e.g., Elasticsearch)
3. Marks messages as processed or removes them from the outbox

## Implementation Details

**Message Channel Integration**: The pattern often uses reliable [[Message Queue|message channels]] like Kafka to:
- Achieve idempotency through message deduplication
- Guarantee delivery through retry mechanisms
- Provide ordering guarantees when needed

**Exactly-Once Processing**: The outbox processor ensures each message is processed exactly once, preventing duplicate updates to secondary systems.

## Benefits

**Consistency**: Guarantees that if the primary data is saved, the corresponding message will eventually be processed.

**Reliability**: Uses the database's ACID properties to ensure atomic writes to both main and outbox tables.

**Decoupling**: Secondary systems can be temporarily unavailable without affecting primary operations.

**Scalability**: Asynchronous processing allows the system to handle high write volumes without blocking primary operations.

The Outbox pattern provides a robust foundation for building eventually consistent distributed systems without the complexity of distributed transactions.

---
*Related: [[Message Queue]], [[Database Transactions]], [[Eventual Consistency]], [[Saga Pattern]]*
