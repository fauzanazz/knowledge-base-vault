---
title: "Event Sourcing"
category: system-design
summary: "Event sourcing is an architectural pattern that stores immutable state transitions as events rather than current state. It enables audit trails, time travel queries, and simplified debugging while supporting high-performance systems through event replay."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:37:24.874Z
---

# Event Sourcing

> Event sourcing is an architectural pattern that stores immutable state transitions as events rather than current state. It enables audit trails, time travel queries, and simplified debugging while supporting high-performance systems through event replay.

# Event Sourcing

Event sourcing is an architectural pattern where instead of storing the current state of entities, the system stores a sequence of immutable events that represent state changes over time. This approach provides complete audit trails and enables powerful debugging and recovery capabilities.

## Core Concepts

### Events vs State
**Traditional Approach:**
```
User: { id: 123, balance: 950, status: 'active' }
```

**Event Sourcing Approach:**
```
Events:
1. UserCreated { id: 123, initialBalance: 1000 }
2. MoneyWithdrawn { id: 123, amount: 50 }
3. AccountActivated { id: 123 }
```

### Event Store
A specialized database that:
- Stores events in append-only fashion
- Maintains event ordering through sequence numbers
- Supports efficient event replay
- Provides strong consistency guarantees

## Implementation in [[Stock Exchange System Design]]

### Order Lifecycle Events
```
OrderPlaced { orderId, symbol, price, quantity, timestamp }
OrderMatched { orderId, matchedQuantity, executionPrice }
OrderCanceled { orderId, reason }
OrderFilled { orderId, totalFilled }
```

### Event Processing
Components subscribe to event streams and maintain their own projections:
- **Order Manager**: Tracks order states
- **[[Matching Engine]]**: Maintains order books
- **Risk Manager**: Monitors exposure limits
- **Market Data**: Builds candlestick charts

## Benefits

### Audit Trail
Complete history of all state changes:
- Regulatory compliance for financial systems
- Debugging complex system behaviors
- Forensic analysis of incidents

### Time Travel
Query system state at any point in time:
```
// Get account balance as of specific date
balance = replayEvents(accountId, until: '2024-01-15')
```

### Simplified Recovery
Restore system state by replaying events:
- Fast recovery from failures
- Easy migration between systems
- Simplified backup strategies

## Performance Optimizations

### Snapshots
Periodically store state snapshots to avoid replaying all events:
```
Snapshot { timestamp, entityId, state }
+ Events since snapshot
= Current state
```

### Memory-Mapped Files
In high-performance systems like stock exchanges:
- Use mmap for zero-copy event access
- Store events in `/dev/shm` for memory-only operation
- Achieve microsecond-level latency

### Event Compaction
Remove obsolete events while preserving final state:
- Compact old events into snapshots
- Reduce storage requirements
- Maintain query performance

## Challenges

### Event Schema Evolution
Manage changes to event structure over time:
- Version events to handle schema changes
- Maintain backward compatibility
- Use event upcasting for migrations

### Query Complexity
Some queries become more complex:
- Aggregations across multiple entities
- Complex filtering and searching
- May require separate read models (CQRS)

### Storage Growth
Event stores grow continuously:
- Implement archival strategies
- Use compression for old events
- Consider event retention policies

## Integration Patterns

### CQRS (Command Query Responsibility Segregation)
Often combined with event sourcing:
- Commands modify state (generate events)
- Queries read from optimized projections
- Separate read and write models

### Saga Pattern
Manage distributed transactions:
- Break complex operations into events
- Handle compensation for failures
- Maintain consistency across services

Event sourcing is particularly valuable in financial systems, audit-heavy applications, and systems requiring high reliability and debugging capabilities.

---
*Related: [[CQRS]], [[Stock Exchange System Design]], [[Distributed Systems]], [[Audit Trail]], [[State Management]]*
