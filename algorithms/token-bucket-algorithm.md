---
title: "Token Bucket Algorithm"
category: algorithms
summary: "The token bucket algorithm is a rate limiting technique where tokens are added to a bucket at a fixed rate and each request consumes a token. It's memory-efficient and supports traffic bursts while maintaining overall rate limits."
sources:
  - raw/articles/rate-limiter-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:35:50.022Z
---

# Token Bucket Algorithm

> The token bucket algorithm is a rate limiting technique where tokens are added to a bucket at a fixed rate and each request consumes a token. It's memory-efficient and supports traffic bursts while maintaining overall rate limits.

# Token Bucket Algorithm

The token bucket algorithm is a popular [[Rate Limiter]] technique that controls request rates by using a bucket metaphor. Tokens are continuously added to a bucket at a predetermined rate, and each incoming request must consume a token to be processed.

## How It Works

1. **Token Generation**: Tokens are added to the bucket at a fixed refill rate
2. **Request Processing**: Each request consumes one token from the bucket
3. **Bucket Capacity**: The bucket has a maximum size that limits token accumulation
4. **Request Rejection**: Requests are rejected when no tokens are available

## Key Parameters

- **Bucket Size**: Maximum number of tokens the bucket can hold
- **Refill Rate**: Rate at which new tokens are added to the bucket

## Advantages

- **Easy Implementation**: Simple logic and straightforward to code
- **Memory Efficient**: Requires minimal storage (just token count and timestamp)
- **Burst Support**: Allows traffic bursts up to bucket capacity
- **Flexible**: Can handle varying traffic patterns effectively

## Disadvantages

- **Parameter Tuning**: Requires careful configuration of bucket size and refill rate
- **Burst Limitations**: Large bursts can quickly exhaust all tokens

## Use Cases

Token bucket is ideal for APIs that need to allow occasional traffic spikes while maintaining long-term rate limits. It's commonly used in cloud services, payment systems, and social media platforms where bursty traffic patterns are expected but overall throughput must be controlled.

---
*Related: [[Rate Limiter]], [[Leaking Bucket Algorithm]], [[System Design Interview Framework]]*
