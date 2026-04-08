---
title: "API Idempotency"
category: api-design
summary: "API idempotency ensures that executing the same request multiple times produces the same result as executing it once, enabling safe retries and robust client implementations."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.260Z
---

# API Idempotency

> API idempotency ensures that executing the same request multiple times produces the same result as executing it once, enabling safe retries and robust client implementations.

# API Idempotency

Idempotency is a critical property for robust APIs where executing the same request multiple times produces the same result as executing it once. This enables safe retries when requests timeout or fail.

## HTTP Method Idempotency

HTTP methods have different idempotency characteristics:
- **GET, PUT, DELETE**: Naturally idempotent
- **POST**: Not idempotent by default, requires special handling

## Idempotency Keys

For non-idempotent operations like POST, idempotency can be achieved using client-generated keys:

1. **Client generates UUID**: Included in `Idempotency-Key` header
2. **Server checks database**: If key exists, return cached response
3. **Server executes request**: Store key and result atomically
4. **Key cleanup**: Remove keys after TTL expires

## Implementation Challenges

**Atomicity**: Storing the idempotency key and executing the request must be atomic to prevent inconsistent states if the server crashes between operations.

**ACID databases**: Easy to achieve with transaction guarantees
**External systems**: More challenging when request involves external state changes

## Edge Cases

Consider this scenario:
1. Client A creates resource, request fails but resource is created
2. Client B deletes the resource
3. Client A retries with same idempotency key

The least surprising behavior is to return success, maintaining the illusion that the original request succeeded.

## Benefits

Idempotent APIs enable:
- **Robust clients**: Safe retry logic without side effects
- **Better user experience**: Reduced duplicate operations
- **System reliability**: Graceful handling of network failures

Idempotency is essential for building reliable [[REST APIs]] in [[Distributed Systems]] where network failures are common and retries are necessary for resilience.

---
*Related: [[REST APIs]], [[HTTP Protocol]], [[System Resiliency]], [[API Design]], [[Database Transactions]]*
