---
title: "Two-Phase Locking"
category: database
summary: "Two-phase locking is a pessimistic concurrency control protocol that uses read and write locks to ensure transaction isolation and prevent race conditions."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:09:02.507Z
---

# Two-Phase Locking

> Two-phase locking is a pessimistic concurrency control protocol that uses read and write locks to ensure transaction isolation and prevent race conditions.

# Two-Phase Locking

Two-phase locking (2PL) is a pessimistic concurrency control protocol that ensures transaction isolation by using locks to prevent conflicting access to shared data.

## Lock Types

**Read Locks (Shared)**:
- Multiple transactions can hold read locks on the same object
- Prevents write locks from being acquired
- Released when all holding transactions commit

**Write Locks (Exclusive)**:
- Only one transaction can hold a write lock
- Blocks both read and write lock acquisition
- Released when the transaction commits

## Protocol Rules

**Two phases**:
1. **Growing phase**: Acquire locks as needed, cannot release any locks
2. **Shrinking phase**: Release locks, cannot acquire new locks

This ensures **serializability** - the strongest isolation level where concurrent execution appears sequential.

## Deadlock Handling

Deadlocks occur when transactions wait for each other's locks in a cycle:
```
Transaction A: holds lock on X, wants lock on Y
Transaction B: holds lock on Y, wants lock on X
```

**Solutions**:
- **Detection**: Monitor wait-for graphs, abort victim transactions
- **Prevention**: Order lock acquisition, timeout mechanisms
- **Avoidance**: Timestamp-based protocols

## Trade-offs

**Advantages**:
- Guarantees strong consistency
- Prevents all isolation anomalies
- Well-understood and widely implemented

**Disadvantages**:
- Reduced concurrency due to blocking
- Potential for deadlocks
- Performance overhead from lock management

2PL is most suitable for conflict-heavy workloads where consistency is prioritized over maximum concurrency.

---
*Related: [[ACID Properties]], [[Optimistic Concurrency Control]], [[Multi-Version Concurrency Control]]*
