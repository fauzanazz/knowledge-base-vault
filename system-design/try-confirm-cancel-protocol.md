---
title: "Try-Confirm/Cancel Protocol"
category: system-design
summary: "Try-Confirm/Cancel (TC/C) is a distributed transaction protocol that uses compensating transactions to achieve atomicity across multiple databases. It operates in two phases with independent transactions rather than a single distributed transaction."
sources:
  - raw/articles/digital-wallet-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:24:51.677Z
---

# Try-Confirm/Cancel Protocol

> Try-Confirm/Cancel (TC/C) is a distributed transaction protocol that uses compensating transactions to achieve atomicity across multiple databases. It operates in two phases with independent transactions rather than a single distributed transaction.

# Try-Confirm/Cancel Protocol

Try-Confirm/Cancel (TC/C) is a variation of the [[Two-Phase Commit]] protocol that implements distributed transactions using compensating transactions. Unlike 2PC, TC/C executes two independent transactions rather than a single distributed transaction.

## Protocol Phases

### Phase 1: Try
- Coordinator executes the first part of the transaction
- Resources are reserved or operations are performed
- Each database responds with success or failure

### Phase 2a: Confirm
- If all databases responded "yes" in Phase 1
- Coordinator instructs databases to execute confirming operations
- Completes the intended transaction

### Phase 2b: Cancel
- If any database responded "no" in Phase 1
- Coordinator instructs databases to execute compensating operations
- Reverses any changes made during the Try phase

## Example: Balance Transfer

For transferring $1 from Account A to Account C:

| Phase | Account A | Account C |
|-------|-----------|----------|
| Try | Balance -$1 | No operation |
| Confirm | No operation | Balance +$1 |
| Cancel | Balance +$1 | No operation |

## Key Properties

- **Database Agnostic**: Works with any database supporting transactions
- **Business Logic Complexity**: Distributed transaction logic handled in application layer
- **Parallelizable**: Operations can be executed in parallel for better performance
- **Compensation-Based**: Uses compensating transactions to handle failures

## Failure Recovery

TC/C maintains phase status tables to handle coordinator failures:
- Transaction ID and content
- Try phase status (not sent, sent, response received)
- Second phase type (confirm or cancel)
- Out-of-order execution flags

## Comparison with Other Protocols

Unlike [[Two-Phase Commit]], TC/C allows parallel execution and uses compensating actions. Compared to [[Saga Pattern]], TC/C supports parallel operations while Saga requires linear execution.

---
*Related: [[Two-Phase Commit]], [[Distributed Transactions]], [[Saga Pattern]], [[Digital Wallet System]], [[Compensating Transactions]]*
