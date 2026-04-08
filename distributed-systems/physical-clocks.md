---
title: "Physical Clocks"
category: distributed-systems
summary: "Physical clocks provide wall-time timestamps in distributed systems but face synchronization challenges due to clock drift and network latency."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.009Z
---

# Physical Clocks

> Physical clocks provide wall-time timestamps in distributed systems but face synchronization challenges due to clock drift and network latency.

# Physical Clocks

**Physical clocks** provide wall-time timestamps based on hardware timing mechanisms. Every machine has access to a physical clock, typically based on vibrating quartz crystals.

## Clock Types

**Quartz clocks** are cheap but not highly accurate, requiring periodic synchronization with more precise time sources.

**Atomic clocks** use quantum mechanics principles and are extremely accurate (within 1 second over 3 million years). These serve as reference clocks for synchronization protocols.

**Monotonic clocks** are forward-only clocks measuring elapsed time since a specific event (like system boot). They're useful for measuring durations on the same machine but can't compare timestamps across different machines.

## Network Time Protocol (NTP)

NTP is the most common protocol for clock synchronization. It accounts for network latency when synchronizing with high-accuracy time servers.

## Challenges

The main problem with physical clocks is that synchronization can cause clocks to "go backward" on the origin machine. This creates situations where operation A, which happens after operation B, receives a smaller timestamp.

Clock drift and network delays make perfect synchronization impossible, leading to potential ordering inconsistencies in distributed systems.

## Use Cases

Physical clocks work well for logging and approximate timestamps but are insufficient when precise event ordering is required across processes. For ordering guarantees, [[Logical Clocks]] provide better solutions.

---
*Related: [[Logical Clocks]], [[Network Time Protocol]], [[Event Ordering]]*
