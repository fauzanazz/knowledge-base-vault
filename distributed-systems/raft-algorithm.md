---
title: "Raft Algorithm"
category: distributed-systems
summary: "Raft is a consensus algorithm that provides strong consistency guarantees through leader election and log replication, designed to be more understandable than Paxos."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.873Z
---

# Raft Algorithm

> Raft is a consensus algorithm that provides strong consistency guarantees through leader election and log replication, designed to be more understandable than Paxos.

# Raft Algorithm

Raft is a consensus algorithm that provides the strongest consistency guarantee in [[Distributed Systems]] - to clients, data appears to be stored on a single process. It's designed to be simpler to understand than Paxos while providing similar guarantees.

## Core Components

Raft consists of two main parts:
1. **[[Leader Election]]** - Selecting a single leader from multiple candidates
2. **Log Replication** - Replicating state changes across all nodes

## State Machine Replication Process

1. **Leader Election** - System starts by electing a leader using Raft's election algorithm
2. **Log Persistence** - Only leader accepts state updates, persisting them in local log without changing state
3. **Replication** - Leader sends `AppendEntries` requests to all followers with new entries
4. **Acknowledgment** - Followers append to local log and send acknowledgment back
5. **Commit** - Once leader receives majority confirmations (quorum), it persists state change
6. **Propagation** - Followers persist state change on subsequent `AppendEntries` indicating commit

## Fault Tolerance

Raft can tolerate F failures where total nodes = 2F + 1.

**Leader Failure** - New election occurs. Candidates with less up-to-date logs cannot win elections.

**Follower Failure** - When follower returns, it receives missed log entries through `AppendEntries` messages. If entries are missing, leader sends progressively more entries until follower is caught up.

## Consensus Properties

Raft solves the distributed consensus problem by ensuring:
- Every non-faulty process eventually agrees on a value
- Final decision is the same everywhere  
- Agreed value was proposed by a process

This makes Raft suitable for implementing fault-tolerant systems like etcd and other distributed key-value stores.

---
*Related: [[Leader Election]], [[Consensus]], [[Database Replication]], [[Distributed Systems]]*
