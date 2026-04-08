---
title: "Saga Pattern"
category: distributed-systems
summary: "The Saga pattern implements distributed transactions as a sequence of local transactions with compensating actions, providing atomicity without the blocking behavior of two-phase commit."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:44:17.481Z
---

# Saga Pattern

> The Saga pattern implements distributed transactions as a sequence of local transactions with compensating actions, providing atomicity without the blocking behavior of two-phase commit.

# Saga Pattern

The Saga pattern provides a solution for implementing distributed transactions without the blocking behavior of [[Two-Phase Commit]]. It breaks long-running transactions into a sequence of local transactions, each with a corresponding compensating transaction.

## Core Concept

**Transaction Decomposition**: A distributed transaction is composed of several local transactions that can be executed independently across different services or databases.

**Compensating Transactions**: Each local transaction has a corresponding compensating transaction that can reverse its effects if the overall saga fails.

**Atomicity Guarantee**: Either all local transactions succeed, or compensating transactions reverse any partial results.

## Implementation Example

Travel booking saga:

**Forward Transactions**:
- T1: Book flight via third-party service
- T2: Book hotel via third-party service

**Compensating Transactions**:
- C1: Cancel flight booking
- C2: Cancel hotel booking

**Execution Flow**:
1. Execute T1 (book flight)
2. Execute T2 (book hotel)
3. If any step fails, execute compensating transactions in reverse order (C2, then C1)

## Orchestration Approach

**Orchestrator Responsibilities**:
- Manages execution of local transactions
- Tracks transaction state and dependencies
- Executes compensating transactions on failure
- Checkpoints intermediate state for crash recovery

**Communication**: Uses [[Message Queue|message channels]] like Kafka for reliable, idempotent communication with participants.

## Practical Considerations

**Workflow Engines**: Modern implementations leverage existing workflow engines like Temporal rather than building custom orchestrators.

**Idempotency**: All operations must be idempotent to handle message retries and temporary failures.

**State Management**: Orchestrator must persist transaction state to handle crashes and restarts.

## Trade-offs

**Advantages**:
- Non-blocking, asynchronous execution
- Better availability than 2PC
- Suitable for long-running transactions
- Works across organizational boundaries

**Limitations**:
- Lacks isolation between concurrent sagas
- Compensating actions may not perfectly reverse effects
- More complex error handling and monitoring

Sagas represent a pragmatic approach to distributed transactions that prioritizes availability and performance over strict consistency guarantees.

---
*Related: [[Two-Phase Commit]], [[Message Queue]], [[Database Transactions]], [[Distributed Systems]]*
