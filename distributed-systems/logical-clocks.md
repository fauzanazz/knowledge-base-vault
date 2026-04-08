---
title: "Logical Clocks"
category: distributed-systems
summary: "Logical clocks measure the passing of time in terms of logical operations rather than wall-clock time, enabling event ordering in distributed systems."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.008Z
---

# Logical Clocks

> Logical clocks measure the passing of time in terms of logical operations rather than wall-clock time, enabling event ordering in distributed systems.

# Logical Clocks

**Logical clocks** measure time progression through logical operations rather than physical time, solving the challenge of ordering events across distributed systems without synchronized physical clocks.

## Lamport Clocks

**Lamport clocks** use a simple counter that increments with each operation. When sending messages between machines:

1. Each message includes the sender's logical clock counter
2. Receiving machines compare their counter with the received counter
3. Subsequent operations use a counter greater than both values
4. Process IDs break ties when counters are identical

Lamport clocks ensure dependent operations have different timestamps, but unrelated operations may have identical ones. However, they don't guarantee causal relationships - event A can happen before B even if B has a larger timestamp.

## Vector Clocks

**Vector clocks** provide stronger guarantees by maintaining an array of counters, one per process:

1. All counters start at 0
2. Operations increment the local process counter
3. Messages include the entire counter array
4. Receivers merge arrays by taking the maximum of each element
5. The receiving process then increments its own counter

Vector clocks guarantee that event A happened before event B if every counter in A is less than or equal to the corresponding counter in B, and at least one counter in A is strictly less than B's counter.

## Trade-offs

Vector clocks require storage proportional to the number of processes, which can become problematic in large systems. Alternative approaches like dotted version clocks address this scalability concern.

---
*Related: [[Physical Clocks]], [[Event Ordering]], [[Distributed Systems]]*
