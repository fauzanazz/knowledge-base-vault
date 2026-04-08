---
title: "Logical Clocks"
category: distributed-systems
summary: "Logical clocks measure the passing of time in terms of logical operations rather than wall-clock time, enabling ordering of events in distributed systems."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.872Z
---

# Logical Clocks

> Logical clocks measure the passing of time in terms of logical operations rather than wall-clock time, enabling ordering of events in distributed systems.

# Logical Clocks

Logical clocks measure passing of time in terms of logical operations, not wall-clock time. They solve the fundamental problem of ordering events across multiple machines in [[Distributed Systems]] where there is no single global clock.

## Lamport Clocks

A Lamport clock is a simple counter that gets incremented on each operation. For distributed systems:

- Each message sent includes the logical clock's counter
- Receiving machines consider the counter of received messages
- Subsequent operations use a counter bigger than any received

This ensures dependent operations have different timestamps, but unrelated operations can have identical ones. Process IDs can break ties for total ordering.

**Limitation**: Lamport clocks don't imply causal relationship - event A can happen before B even if B's timestamp is greater.

## Vector Clocks

Vector clocks provide stronger guarantees by maintaining an array of counters, one for each process:

- Counters start from 0
- Operations increment the process's own counter
- Messages include the entire vector
- Receivers merge vectors by taking max of each element, then increment own counter

Vector clocks guarantee that event A happened before B if:
- Every counter in A ≤ every counter in B
- At least one counter in A < corresponding counter in B

If neither condition holds, events are concurrent.

**Trade-off**: Storage requirements grow with every new process. Alternatives like dotted version clocks address this limitation.

Vector clocks are essential when you need to derive the order of events across different processes in distributed systems.

---
*Related: [[Distributed Systems]], [[Time]], [[Vector Clocks]]*
