---
title: "Rate Limiter"
category: system-design
summary: "A rate limiter is a system component that controls traffic rates sent by clients or services to prevent abuse, reduce costs, and ensure server stability. It can be implemented using various algorithms like token bucket, leaking bucket, or sliding window approaches."
sources:
  - raw/articles/_done/rate-limiter-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:00:34.026Z
---

# Rate Limiter

> A rate limiter is a system component that controls traffic rates sent by clients or services to prevent abuse, reduce costs, and ensure server stability. It can be implemented using various algorithms like token bucket, leaking bucket, or sliding window approaches.

# Rate Limiter

A rate limiter is a critical system component used to control the rate of traffic sent by clients or services. It serves as a protective mechanism that prevents system abuse, reduces operational costs, and ensures server stability by filtering excessive requests.

## Benefits

- **DoS Attack Prevention**: Blocks excess calls to avoid resource starvation
- **Cost Reduction**: Limits unnecessary requests to reduce server expenses
- **Overload Prevention**: Filters excessive requests to stabilize server performance

## Implementation Placement

Rate limiters can be implemented in three main locations:

- **Client-Side**: Unreliable due to potential misuse and lack of control
- **Server-Side**: Preferred approach for maximum control and reliability
- **[[API Gateway]]**: Flexible middleware option for integrated rate limiting in [[Microservices Architecture]]

## Rate Limiting Algorithms

### Token Bucket
Tokens are added to a bucket at a fixed rate; each request consumes a token. Supports traffic bursts but requires careful parameter tuning.

### Leaking Bucket
Processes requests at a fixed rate using a FIFO queue. Provides stable outflow but may delay recent requests during traffic bursts.

### Fixed Window Counter
Divides time into fixed intervals with counters to limit requests. Simple but vulnerable to traffic spikes at window edges.

### Sliding Window Log
Tracks timestamps for accurate rolling time windows. Highly accurate but memory-intensive.

### Sliding Window Counter
Combines fixed window and sliding log methods. Memory-efficient and handles bursts well through approximation.

## Architecture

Rate limiters typically use in-memory [[Caching]] solutions like Redis for fast counter operations. The middleware checks counters before processing requests, either allowing or rejecting them based on configured limits.

## Distributed Challenges

In distributed environments, rate limiters face race conditions and synchronization issues. Solutions include using locks, Lua scripts, or [[Redis Sorted Sets]] with centralized data stores for coordination.

---
*Related: [[API Gateway]], [[Microservices Architecture]], [[Caching]], [[Redis Sorted Sets]], [[System Scaling]]*
