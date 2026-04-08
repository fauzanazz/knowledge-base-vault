---
title: "System Models"
category: distributed-systems
summary: "System models define assumptions about process behavior, communication links, and timing in distributed systems to enable reasoning about correctness and failure scenarios."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.871Z
---

# System Models

> System models define assumptions about process behavior, communication links, and timing in distributed systems to enable reasoning about correctness and failure scenarios.

# System Models

A system model is a set of assumptions about the behavior of processes, communication links and timing while abstracting away the complexities of the technology being used. These models are essential for reasoning about [[Distributed Systems]] and verifying algorithm correctness.

## Communication Link Models

**Fair-loss link** - Messages may be lost and duplicated, but if sender keeps retransmitting messages, they will eventually be delivered.

**Reliable link** - Messages are delivered exactly once without loss or duplication. Can be implemented on top of fair-loss links by deduplicating messages at the receiving end.

**Authenticated reliable link** - Same as reliable link but also assumes the receiver can authenticate the sender.

## Process Failure Models

**Arbitrary-fault model** (Byzantine) - Process can deviate from algorithm in unexpected ways, leading to crashes due to bugs or malicious activity. Used for safety-critical systems like airplanes, nuclear power plants, and Bitcoin.

**Crash-recovery model** - Process doesn't deviate from algorithm but can crash and restart at any time, losing its in-memory state. Most common model for typical distributed systems.

**Crash-stop model** - Process doesn't deviate from algorithm but when it crashes, it doesn't come back online. Models unrecoverable hardware faults.

## Timing Models

**Synchronous model** - Operations never take more than a certain amount of time. Not realistic for most systems.

**Asynchronous model** - Operations can take indefinite time. More realistic but harder to work with.

**Partially synchronous model** - System behaves synchronously most of the time. Representative of real-world systems and commonly assumed in practice.

---
*Related: [[Distributed Systems]], [[Failure Detection]], [[System Resiliency]]*
