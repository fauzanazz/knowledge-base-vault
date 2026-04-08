---
title: "Optimistic Concurrency Control"
category: database
summary: "Optimistic concurrency control allows transactions to execute without blocking, checking for conflicts only at commit time and restarting conflicted transactions."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:09:02.508Z
---

# Optimistic Concurrency Control

> Optimistic concurrency control allows transactions to execute without blocking, checking for conflicts only at commit time and restarting conflicted transactions.

# Optimistic Concurrency Control

Optimistic Concurrency Control (OCC) is a non-blocking approach to transaction isolation that assumes conflicts are rare and checks for them only at commit time.

## Core Principle

OCC operates on the optimistic assumption that:
- Transactions are typically short-lived
- Conflicts between concurrent transactions are uncommon
- It's better to do work and potentially restart than to block

## Three-Phase Protocol

**1. Read Phase**: 
- Transactions read data and make changes to a private workspace
- No locks are acquired
- Changes are not visible to other transactions

**2. Validation Phase**:
- Check if the transaction's read set conflicts with other committed transactions
- Verify that read data hasn't been modified since the transaction started

**3. Write Phase**:
- If validation succeeds, apply changes to the database
- If validation fails, abort and restart the transaction

## Conflict Detection

Common validation techniques:
- **Timestamp-based**: Compare transaction timestamps with data modification times
- **Version-based**: Check if data versions have changed since read
- **Workspace comparison**: Analyze overlapping read/write sets

## Performance Characteristics

**Advantages**:
- High concurrency for read-heavy workloads
- No deadlocks (transactions don't wait for locks)
- Better performance when conflicts are rare

**Disadvantages**:
- Wasted work when transactions abort
- Potential starvation under high contention
- Complex validation logic

## Use Cases

OCC is ideal for:
- Read-heavy applications
- Short transactions
- Low-conflict environments
- Web applications with optimistic updates

> ⚠️ *This article may be incomplete. Run `kb compile --full` to regenerate.*

---
*Related: [[Two-Phase Locking]], [[Multi-Version Concurrency Control]], [[ACID Properties]]*
