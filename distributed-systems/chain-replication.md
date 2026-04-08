---
title: "Chain Replication"
category: distributed-systems
summary: "Chain replication arranges processes in a linear chain where writes flow from head to tail, providing strong consistency with different trade-offs than leader-based replication."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.013Z
---

# Chain Replication

> Chain replication arranges processes in a linear chain where writes flow from head to tail, providing strong consistency with different trade-offs than leader-based replication.

# Chain Replication

**Chain replication** arranges processes in a linear chain with the leftmost process as the head and rightmost as the tail, offering an alternative to leader-based replication protocols.

## Operation Flow

**Writes**: Clients send writes to the head, which persists the change and forwards it down the chain. Each node persists and forwards until reaching the tail. Acknowledgments flow back from tail to head, then to the client.

**Reads**: Served exclusively by the tail, ensuring strong consistency since the tail has all committed writes.

## Fault Tolerance

A separate **control plane** (replicated using [[Raft Algorithm]] or similar) monitors system health and handles failures:

**Head failure**: New head appointed, clients notified. Uncommitted writes are safely lost since clients never received acknowledgment.

**Tail failure**: Predecessor becomes new tail. Consistency maintained since all tail updates were acknowledged by predecessors.

**Intermediate failure**: Predecessor links to successor. The successor communicates the last received change to the control plane for forwarding.

## Trade-offs

**Advantages**:
- Strong consistency without leader coordination for reads
- Clear separation of read/write responsibilities
- Can tolerate up to N-1 failures

**Disadvantages**:
- All nodes must process writes (single slow node affects all writes)
- Write delays during failure recovery
- Control plane limited to C/2 failures

## Optimizations

- **Write pipelining**: Multiple dependent writes can be in-flight simultaneously
- **Distributed reads**: Other replicas can serve reads by marking objects as "dirty" and consulting the tail for uncommitted writes

Chain replication delegates coordination to the control plane, achieving higher throughput than consensus-based approaches.

---
*Related: [[Database Replication]], [[Raft Algorithm]], [[Consistency Models]]*
