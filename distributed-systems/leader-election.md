---
title: "Leader Election"
category: distributed-systems
summary: "Leader election algorithms select a single process from a group to coordinate shared resources or assign work, ensuring safety and liveness properties."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.010Z
---

# Leader Election

> Leader election algorithms select a single process from a group to coordinate shared resources or assign work, ensuring safety and liveness properties.

# Leader Election

**Leader election** selects one process from N equally valid candidates to gain exclusive access to shared resources or coordinate work distribution.

## Required Properties

- **Safety**: Only one leader exists at any time
- **Liveness**: The algorithm works correctly despite failures

## Raft Leader Election

Raft implements leader election through a state machine with three states:

**Follower**: Recognizes another process as leader and expects periodic heartbeats

**Candidate**: Starts new elections, proposing itself as leader

**Leader**: Coordinates the system and sends heartbeats

### Election Process

Time is divided into **election terms** with consecutive integer identifiers. Elections begin each term:

1. Processes start as followers expecting leader heartbeats
2. Missing heartbeats trigger timeout, transitioning to candidate state
3. Candidates request votes from all other processes
4. Majority votes win the election
5. Receiving heartbeats from higher-term leaders causes candidates to step down
6. Split votes trigger new elections after random timeouts

## Practical Implementation

Rather than implementing from scratch, use fault-tolerant key-value stores with TTL and linearizable compare-and-swap operations (like etcd or ZooKeeper).

## Considerations

Leaders should minimize work to reduce impact of occasional multiple leaders due to network partitions. Race conditions can occur after acquiring leadership if network issues cause lease loss while the process still believes it's the leader.

---
*Related: [[Raft Algorithm]], [[Consensus]], [[Failure Detection]]*
