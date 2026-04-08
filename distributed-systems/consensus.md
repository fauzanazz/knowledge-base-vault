---
title: "Consensus"
category: distributed-systems
summary: "Consensus is a fundamental distributed systems problem where all non-faulty processes must agree on a single value, typically implemented using algorithms like Raft or Paxos."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:32:14.866Z
---

# Consensus

> Consensus is a fundamental distributed systems problem where all non-faulty processes must agree on a single value, typically implemented using algorithms like Raft or Paxos.

# Consensus

**Consensus** is a fundamental problem in distributed systems where a group of processes must agree on a single value despite failures.

## Definition

Consensus requires three properties:
1. **Agreement**: Every non-faulty process eventually agrees on a value
2. **Validity**: The agreed value was proposed by some process
3. **Termination**: All non-faulty processes eventually decide

## Relationship to State Machine Replication

Solving state machine replication through [[Raft Algorithm]] naturally leads to consensus solutions. The replication process requires nodes to agree on the sequence and content of log entries.

## Practical Implementation

Rather than implementing consensus from scratch, use established solutions:

**etcd**: Distributed key-value store using Raft consensus
**ZooKeeper**: Coordination service with consensus-based replication

### Lease Acquisition Example

Consensus enables [[Leader Election]] through lease acquisition:
1. Clients attempt to create a key with specific TTL
2. Only one client succeeds due to consensus
3. Clients subscribe to state changes for lease reacquisition

## Performance Considerations

Consensus requires coordination, creating scalability bottlenecks. The [[CAP Theorem]] describes trade-offs between consistency, availability, and partition tolerance.

Alternatives like [[Chain Replication]] delegate consensus to a separate control plane, moving coordination away from the critical path for better performance while maintaining strong consistency guarantees.

---
*Related: [[Raft Algorithm]], [[Leader Election]], [[CAP Theorem]]*
