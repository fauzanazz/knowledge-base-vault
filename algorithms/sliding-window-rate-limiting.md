---
title: "Sliding Window Rate Limiting"
category: algorithms
summary: "Sliding window rate limiting uses rolling time windows to provide more accurate rate limiting than fixed windows. It includes sliding window log and sliding window counter approaches, each with different memory and accuracy trade-offs."
sources:
  - raw/articles/rate-limiter-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:35:50.023Z
---

# Sliding Window Rate Limiting

> Sliding window rate limiting uses rolling time windows to provide more accurate rate limiting than fixed windows. It includes sliding window log and sliding window counter approaches, each with different memory and accuracy trade-offs.

# Sliding Window Rate Limiting

Sliding window rate limiting is a sophisticated [[Rate Limiter]] approach that uses rolling time windows to provide more accurate request throttling compared to fixed window methods. It addresses the edge case problems inherent in fixed window counters.

## Sliding Window Log

### How It Works
- **Timestamp Tracking**: Maintains timestamps of all requests within the time window
- **Rolling Window**: Continuously slides the time window as new requests arrive
- **Precise Counting**: Counts only requests within the exact time window

### Characteristics
- **High Accuracy**: Provides precise rate limiting without edge case issues
- **Memory Intensive**: Stores individual timestamps for each request
- **Complex Implementation**: Requires timestamp management and cleanup

## Sliding Window Counter

### How It Works
- **Hybrid Approach**: Combines fixed window counters with sliding window logic
- **Approximation**: Uses weighted calculations based on current and previous windows
- **Smoothing**: Reduces traffic spikes at window boundaries

### Formula
Requests in current window = (Requests in current window) + (Requests in previous window × overlap percentage)

### Characteristics
- **Memory Efficient**: Only stores counters, not individual timestamps
- **Good Approximation**: Balances accuracy with resource efficiency
- **Burst Handling**: Better manages traffic spikes than fixed windows

## Advantages Over Fixed Windows

- **Edge Case Prevention**: Eliminates the problem where traffic spikes at window edges exceed limits
- **Smoother Rate Limiting**: Provides more consistent throttling behavior
- **Better User Experience**: Reduces sudden request rejections

## Trade-offs

**Sliding Window Log**: Maximum accuracy but high memory usage
**Sliding Window Counter**: Good balance of accuracy and efficiency through approximation

Sliding window approaches are ideal for systems requiring precise rate limiting without the memory overhead of tracking every individual request.

---
*Related: [[Rate Limiter]], [[Token Bucket Algorithm]], [[Caching]]*
