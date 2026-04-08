---
title: "Back-of-the-Envelope Estimation"
category: system-design
summary: "Back-of-the-envelope estimation is a crucial skill for system design interviews that involves making quick, rough calculations to assess system capacity and performance requirements."
sources:
  - raw/articles/_done/back-of-the-envelope-estimation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:00:19.070Z
---

# Back-of-the-Envelope Estimation

> Back-of-the-envelope estimation is a crucial skill for system design interviews that involves making quick, rough calculations to assess system capacity and performance requirements.

# Back-of-the-Envelope Estimation

Back-of-the-envelope estimation is a fundamental skill in system design interviews that involves making quick, rough calculations to evaluate whether system designs meet requirements. According to Jeff Dean, Google Senior Fellow, these estimates help assess system capacity and performance through thought experiments and common performance benchmarks.

## Key Concepts

### Power of Two
Understanding data volume in terms of powers of two is essential for accurate storage and bandwidth calculations. This knowledge forms the foundation for capacity planning and resource estimation.

### Latency Numbers
Critical latency benchmarks every programmer should know:
- L1 Cache Access: 0.5 ns
- L2 Cache Access: 7 ns
- Main Memory Access: 100 ns
- SSD Random Read: 150 µs
- HDD Random Seek: 10 ms
- Round-Trip in Data Center: 500 µs
- Inter-Region Data Center: 150 ms

Key insights: memory is fast, disk is slow, avoid disk seeks, and compress data before internet transmission.

### Availability Numbers
High availability is expressed in "nines":
- 99% (Two Nines): ~3.65 days/year downtime
- 99.9% (Three Nines): ~8.8 hours/year downtime
- 99.99% (Four Nines): ~52 minutes/year downtime
- 99.999% (Five Nines): ~5.3 minutes/year downtime

Cloud providers typically aim for SLAs of 99.9% or higher.

## Estimation Process

Effective estimation requires:
1. **Rounding and Approximation**: Focus on process over precision
2. **Clear Assumptions**: Document all assumptions for reference
3. **Labeled Units**: Avoid ambiguity with proper unit labeling
4. **Common Scenarios**: Practice with [[QPS]], peak traffic, storage, cache, and server requirements

## Example: Twitter Scale
For a system with 300M monthly active users:
- Daily Active Users: 150M (50% of MAU)
- Average QPS: ~3,500 (150M × 2 tweets ÷ 86,400 seconds)
- Peak QPS: ~7,000 (2× average)
- Daily media storage: 30TB (10% of tweets with 1MB media)
- 5-year storage: ~55PB

This systematic approach enables quick yet reasonably accurate capacity planning for large-scale systems.

---
*Related: [[System Design Interview Framework]], [[System Scaling]], [[Load Balancer]], [[Caching]]*
