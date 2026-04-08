---
title: "Multi-Version Concurrency Control"
category: database
summary: "MVCC maintains multiple versions of data records to allow read transactions to access consistent snapshots without blocking write transactions."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:09:02.509Z
---

# Multi-Version Concurrency Control

> MVCC maintains multiple versions of data records to allow read transactions to access consistent snapshots without blocking write transactions.

# Multi-Version Concurrency Control

Multi-Version Concurrency Control (MVCC) is a concurrency control method that maintains multiple versions of data to provide consistent reads without blocking writes.

## Core Mechanism

**Version Management**:
- Each record maintains version tags/timestamps
- Write operations create new versions rather than overwriting
- Read operations access the appropriate version based on transaction start time

**Snapshot Isolation**:
- Read transactions see a consistent snapshot of data as of their start time
- Reads never block writes, and writes never block reads
- Only write-write conflicts require coordination

## Implementation Details

**Version Selection**:
- Readers access the newest version committed before their transaction started
- Multiple concurrent readers can access the same version
- Writers create new versions with higher timestamps

**Garbage Collection**:
- Old versions are eventually cleaned up when no active transactions need them
- Requires tracking which transactions are still active
- Balance between storage overhead and read performance

## Advantages

**Performance Benefits**:
- Read-only transactions never block or get aborted
- High concurrency for mixed read/write workloads
- Eliminates many traditional locking conflicts

**Consistency**:
- Provides snapshot isolation by default
- Can be combined with [[Two-Phase Locking]] or [[Optimistic Concurrency Control]] for writes

## Trade-offs

**Storage Overhead**:
- Requires additional space for version metadata
- Multiple versions of the same data consume more storage
- Garbage collection adds complexity

**Implementation Complexity**:
- More complex than single-version schemes
- Requires careful version management and cleanup

MVCC is the most widely used concurrency control scheme in modern databases due to its excellent read performance characteristics.

---
*Related: [[Two-Phase Locking]], [[Optimistic Concurrency Control]], [[ACID Properties]]*
