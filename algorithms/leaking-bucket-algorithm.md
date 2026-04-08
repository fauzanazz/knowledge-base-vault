---
title: "Leaking Bucket Algorithm"
category: algorithms
summary: "The leaking bucket algorithm processes requests at a fixed rate using a FIFO queue, providing stable outflow rates for rate limiting. It smooths traffic bursts but may delay recent requests during high traffic periods."
sources:
  - raw/articles/rate-limiter-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:35:50.022Z
---

# Leaking Bucket Algorithm

> The leaking bucket algorithm processes requests at a fixed rate using a FIFO queue, providing stable outflow rates for rate limiting. It smooths traffic bursts but may delay recent requests during high traffic periods.

# Leaking Bucket Algorithm

The leaking bucket algorithm is a [[Rate Limiter]] technique that processes requests at a constant, fixed rate regardless of the input rate. It uses a FIFO (First-In-First-Out) queue to buffer incoming requests and processes them at a steady pace.

## How It Works

1. **Request Queuing**: Incoming requests are added to a FIFO queue
2. **Fixed Processing**: Requests are processed from the queue at a constant rate
3. **Queue Overflow**: When the queue is full, new requests are rejected
4. **Steady Outflow**: Output rate remains constant regardless of input spikes

## Key Characteristics

- **Constant Output Rate**: Processes requests at a predetermined fixed rate
- **Queue-Based**: Uses a buffer to handle temporary traffic spikes
- **FIFO Processing**: Maintains request order through first-in-first-out processing

## Advantages

- **Memory Efficient**: Simple queue structure with minimal overhead
- **Stable Outflow**: Guarantees consistent processing rate
- **Predictable Performance**: Output rate is always constant and predictable

## Disadvantages

- **Burst Delays**: Traffic bursts may cause delays for recent requests
- **Queue Latency**: Requests may experience variable waiting times
- **Less Flexible**: Cannot accommodate legitimate traffic spikes

## Comparison with Token Bucket

Unlike the [[Token Bucket Algorithm]] which allows bursts, leaking bucket enforces strict rate limits. Token bucket is better for systems that need to handle legitimate traffic spikes, while leaking bucket is ideal for systems requiring consistent, predictable processing rates.

## Implementation Example

Uber's rate limiting library (uber-go/ratelimit) implements a leaking bucket approach for consistent request processing in their microservices architecture.

---
*Related: [[Rate Limiter]], [[Token Bucket Algorithm]], [[Message Queue]]*
