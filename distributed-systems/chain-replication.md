---
title: "Chain Replication"
category: distributed-systems
summary: "Chain replication arranges processes in a linear chain where writes flow from head to tail and reads are served by the tail, providing strong consistency with different performance characteristics than leader-based replication."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:07:00.685Z
---

# Chain Replication

> Chain replication arranges processes in a linear chain where writes flow from head to tail and reads are served by the tail, providing strong consistency with different performance characteristics than leader-based replication.

# Chain Replication

Chain replication is an alternative to leader-based replication protocols like [[Raft Algorithm]]. It arranges processes in a linear chain with distinct roles for the head (leftmost) and tail (rightmost) processes.

## Architecture

**Write Path**: Clients send writes to the head, which persists the change and forwards it down the chain. Each node persists and forwards until reaching the tail.

**Acknowledgment Path**: The tail acknowledges receipt back up the chain. Only when the head receives confirmation does it inform the client of successful write.

**Read Path**: Reads are served exclusively by the tail, ensuring strong consistency since the tail has all committed writes.

## Fault Tolerance

Chain replication delegates failure handling to a separate **control plane** - a replicated component that monitors system health and reacts to failures:

**Head failure**: New head appointed, clients notified. Uncommitted writes are safely lost since clients never received acknowledgment.

**Tail failure**: Predecessor becomes new tail. All updates at the tail were necessarily acknowledged by predecessors.

**Intermediate failure**: Predecessor links to successor. Failing node's successor communicates last received change to control plane for forwarding.

## Performance Characteristics

**Advantages**:
- Strong consistency for reads without leader coordination
- Clear separation of read/write responsibilities
- Can tolerate up to N-1 failures

**Trade-offs**:
- Writes must traverse all nodes (single slow node affects all writes)
- Write delays during failure recovery until control plane responds
- Control plane itself can only tolerate C/2 failures

## Optimizations

- **Write pipelining**: Multiple dependent writes can be in-flight simultaneously
- **Distributed reads**: Other replicas can serve reads by marking objects as "dirty" and consulting the tail for verification

Chain replication demonstrates how moving coordination off the critical path can achieve both high throughput and strong consistency.

---
*Related: [[Database Replication]], [[Raft Algorithm]], [[Consistency Models]], [[Fault Tolerance]]*
