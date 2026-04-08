---
title: "Exactly-Once Delivery"
category: system-design
summary: "Exactly-once delivery ensures operations are executed at-least-once and at-most-once simultaneously, preventing duplicate processing in distributed systems through retry mechanisms and idempotency."
sources:
  - raw/articles/_done/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:58:07.365Z
---

# Exactly-Once Delivery

> Exactly-once delivery ensures operations are executed at-least-once and at-most-once simultaneously, preventing duplicate processing in distributed systems through retry mechanisms and idempotency.

# Exactly-Once Delivery

**Exactly-once delivery** ensures that an operation is executed exactly one time, even in the presence of failures. This is achieved by combining at-least-once and at-most-once guarantees simultaneously.

## Core Components

### At-Least-Once Guarantee
Achieved through retry mechanisms with various strategies:
- **Immediate retry**: Send another request immediately after failure
- **Fixed intervals**: Wait a fixed time before retrying
- **Incremental intervals**: Gradually increase retry intervals
- **Exponential back-off**: Double retry interval between attempts (recommended default)
- **Cancel**: Stop retrying when errors are terminal or threshold reached

Servers should specify retry intervals using `Retry-After` headers for optimal client behavior.

### At-Most-Once Guarantee
Prevents duplicate processing through [[Idempotency]] mechanisms:
- Clients include `idempotency-key` headers (typically UUIDs)
- Servers use database unique constraints to detect duplicates
- Failed insertions due to constraint violations return existing results

## Payment System Applications

In [[Payment System]] designs, exactly-once delivery prevents:
- Double-charging customers when they click "pay" multiple times
- Duplicate PSP processing when downstream services fail
- Financial inconsistencies during network failures

### Implementation Example
```
POST /v1/payments
Idempotency-Key: uuid-12345
{
  "amount": "10.00",
  "payment_order_id": "order-67890"
}
```

## Database Implementation

Exactly-once delivery uses database mechanisms:
1. Server attempts to insert new row with unique key
2. Insertion fails due to constraint violation on duplicate
3. Server detects error and returns existing object
4. Client receives same result regardless of retry count

## Integration with Other Systems

Exactly-once delivery works with:
- [[Message Queue]] systems for reliable message processing
- [[Payment System]] components for financial accuracy
- External PSPs using nonce values for deduplication

This pattern is essential for maintaining data consistency and preventing duplicate operations in distributed financial systems.

---
*Related: [[Idempotency]], [[Payment System]], [[Message Queue]], [[Retry Mechanisms]], [[Database Constraints]]*
