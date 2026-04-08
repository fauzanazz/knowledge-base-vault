---
title: "Two-Phase Commit"
category: distributed-systems
summary: "Two-phase commit is a distributed consensus protocol that ensures atomic transactions across multiple processes through a coordinator-participant model with prepare and commit phases."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.480Z
---

# Two-Phase Commit

> Two-phase commit is a distributed consensus protocol that ensures atomic transactions across multiple processes through a coordinator-participant model with prepare and commit phases.

# Two-Phase Commit

Two-phase commit (2PC) is a distributed consensus protocol that implements atomic transactions across multiple processes. It ensures that either all participants commit a transaction or all abort it.

## Protocol Structure

**Roles**:
- **Coordinator**: Orchestrates the transaction (often the initiating client)
- **Participants**: Processes that execute parts of the transaction

**Phase 1 - Prepare**:
1. Coordinator sends **prepare** requests to all participants
2. Participants respond with **prepared** (ready to commit) or **abort**
3. Participants that vote "prepared" cannot proceed without coordinator's decision

**Phase 2 - Commit/Abort**:
1. If all participants are prepared: Coordinator sends **commit** requests
2. If any participant aborts: Coordinator sends **abort** requests
3. Participants execute the final decision and acknowledge

## Critical Points

**Participant Blocking**: Once a participant votes "prepared," it cannot make progress without the coordinator's decision, creating a potential blocking scenario.

**Coordinator Persistence**: The coordinator must see the transaction through to completion, retrying failed participants until they respond.

## Limitations

**Performance Issues**:
- Multiple round trips increase latency
- Synchronous and blocking nature reduces throughput

**Availability Problems**:
- Single coordinator failure blocks all participants
- Process crashes can halt entire transaction

**Consensus Type**: 2PC implements **uniform consensus** where all processes (including faulty ones) must agree, making it harder than standard consensus.

## Mitigation Strategies

**Fault Tolerance**:
- Replicate coordinator using [[Raft Algorithm]] or similar
- Use primary-secondary replication for participants
- Implement timeout and retry mechanisms

**Alternative Approaches**: Modern systems often prefer [[Saga Pattern]] or other asynchronous transaction patterns to avoid 2PC's blocking behavior.

Despite its limitations, 2PC remains important for scenarios requiring strong atomicity guarantees across distributed resources.

---
*Related: [[ACID Properties]], [[Consensus Algorithms]], [[Database Transactions]], [[Saga Pattern]]*
