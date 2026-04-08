---
title: "Market Data Publisher"
category: system-design
summary: "A market data publisher receives execution streams from trading engines and constructs real-time market data including order books and candlestick charts. It distributes this data to subscribers using optimized data structures and multicast protocols."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:20:49.881Z
---

# Market Data Publisher

> A market data publisher receives execution streams from trading engines and constructs real-time market data including order books and candlestick charts. It distributes this data to subscribers using optimized data structures and multicast protocols.

# Market Data Publisher

A market data publisher is a component in [[Stock Exchange System Design]] that processes execution streams and generates real-time market data for distribution to traders and market participants.

## Core Responsibilities

**Data Reconstruction:**
- Receives execution events from the [[Matching Engine]]
- Rebuilds order books from execution streams
- Constructs candlestick charts from trade data
- Maintains Level 1, 2, and 3 market data

**Data Distribution:**
- Publishes market data to subscribers in real-time
- Ensures fair distribution timing to prevent market manipulation
- Supports different data granularities based on subscription tiers

## Market Data Types

**Level 1 (L1):** Best bid/ask prices and quantities
**Level 2 (L2):** Multiple price levels with depth information
**Level 3 (L3):** Full order book with individual order details

**Candlestick Data:**
- Open, high, low, close prices for time intervals
- Trading volume for each period
- Configurable time windows (1min, 5min, 1hour, etc.)

## Performance Optimizations

**Ring Buffers:**
- Pre-allocated circular buffers for candlestick storage
- Lock-free data structures for high-frequency updates
- Cache padding to prevent false sharing

**Memory Management:**
- Limited in-memory data retention
- Automatic persistence to disk for historical data
- Columnar database storage (e.g., KDB) for analytics

## Distribution Fairness

**Multicast Protocol:**
- Uses reliable UDP multicast for simultaneous delivery
- Ensures all subscribers receive data at the same time
- Prevents information asymmetry in the market

**Colocation Services:**
- Offers low-latency data feeds for colocated servers
- Provides VIP service for institutional clients
- Reduces network latency through proximity

## Data Structure Implementation

```
class Candlestick {
  private long openPrice, closePrice;
  private long highPrice, lowPrice;
  private long volume, timestamp;
  private int interval;
}
```

The publisher maintains separate data streams for different subscriber types and implements sophisticated caching strategies to handle varying data freshness requirements across different market participants.

---
*Related: [[Stock Exchange System Design]], [[Matching Engine]], [[Order Book]], [[Market Data]], [[Multicast Protocol]]*
