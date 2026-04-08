---
title: "Matching Engine"
category: system-design
summary: "A matching engine is the core component of a stock exchange that maintains order books and matches buy/sell orders. It must process orders deterministically with microsecond-level latency while ensuring fairness and accuracy."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:37:24.873Z
---

# Matching Engine

> A matching engine is the core component of a stock exchange that maintains order books and matches buy/sell orders. It must process orders deterministically with microsecond-level latency while ensuring fairness and accuracy.

# Matching Engine

A matching engine, also called a cross engine, is the heart of any [[Stock Exchange System Design]]. It's responsible for maintaining order books, matching buy and sell orders, and generating execution streams with deterministic behavior.

## Core Responsibilities

### Order Book Management
Maintains separate buy and sell order books for each trading symbol, organized by price levels. Each price level contains orders at that specific price, typically organized as a FIFO queue.

### Order Matching
Matches incoming orders against existing orders in the book using algorithms like:
- **FIFO (First In, First Out)**: Orders at the same price level are matched in arrival order
- **Pro-rata**: Distributes fills proportionally based on order size
- **Price-time priority**: Prioritizes better prices first, then time

### Execution Generation
When a match occurs, the engine generates two executions (fills) - one for the buy side and one for the sell side. These executions are sequenced to ensure deterministic processing.

## Performance Optimization

### Data Structures
Optimal order book implementation uses:
```
class OrderBook {
  private Map<Price, PriceLevel> buyLevels;
  private Map<Price, PriceLevel> sellLevels;
  private Map<OrderID, Order> orderMap; // O(1) lookup
  private PriceLevel bestBid;
  private PriceLevel bestOffer;
}
```

### Memory Management
- Pre-allocated ring buffers to avoid garbage collection
- Object pooling for order and execution objects
- Cache-friendly data layout with padding

### Processing Loop
Runs in a tight application loop pinned to a specific CPU core to avoid context switching and ensure consistent latency.

## Determinism

### Functional Determinism
Achieved through the [[Sequencer]] component that stamps each order with a sequence ID. Orders are processed strictly in sequence order, ensuring identical results regardless of timing.

### Latency Determinism
Monitored through 99th percentile latency metrics. Techniques to reduce variance include:
- Avoiding garbage collection in critical paths
- Using lock-free data structures
- Minimizing system calls

## Fault Tolerance

Matching engines maintain state through [[Event Sourcing]], allowing for:
- Fast recovery by replaying events from the last checkpoint
- Hot standby instances that process events but don't publish results
- Cross-data center replication for disaster recovery

## Integration

The matching engine integrates with other exchange components:
- Receives orders from the [[Order Manager]] via the sequencer
- Sends executions to the [[Market Data Publisher]] for order book reconstruction
- Provides execution streams to the reporting system

Modern matching engines can process millions of orders per second with sub-microsecond latency, making them critical for high-frequency trading environments.

---
*Related: [[Order Book]], [[Stock Exchange System Design]], [[Event Sourcing]], [[High Frequency Trading]]*
