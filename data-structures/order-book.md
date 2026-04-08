---
title: "Order Book"
category: data-structures
summary: "An order book is a data structure that maintains lists of buy and sell orders for a financial instrument, organized by price levels. It requires efficient operations for order placement, matching, and cancellation with constant-time complexity."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:37:24.874Z
---

# Order Book

> An order book is a data structure that maintains lists of buy and sell orders for a financial instrument, organized by price levels. It requires efficient operations for order placement, matching, and cancellation with constant-time complexity.

# Order Book

An order book is a fundamental data structure in financial trading systems that maintains lists of buy and sell orders for a specific instrument, organized by price levels. It's the core component that enables price discovery and order matching in electronic exchanges.

## Structure

### Price Levels
Orders are grouped by price into price levels:
```
class PriceLevel {
  private Price limitPrice;
  private long totalVolume;
  private List<Order> orders; // FIFO queue
}
```

### Dual-Sided Book
```
class OrderBook {
  private Book<Buy> buyBook;   // Bids
  private Book<Sell> sellBook; // Asks
  private PriceLevel bestBid;
  private PriceLevel bestOffer;
  private Map<OrderID, Order> orderMap;
}
```

## Key Operations

### Order Placement
- **Time Complexity**: O(1)
- New orders are added to the tail of the appropriate price level
- If price level doesn't exist, create new level

### Order Matching
- **Time Complexity**: O(1) per match
- Orders are matched from the head of the best price level
- Follows FIFO principle within each price level

### Order Cancellation
- **Time Complexity**: O(1)
- Uses orderMap for instant order lookup
- Doubly-linked list enables O(1) deletion

## Optimization Techniques

### Doubly-Linked Lists
Instead of standard lists, use doubly-linked lists for:
- O(1) insertion at tail (new orders)
- O(1) deletion from head (matching)
- O(1) deletion from middle (cancellation)

### Memory Management
- Pre-allocated ring buffers to avoid garbage collection
- Object pooling for orders and price levels
- Cache-line padding to prevent false sharing

## Market Data Levels

### Level 1 (L1)
Best bid/ask prices and quantities only:
- Bid: $100.50 (1000 shares)
- Ask: $100.52 (500 shares)

### Level 2 (L2)
Multiple price levels with aggregated quantities:
- Shows depth of market with 5-10 price levels
- Used by most retail trading platforms

### Level 3 (L3)
Complete order book with individual order details:
- Shows queue position and order sizes
- Typically restricted to market makers

## Implementation Considerations

### Price Representation
Use fixed-point arithmetic instead of floating-point to avoid precision issues:
```
// Store price as integer (cents)
long priceInCents = 10052; // $100.52
```

### Memory Layout
Optimize for cache efficiency:
- Group frequently accessed fields together
- Align data structures to cache line boundaries
- Use padding to prevent false sharing

## Integration with [[Matching Engine]]

The order book works closely with the matching engine:
- Provides efficient order lookup and modification
- Supports real-time market data generation
- Enables fast order matching algorithms

Order books are critical for maintaining fair and efficient markets, requiring careful optimization to handle the high-frequency trading demands of modern financial markets.

---
*Related: [[Matching Engine]], [[Stock Exchange System Design]], [[High Frequency Trading]], [[Market Data]]*
