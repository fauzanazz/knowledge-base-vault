---
title: "Idempotency"
category: system-design
summary: "Idempotency ensures that performing the same operation multiple times produces the same result. It's crucial for preventing duplicate transactions and handling network failures in distributed systems."
sources:
  - raw/articles/hotel-reservation-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:48:31.940Z
---

# Idempotency

> Idempotency ensures that performing the same operation multiple times produces the same result. It's crucial for preventing duplicate transactions and handling network failures in distributed systems.

# Idempotency

Idempotency is a property where performing the same operation multiple times produces the same result as performing it once. This concept is essential for building reliable distributed systems that can handle network failures and duplicate requests.

## Implementation Strategies

### Idempotency Keys

The most common approach uses unique identifiers for each operation:

```json
{
  "reservationID": "13422445",
  "hotelID": "245",
  "startDate": "2021-04-28",
  "endDate": "2021-04-30"
}
```

The `reservationID` serves as an idempotency key, preventing duplicate bookings even if the request is sent multiple times.

### Database Constraints

Implement unique constraints on idempotency keys:

```sql
ALTER TABLE reservations 
ADD CONSTRAINT unique_reservation_id 
UNIQUE (reservation_id);
```

## Benefits

- **Duplicate Prevention**: Eliminates double-charging, duplicate orders, or repeated actions
- **Network Resilience**: Handles timeout scenarios where clients retry requests
- **User Experience**: Prevents accidental double-clicks from causing issues
- **System Reliability**: Reduces data inconsistencies in distributed environments

## Common Patterns

### Client-Generated Keys

Clients generate unique identifiers (UUIDs) before making requests, ensuring each operation has a distinct key.

### Server-Generated Tokens

Servers provide idempotency tokens during initial interactions, which clients include in subsequent operations.

### Natural Keys

Use business logic to create natural idempotency keys, such as combining user ID, timestamp, and operation type.

## Implementation Considerations

- **Key Expiration**: Implement TTL for idempotency keys to prevent unbounded storage growth
- **Response Caching**: Store original responses to return identical results for duplicate requests
- **Error Handling**: Distinguish between retryable and non-retryable failures

## Use Cases

Idempotency is critical for:

- Payment processing systems
- [[Hotel Reservation System]]s
- API gateways and microservices
- Message processing systems
- Financial transactions

Proper idempotency implementation is fundamental for building robust, user-friendly distributed systems.

---
*Related: [[Hotel Reservation System]], [[Distributed Systems]], [[API Design]], [[Database Constraints]]*
