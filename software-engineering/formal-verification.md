---
title: "Formal Verification"
category: software-engineering
summary: "Formal verification uses mathematical specifications to detect bugs and architecture shortcomings before writing code. TLA+ is a well-known formal specification language used by companies like Microsoft and Amazon."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.397Z
---

# Formal Verification

> Formal verification uses mathematical specifications to detect bugs and architecture shortcomings before writing code. TLA+ is a well-known formal specification language used by companies like Microsoft and Amazon.

# Formal Verification

Formal verification involves writing high-level specifications of system behavior to detect subtle bugs and architecture shortcomings before implementing any code.

## Purpose and Benefits

A specification helps reason about system behaviors and serves as documentation and implementation guidance. It doesn't require describing every system detail - focus on paths most likely to contain errors that are hard to detect through testing.

Formal verification is particularly valuable for systems running at huge scale, as they will eventually encounter states that humans cannot imagine or predict.

## TLA+ Language

TLA+ is a well-known formal specification language that describes system behavior through:
- Set of possible states
- State transitions and changes

Major companies use TLA+ for their most complex distributed systems:
- Microsoft uses it for system specifications
- Amazon uses it for services like S3 and CosmosDB

## Safety and Liveness Properties

Formal verification validates two key properties:

**Safety** - Something is true for all behaviors of the system (invariant). Example: data consistency is maintained across all operations.

**Liveness** - Something eventually happens. Example: a request will eventually receive a response.

## Practical Example

Consider migrating from key-value store X to Y:
1. Service writes to both X and Y while reading from X
2. Batch process backfills Y with data from X  
3. Application switches to read/write exclusively from Y

This approach seems reasonable but formal verification reveals issues like write ordering problems and consistency violations that could be caught before implementation.

---
*Related: [[Software Testing]], [[System Verification]], [[Distributed Systems]]*
