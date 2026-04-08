---
title: "Quorum Consensus"
category: system-design
summary: "Quorum consensus is a distributed systems technique that ensures data consistency by requiring a minimum number of nodes to agree on read and write operations. It balances consistency, availability, and performance through configurable parameters."
sources:
  - raw/articles/key-value-store-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:40:37.042Z
---

# Quorum Consensus

> Quorum consensus is a distributed systems technique that ensures data consistency by requiring a minimum number of nodes to agree on read and write operations. It balances consistency, availability, and performance through configurable parameters.

# Quorum Consensus

**Quorum consensus** is a distributed systems technique used to ensure data consistency across replicated nodes by requiring agreement from a minimum number of replicas for read and write operations.

## Parameters

### Core Variables
- **N**: Total number of replicas
- **W**: Write quorum size (minimum replicas that must acknowledge a write)
- **R**: Read quorum size (minimum replicas that must respond to a read)

### Consistency Rule
**Strong Consistency**: `W + R > N`

This ensures that read and write quorums overlap, guaranteeing that reads will see the most recent writes.

## Configuration Trade-offs

### Fast Read Optimization
**R = 1, W = N**
- Reads require only one replica response
- Writes must be acknowledged by all replicas
- Use case: Read-heavy workloads

### Fast Write Optimization
**W = 1, R = N**
- Writes require only one replica acknowledgment
- Reads must query all replicas
- Use case: Write-heavy workloads

### Balanced Approach
**N = 3, W = 2, R = 2**
- Common configuration providing strong consistency
- Balances performance and fault tolerance
- Can tolerate one replica failure

### Eventual Consistency
**W + R ≤ N**
- Prioritizes availability over strong consistency
- Accepts potential stale reads
- Suitable for [[Key-Value Store]] systems choosing AP in [[CAP Theorem]]

## Operational Behavior

### Write Process
1. Client sends write request to coordinator
2. Coordinator forwards request to N replicas
3. Wait for W replicas to acknowledge
4. Return success to client once W threshold met
5. Continue propagating to remaining replicas asynchronously

### Read Process
1. Client sends read request to coordinator
2. Coordinator queries R replicas
3. Return data once R responses received
4. Resolve conflicts if multiple versions exist (using [[Vector Clock]])

## Failure Handling

### Sloppy Quorum
When some replicas are unavailable:
- Use first W healthy servers for writes
- Use first R healthy servers for reads
- Temporarily ignore failed replicas
- Implement **hinted handoff** for recovery

### Hinted Handoff
- Healthy nodes temporarily store data for failed nodes
- When failed nodes recover, stored data is transferred
- Ensures eventual consistency after failures

## Benefits

- **Tunable Consistency**: Adjust W and R based on requirements
- **Fault Tolerance**: Continue operating with some replica failures
- **Performance Control**: Optimize for read or write latency
- **Availability**: Maintain service during partial failures

## Applications

Quorum consensus is fundamental to:
- [[Key-Value Store]] implementations
- [[Database Replication]] systems
- Distributed storage systems (Cassandra, DynamoDB)
- Blockchain consensus mechanisms

It enables systems to balance the trade-offs inherent in the [[CAP Theorem]] by providing configurable consistency guarantees.

---
*Related: [[Key-Value Store]], [[Database Replication]], [[CAP Theorem]], [[Vector Clock]], [[Distributed Systems]]*
