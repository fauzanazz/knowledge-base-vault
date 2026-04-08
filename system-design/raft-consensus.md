---
title: "Raft Consensus"
category: system-design
summary: "Raft is a consensus algorithm that ensures data consistency across distributed systems through leader election and log replication. It provides strong consistency guarantees as long as a majority of nodes remain operational."
sources:
  - raw/articles/digital-wallet-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:24:51.678Z
---

# Raft Consensus

> Raft is a consensus algorithm that ensures data consistency across distributed systems through leader election and log replication. It provides strong consistency guarantees as long as a majority of nodes remain operational.

# Raft Consensus

Raft is a consensus algorithm designed to manage replicated logs in distributed systems. It ensures that multiple nodes agree on a sequence of operations, providing strong consistency guarantees even in the presence of node failures.

## Core Components

### Leader-Follower Architecture
- **Leader**: Active node that handles all client requests
- **Followers**: Passive nodes that replicate leader's log
- **Candidates**: Nodes attempting to become leader during elections

### Log Replication
- All operations stored as entries in replicated log
- Leader appends entries and replicates to followers
- Entries committed once majority of nodes acknowledge
- Followers apply committed entries to their state machines

## Key Guarantees

### Safety Properties
- **Election Safety**: At most one leader per term
- **Leader Append-Only**: Leaders never overwrite log entries
- **Log Matching**: Identical entries at same index across logs
- **Leader Completeness**: Committed entries present in future leaders
- **State Machine Safety**: Same sequence applied to all nodes

### Availability
- System remains operational with majority of nodes (N/2 + 1)
- Automatic leader election when current leader fails
- No data loss as long as majority survives

## Election Process

1. **Timeout**: Follower becomes candidate after election timeout
2. **Vote Request**: Candidate requests votes from other nodes
3. **Majority Vote**: Candidate becomes leader with majority votes
4. **Heartbeats**: Leader sends periodic heartbeats to maintain authority

## Log Replication Flow

1. Client sends command to leader
2. Leader appends entry to local log
3. Leader sends AppendEntries RPC to followers
4. Followers append entry and acknowledge
5. Leader commits entry after majority acknowledgment
6. Leader notifies followers of commitment
7. All nodes apply entry to state machine

## Use Cases

### Distributed Databases
- Ensuring consistency across database replicas
- Managing distributed transactions
- Coordinating schema changes

### Configuration Management
- Distributed configuration stores (etcd, Consul)
- Service discovery systems
- Cluster membership management

### Financial Systems
- [[Digital Wallet System]] event log replication
- [[Event Sourcing]] with high reliability requirements
- Critical data that requires strong consistency

## Performance Characteristics

- **Latency**: Requires majority acknowledgment for commits
- **Throughput**: Limited by leader's capacity and network
- **Scalability**: Read-only queries can be served by followers

## Comparison with Other Algorithms

Raft prioritizes understandability over performance compared to Paxos, making it easier to implement correctly while providing similar consistency guarantees.

---
*Related: [[Distributed Consensus]], [[Event Sourcing]], [[Digital Wallet System]], [[Leader Election]], [[Log Replication]]*
