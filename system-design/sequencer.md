---
title: "Sequencer"
category: system-design
summary: "A sequencer is a critical component in distributed systems that assigns monotonically increasing sequence numbers to events or messages. It ensures deterministic processing order and enables exactly-once delivery guarantees."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:37:24.875Z
---

# Sequencer

> A sequencer is a critical component in distributed systems that assigns monotonically increasing sequence numbers to events or messages. It ensures deterministic processing order and enables exactly-once delivery guarantees.

# Sequencer

A sequencer is a system component that assigns monotonically increasing sequence numbers to events or messages, ensuring deterministic processing order in distributed systems. It's particularly critical in financial systems like [[Stock Exchange System Design]] where order of operations directly impacts fairness and correctness.

## Core Functions

### Sequence Number Assignment
Assigns unique, ordered identifiers to incoming events:
```
Incoming Order → Sequencer → Order with Sequence ID
{ symbol: 'AAPL', price: 150 } → { id: 12345, symbol: 'AAPL', price: 150 }
```

### Deterministic Ordering
Ensures all downstream components process events in identical order:
- Critical for [[Matching Engine]] fairness
- Enables consistent state across replicas
- Supports audit and compliance requirements

### Event Validation
Verifies sequence continuity:
```
if (event.sequenceId != expectedSequence) {
  return ERROR(OUT_OF_ORDER, expectedSequence);
}
```

## Implementation Approaches

### Single Writer Pattern
One sequencer instance handles all ordering:
- **Pros**: Simple, guaranteed ordering
- **Cons**: Single point of failure, throughput bottleneck
- **Use case**: High-consistency requirements

### Partitioned Sequencing
Separate sequencers for different partitions:
```
// Route by symbol hash
partition = hash(symbol) % numPartitions
sequencer = sequencers[partition]
```

### Hybrid Approach
Combine global and local sequencing:
- Global sequence for cross-partition ordering
- Local sequences within partitions
- Used in distributed [[Message Queue]] systems

## Performance Optimization

### Memory-Mapped Storage
In ultra-low latency systems:
```
// Store sequence in shared memory
mmap_file = mmap("/dev/shm/sequence", size)
atomic_increment(mmap_file->sequence)
```

### Batch Processing
Process multiple events per sequence operation:
- Reduce per-event overhead
- Improve throughput
- Maintain ordering within batches

### Lock-Free Implementation
Use atomic operations for high concurrency:
```cpp
std::atomic<uint64_t> sequence_counter{0};
uint64_t next_sequence = sequence_counter.fetch_add(1);
```

## Fault Tolerance

### Sequence Persistence
Store sequence state for recovery:
- Periodic checkpoints to durable storage
- [[Write-Ahead Log]] for sequence operations
- Fast recovery on restart

### Leader Election
For high availability:
- Multiple sequencer instances
- Leader election using Raft or similar
- Automatic failover on leader failure

### Gap Detection
Handle missing sequences:
```
if (receivedSeq > expectedSeq + 1) {
  requestMissingEvents(expectedSeq + 1, receivedSeq - 1);
}
```

## Integration Patterns

### Event Sourcing
Sequencer enables deterministic [[Event Sourcing]]:
- Events processed in sequence order
- Reproducible state reconstruction
- Simplified debugging and testing

### Exactly-Once Processing
Combined with idempotency:
```
if (processedSequences.contains(event.sequenceId)) {
  return ALREADY_PROCESSED;
}
processEvent(event);
processedSequences.add(event.sequenceId);
```

### Replication
Sequence numbers enable consistent replication:
- Replicas process events in same order
- Detect and handle replication lag
- Support read-after-write consistency

## Use Cases

### Financial Trading
- Order processing fairness
- Regulatory compliance
- Market data consistency

### Distributed Databases
- Transaction ordering
- Replication consistency
- Conflict resolution

### Message Systems
- Delivery guarantees
- Consumer coordination
- Partition ordering

## Monitoring

Key metrics to track:
- Sequence generation rate
- Gap detection frequency
- Processing latency
- Failover time

Sequencers are fundamental building blocks for systems requiring strong ordering guarantees and deterministic behavior.

---
*Related: [[Event Sourcing]], [[Stock Exchange System Design]], [[Distributed Systems]], [[Exactly-Once Delivery]], [[Message Queue]]*
