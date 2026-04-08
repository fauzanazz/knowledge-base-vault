---
title: "System Models"
category: distributed-systems
summary: "System models define assumptions about process behavior, communication links, and timing in distributed systems to enable reasoning about correctness and failure scenarios."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.006Z
---

# System Models

> System models define assumptions about process behavior, communication links, and timing in distributed systems to enable reasoning about correctness and failure scenarios.

# System Models

A **system model** is a set of assumptions about the behavior of processes, communication links, and timing that abstracts away technological complexities to enable reasoning about distributed systems.

## Communication Link Models

- **Fair-loss link**: Messages may be lost and duplicated, but if the sender keeps retransmitting, they will eventually be delivered
- **Reliable link**: Messages are delivered exactly once without loss or duplication, implemented on top of fair-loss links through deduplication
- **Authenticated reliable link**: Same as reliable link but includes sender authentication

## Process Failure Models

- **Arbitrary-fault model (Byzantine)**: Processes can deviate from algorithms in unexpected ways due to bugs or malicious activity. Used in safety-critical systems like airplanes and nuclear power plants
- **Crash-recovery model**: Processes don't deviate from algorithms but can crash and restart, losing in-memory state
- **Crash-stop model**: Processes can crash permanently without recovery, modeling unrecoverable hardware faults

## Timing Models

- **Synchronous model**: Operations never exceed a certain time bound (unrealistic for most systems)
- **Asynchronous model**: Operations can take indefinite time
- **Partially synchronous model**: System behaves synchronously most of the time, representative of real-world systems

Most distributed systems assume fair-loss links, crash-recovery processes, and partial synchrony for practical algorithm design.

---
*Related: [[Failure Detection]], [[Consensus]], [[Leader Election]]*
