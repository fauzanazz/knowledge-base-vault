---
title: "Two-Phase Commit"
category: system-design
summary: "Two-Phase Commit (2PC) is a distributed transaction protocol that ensures atomicity across multiple databases through a coordinator-based approach. It operates in prepare and commit phases but suffers from blocking behavior and coordinator failure issues."
sources:
  - raw/articles/digital-wallet-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:24:51.678Z
---

# Two-Phase Commit

> Two-Phase Commit (2PC) is a distributed transaction protocol that ensures atomicity across multiple databases through a coordinator-based approach. It operates in prepare and commit phases but suffers from blocking behavior and coordinator failure issues.

# Two-Phase Commit

Two-Phase Commit (2PC) is a distributed transaction protocol that ensures atomicity across multiple databases or resource managers. It uses a coordinator to manage the transaction lifecycle and guarantee that all participants either commit or abort the transaction together.

## Protocol Phases

### Phase 1: Prepare
1. **Coordinator** sends PREPARE message to all participants
2. **Participants** perform transaction operations but don't commit
3. **Participants** respond with VOTE-COMMIT or VOTE-ABORT
4. **Participants** enter prepared state and hold locks

### Phase 2: Commit/Abort
- **If all voted COMMIT**: Coordinator sends COMMIT to all participants
- **If any voted ABORT**: Coordinator sends ABORT to all participants
- **Participants** execute the decision and release locks
- **Participants** acknowledge completion to coordinator

## Key Properties

### Atomicity Guarantee
- All participants commit or all abort
- No partial transaction states visible
- Maintains ACID properties across distributed resources

### Blocking Protocol
- Participants hold locks during both phases
- System blocks if coordinator fails after prepare phase
- Requires timeout mechanisms for failure detection

## Failure Scenarios

### Participant Failure
- During Phase 1: Coordinator aborts transaction
- During Phase 2: Coordinator retries or uses timeout
- Recovery requires transaction log replay

### Coordinator Failure
- **Before Phase 2**: Participants can safely abort
- **After Phase 2 decision**: Participants must wait for coordinator recovery
- **Blocking problem**: System cannot proceed without coordinator

## Advantages

- **Strong Consistency**: Guarantees ACID properties
- **Simplicity**: Well-understood protocol
- **Atomicity**: All-or-nothing transaction semantics
- **Industry Standard**: Widely implemented in databases

## Disadvantages

### Performance Issues
- **Lock Contention**: Long-held locks reduce concurrency
- **Network Latency**: Multiple round trips required
- **Blocking**: Coordinator failure blocks entire system

### Scalability Limitations
- **Single Point of Failure**: Coordinator bottleneck
- **Resource Holding**: Locks held across network calls
- **Timeout Complexity**: Difficult to tune timeouts correctly

## Alternatives

Modern distributed systems often prefer:
- **[[Try-Confirm/Cancel Protocol]]**: Non-blocking with compensation
- **[[Saga Pattern]]**: Long-running transactions with rollback
- **[[Event Sourcing]]**: Eventual consistency with audit trails

## Use Cases

2PC remains useful for:
- Traditional database systems requiring strong consistency
- Short-lived transactions with low latency requirements
- Systems where blocking behavior is acceptable
- [[Digital Wallet System]] implementations prioritizing correctness over availability

> ⚠️ *This article may be incomplete. Run `kb compile --full` to regenerate.*

---
*Related: [[Distributed Transactions]], [[Try-Confirm/Cancel Protocol]], [[Saga Pattern]], [[ACID Properties]], [[Digital Wallet System]]*
