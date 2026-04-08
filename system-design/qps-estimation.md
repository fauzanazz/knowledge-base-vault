---
title: "QPS Estimation"
category: system-design
summary: "QPS (Queries Per Second) estimation calculates system traffic intensity and peak load requirements for capacity planning in distributed systems."
sources:
  - raw/articles/back-of-the-envelope-estimation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:35:08.341Z
---

# QPS Estimation

> QPS (Queries Per Second) estimation calculates system traffic intensity and peak load requirements for capacity planning in distributed systems.

# QPS Estimation

QPS (Queries Per Second) estimation is a critical component of [[Back-of-the-Envelope Estimation]] that measures system traffic intensity and helps determine infrastructure requirements for distributed systems.

## Basic QPS Calculation

QPS is calculated using daily active users (DAU) and average operations per user:

```
QPS = (DAU × Operations per user per day) / (24 hours × 3600 seconds)
```

## Peak QPS Considerations

Peak QPS accounts for traffic spikes and is typically calculated as:
- **Peak QPS = 2 × Average QPS** (common approximation)
- Consider time zones, events, and usage patterns
- Plan for seasonal variations and viral content

## Example: Twitter QPS Estimation

Given assumptions:
- 300 million monthly active users (MAU)
- 50% daily active users (DAU = 150M)
- Average 2 tweets per user per day

Calculation:
- **Average QPS:** (150M × 2) / (24 × 3600) = ~3,500 QPS
- **Peak QPS:** 2 × 3,500 = ~7,000 QPS

## Read vs Write QPS

Different operations have different QPS patterns:
- **Read QPS:** Usually much higher than write QPS
- **Write QPS:** Often the bottleneck for system design
- **Read/Write Ratio:** Commonly 100:1 or 1000:1 for social media

## Infrastructure Planning

QPS estimates help determine:
- Number of servers needed
- [[Load Balancer]] requirements
- [[Database Sharding]] strategies
- [[Caching]] layer sizing
- [[Content Delivery Network]] capacity

Accurate QPS estimation is fundamental for designing systems that can handle expected traffic loads while maintaining performance and reliability.

---
*Related: [[Back-of-the-Envelope Estimation]], [[System Scaling]], [[Load Balancer]], [[Database Sharding]]*
