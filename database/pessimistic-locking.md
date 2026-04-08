---
title: "Pessimistic Locking"
category: database
summary: "Pessimistic locking prevents simultaneous updates by acquiring locks on records before modification. While it ensures data consistency, it can create scalability bottlenecks and deadlock risks."
sources:
  - raw/articles/hotel-reservation-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:48:31.893Z
---

# Pessimistic Locking

> Pessimistic locking prevents simultaneous updates by acquiring locks on records before modification. While it ensures data consistency, it can create scalability bottlenecks and deadlock risks.

# Pessimistic Locking

Pessimistic locking is a concurrency control mechanism that prevents simultaneous updates by acquiring exclusive locks on database records before modification, assuming conflicts are likely to occur.

## Implementation

In MySQL, pessimistic locking is implemented using `SELECT ... FOR UPDATE`:

```sql
BEGIN TRANSACTION;
SELECT * FROM inventory 
WHERE product_id = 123 FOR UPDATE;
-- Record is now locked until transaction commits
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id = 123;
COMMIT;
```

The locked rows remain inaccessible to other transactions until the current transaction commits or rolls back.

## Advantages

- **Conflict Prevention**: Eliminates race conditions by serializing access
- **Data Consistency**: Guarantees that updates don't interfere with each other
- **Simple Logic**: Applications don't need complex retry mechanisms
- **Predictable Behavior**: Transactions either succeed or wait, no surprises

## Disadvantages

- **Scalability Issues**: Locks create bottlenecks, especially with long transactions
- **Deadlock Risk**: Multiple resources can create circular wait conditions
- **Reduced Throughput**: Concurrent transactions must wait for lock release
- **Lock Duration Impact**: Long-running transactions block other operations

## Deadlock Scenarios

Deadlocks occur when transactions wait for each other's locks:

- Transaction A locks Resource 1, needs Resource 2
- Transaction B locks Resource 2, needs Resource 1
- Both transactions wait indefinitely

## Best Practices

- Keep transactions short to minimize lock duration
- Acquire locks in consistent order across transactions
- Implement timeout mechanisms for lock acquisition
- Consider lock granularity (row-level vs table-level)

## Use Cases

Pessimistic locking is suitable for:

- High contention scenarios where conflicts are frequent
- Critical operations requiring absolute consistency
- Systems where retry logic is complex or undesirable

However, it's generally not recommended for scalable systems due to performance limitations.

---
*Related: [[Optimistic Locking]], [[Database Concurrency]], [[Deadlocks]], [[Transaction Management]]*
