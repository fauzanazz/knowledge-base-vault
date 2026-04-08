---
title: "Raft Algorithm"
category: distributed-systems
summary: "Raft is a consensus algorithm that provides strong consistency through leader-based replication, designed to be simpler to understand than Paxos."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.011Z
---

# Raft Algorithm

> Raft is a consensus algorithm that provides strong consistency through leader-based replication, designed to be simpler to understand than Paxos.

# Raft Algorithm

**Raft** is a consensus algorithm that provides strong consistency guarantees through leader-based replication. It's designed to be more understandable than Paxos while providing equivalent functionality.

## State Machine Replication

Raft uses state machine replication where the leader emits events for state changes, and followers update their state when receiving events. As long as all followers execute the same sequence of operations, they maintain identical state.

## Replication Process

1. **Leader Election**: A leader is elected using Raft's [[Leader Election]] algorithm
2. **Log Replication**: State changes are persisted in an event log replicated to followers
3. **Append Entries**: Leaders send `AppendEntries` requests containing new entries to all followers
4. **Acknowledgment**: Followers append entries to their logs and acknowledge receipt
5. **Commit**: Once a majority acknowledges (quorum), the leader commits the state change
6. **Propagation**: Subsequent `AppendEntries` requests inform followers to commit

## Fault Tolerance

Raft tolerates F failures with 2F+1 total nodes. Key failure scenarios:

**Leader Failure**: New elections occur, but nodes can't vote for candidates with less up-to-date logs

**Follower Failure**: Returning followers catch up by receiving missed log entries. If entries are missing, the leader sends progressively more historical entries until synchronization succeeds

## Consensus Properties

Raft solves the distributed consensus problem by ensuring:
- Every non-faulty process agrees on values
- Final decisions are identical everywhere  
- Agreed values were proposed by processes

Raft is widely implemented in production systems like etcd and provides the foundation for many distributed databases and coordination services.

---
*Related: [[Leader Election]], [[Consensus]], [[State Machine Replication]]*
