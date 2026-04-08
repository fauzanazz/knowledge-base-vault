---
title: "Optimistic Locking"
category: database
summary: "Optimistic locking allows multiple users to attempt updates simultaneously by using version numbers or timestamps to detect conflicts. It performs well under low contention but degrades with high concurrency."
sources:
  - raw/articles/hotel-reservation-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:48:31.889Z
---

# Optimistic Locking

> Optimistic locking allows multiple users to attempt updates simultaneously by using version numbers or timestamps to detect conflicts. It performs well under low contention but degrades with high concurrency.

# Optimistic Locking

Optimistic locking is a concurrency control mechanism that allows multiple users to attempt updating the same data simultaneously, detecting conflicts only at commit time rather than preventing them upfront.

## How It Works

The most common implementation uses version numbers:

1. Add a `version` column to the database table
2. Read the current version number before modification
3. Increment version by 1 during update
4. Database validates that the new version exceeds the previous one
5. If validation fails, the transaction is rolled back

```sql
UPDATE inventory 
SET quantity = quantity - 1, version = version + 1
WHERE product_id = 123 AND version = 5
```

## Advantages

- **No Lock Overhead**: Doesn't require acquiring database locks
- **Better Performance**: Faster than pessimistic locking under low contention
- **Deadlock Prevention**: Eliminates deadlock scenarios
- **Scalability**: Multiple transactions can proceed concurrently

## Disadvantages

- **High Contention Issues**: Performance degrades significantly with many concurrent updates
- **Retry Logic Required**: Applications must handle rollbacks and implement retry mechanisms
- **Wasted Work**: Failed transactions represent wasted computational effort

## Use Cases

Optimistic locking works best for:

- Systems with low data contention
- Read-heavy workloads with occasional updates
- Applications that can tolerate retry logic
- Scenarios where lock duration would be problematic

## Alternative: Timestamps

Instead of version numbers, some systems use timestamps, though this approach is less reliable due to potential clock synchronization issues across distributed systems.

Optimistic locking is particularly effective in reservation systems, inventory management, and collaborative editing applications where conflicts are relatively rare.

---
*Related: [[Pessimistic Locking]], [[Database Concurrency]], [[Transaction Isolation]], [[Hotel Reservation System]]*
