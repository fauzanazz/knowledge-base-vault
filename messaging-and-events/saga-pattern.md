---
title: "Saga Pattern"
category: system-design
summary: "Saga Pattern is a distributed transaction approach that executes a sequence of independent operations with compensating actions for rollback. It's commonly used in microservice architectures to maintain data consistency across services."
sources:
  - raw/articles/digital-wallet-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:24:51.677Z
---

# Saga Pattern

> Saga Pattern is a distributed transaction approach that executes a sequence of independent operations with compensating actions for rollback. It's commonly used in microservice architectures to maintain data consistency across services.

# Saga Pattern

Saga Pattern is a distributed transaction mechanism designed for microservice architectures. It manages long-running transactions by breaking them into a sequence of independent operations, each with a corresponding compensating action for rollback scenarios.

## How Sagas Work

1. **Sequential Execution**: Operations are ordered and executed from first to last
2. **Independent Operations**: Each operation runs in its own database/service
3. **Rollback on Failure**: When an operation fails, compensating operations execute in reverse order
4. **No Locking**: Unlike [[Two-Phase Commit]], Sagas don't hold locks during execution

## Coordination Approaches

### Choreography
- Services subscribe to events and execute their part of the saga
- Decentralized coordination through event-driven communication
- Business logic distributed across multiple services

### Orchestration
- Central coordinator instructs services in correct order
- Centralized business logic and error handling
- Preferred approach for complex workflows like [[Digital Wallet System]]

## Example: Payment Processing

1. **Reserve Inventory** → Compensating: Release Inventory
2. **Process Payment** → Compensating: Refund Payment
3. **Ship Order** → Compensating: Cancel Shipment

If payment processing fails, the saga executes "Release Inventory" to compensate.

## Key Characteristics

- **Linear Execution**: Operations must execute sequentially
- **Eventual Consistency**: System may be temporarily inconsistent during execution
- **Compensating Actions**: Each operation has a corresponding rollback action
- **No Isolation**: Intermediate states are visible to other transactions

## Comparison with TC/C

| Aspect | Saga | [[Try-Confirm/Cancel Protocol]] |
|--------|------|----------------------------------|
| Execution Order | Linear | Parallel possible |
| Compensating Actions | Rollback phase | Cancel phase |
| Coordination | Orchestration/Choreography | Central coordinator |
| Performance | Higher latency | Lower latency |

## Use Cases

Sagas are ideal for business processes spanning multiple microservices, such as order processing, [[Payment System]] workflows, and complex multi-step operations requiring eventual consistency.

---
*Related: [[Distributed Transactions]], [[Try-Confirm/Cancel Protocol]], [[Digital Wallet System]], [[Microservices]], [[Event-Driven Architecture]]*
