---
title: "Leader Election"
category: distributed-systems
summary: "Leader election algorithms select a single process from a group of candidates to gain exclusive rights to resources or coordinate work, ensuring safety and liveness properties."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.872Z
---

# Leader Election

> Leader election algorithms select a single process from a group of candidates to gain exclusive rights to resources or coordinate work, ensuring safety and liveness properties.

# Leader Election

Leader election is a fundamental problem in [[Distributed Systems]] where one among N processes needs to gain exclusive rights to accessing a shared resource or to assign work to others.

## Required Properties

A leader election algorithm must guarantee:

**Safety** - There will always be at most one leader elected at a time
**Liveness** - The process will work correctly even in the presence of failures

## Raft Leader Election

Raft is a popular leader election algorithm where every process is a state machine with three states:

**Follower state** - Process recognizes another process as leader
**Candidate state** - Process starts a new election, proposing itself as leader  
**Leader state** - Process is the current leader

### Election Process

Time is divided into **election terms** numbered with consecutive integers. Each term begins with a new election.

1. **Startup** - All processes begin as followers
2. **Timeout** - If follower doesn't receive leader heartbeat, it becomes candidate
3. **Campaign** - Candidate requests votes from all other processes
4. **Resolution** - Three outcomes possible:
   - Candidate wins majority → becomes leader
   - Another process wins → reverts to follower
   - Split vote → new election after random timeout

## Practical Implementation

In practice, you rarely implement leader election from scratch. Instead, use fault-tolerant key-value stores with TTL and linearizable compare-and-swap operations like etcd or ZooKeeper.

**Important consideration**: Leaders should do minimal work and systems should handle occasional multiple leaders gracefully, as network partitions can cause lease confusion.

---
*Related: [[Distributed Systems]], [[Raft Algorithm]], [[Consensus]]*
