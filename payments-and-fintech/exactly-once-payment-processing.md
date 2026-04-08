---
title: "Exactly-Once Payment Processing"
category: system-design
summary: "Exactly-once payment processing ensures that payment transactions are executed precisely one time, preventing double-charging customers. It combines at-least-once delivery through retry mechanisms with at-most-once guarantees via idempotency keys and database constraints."
sources:
  - raw/articles/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:16:46.903Z
---

# Exactly-Once Payment Processing

> Exactly-once payment processing ensures that payment transactions are executed precisely one time, preventing double-charging customers. It combines at-least-once delivery through retry mechanisms with at-most-once guarantees via idempotency keys and database constraints.

# Exactly-Once Payment Processing

**Exactly-once payment processing** is a critical guarantee in [[Payment System]]s that ensures each payment transaction is executed precisely one time, preventing both failed payments and double-charging customers.

## Core Concept

Exactly-once = **At-least-once** + **At-most-once**
- **At-least-once**: Retry mechanisms ensure payment eventually succeeds
- **At-most-once**: Idempotency prevents duplicate processing

## At-Least-Once Delivery

### Retry Strategies
- **Immediate Retry**: Instant retry after failure (not recommended)
- **Fixed Intervals**: Wait fixed time between retries
- **Incremental Intervals**: Gradually increase retry intervals
- **Exponential Backoff**: Double interval between retries (recommended)
- **Cancel**: Stop retrying after threshold or terminal errors

### Retry Queue Architecture
- Failed payments published to retry queue
- Background workers process retry attempts
- Dead-letter queue for terminal failures
- Configurable retry limits and timeouts

## At-Most-Once Guarantee

### Idempotency Keys
Unique identifiers (typically UUIDs) that ensure duplicate requests produce identical results:
- Client includes `idempotency-key` header
- Server stores key with transaction result
- Subsequent requests with same key return cached result

### Database Implementation
```sql
CREATE TABLE payments (
  idempotency_key VARCHAR(36) UNIQUE,
  payment_id VARCHAR(36),
  status ENUM('SUCCESS', 'FAILED'),
  amount DECIMAL(10,2)
);
```

### Processing Flow
1. Server attempts to insert new payment record
2. If unique constraint violation occurs:
   - Retrieve existing payment result
   - Return cached response to client
3. If insertion succeeds:
   - Process payment normally
   - Store result for future idempotency checks

## Common Failure Scenarios

### Double-Click Prevention
- User clicks "Pay" button multiple times
- Same idempotency key prevents multiple charges
- UI can disable button after first click

### Network Timeout Handling
- Payment succeeds at PSP but response is lost
- Client retries with same idempotency key
- System returns success without reprocessing

### Partial System Failures
- Payment succeeds at PSP but ledger update fails
- Retry mechanism completes remaining operations
- Idempotency prevents PSP reprocessing

## PSP Integration

Payment Service Providers support idempotency through:
- **Nonce values**: Unique transaction identifiers
- **Duplicate detection**: PSP prevents processing same nonce twice
- **Status APIs**: Allow querying payment status by nonce

Exactly-once processing is fundamental to payment system reliability and customer trust.

---
*Related: *
