---
title: "Consensus"
category: distributed-systems
summary: "Consensus is a fundamental distributed systems problem where multiple nodes must agree on a single value despite failures, solved by algorithms like Raft and Paxos."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.873Z
---

# Consensus

> Consensus is a fundamental distributed systems problem where multiple nodes must agree on a single value despite failures, solved by algorithms like Raft and Paxos.

# Consensus

Consensus is a fundamental problem in [[Distributed Systems]] where a group of nodes must agree on a single value despite the possibility of failures. It emerges naturally from solving state machine replication.

## Consensus Definition

A consensus algorithm must guarantee:
- **Agreement** - Every non-faulty process eventually agrees on a value
- **Validity** - The final decision of non-faulty processes is the same everywhere
- **Integrity** - The value that has been agreed on has been proposed by a process

## Use Cases

**[[Leader Election]]** - Agreeing on which process in a group gets to acquire a lease

**Configuration Management** - Agreeing on system configuration changes across nodes

**Atomic Commitment** - Ensuring all nodes commit or abort a distributed transaction

## Implementation Approaches

Rather than implementing consensus from scratch, use proven solutions:

**etcd** - Distributed key-value store using [[Raft Algorithm]] for consensus

**ZooKeeper** - Coordination service using ZAB (ZooKeeper Atomic Broadcast) protocol

Both replicate their state for fault-tolerance and provide linearizable operations.

## Practical Implementation

To implement lease acquisition:
1. Clients attempt to create a key with specific TTL in the key-value store
2. Only one client succeeds due to linearizable compare-and-swap operations
3. Clients subscribe to state changes to reacquire lease if leader dies

## Relationship to CAP Theorem

Consensus algorithms must choose between availability and consistency during network partitions. Most consensus systems prioritize consistency, becoming unavailable during partitions rather than risking split-brain scenarios.

Consensus is essential for building reliable distributed systems but comes with performance costs due to required coordination.

---
*Related: [[Raft Algorithm]], [[Leader Election]], [[Distributed Systems]], [[CAP Theorem]]*
