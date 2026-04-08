---
title: "Software Testing"
category: software-engineering
summary: "Software testing verifies application behavior and catches bugs early in development. Tests enable confident system modifications and serve as living documentation, though they cannot predict all possible system states."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.396Z
---

# Software Testing

> Software testing verifies application behavior and catches bugs early in development. Tests enable confident system modifications and serve as living documentation, though they cannot predict all possible system states.

# Software Testing

Software testing verifies that parts of an application work correctly, catching bugs early in the development process. The longer it takes to detect a bug, the more expensive it becomes to fix.

## Benefits of Testing

- **Early Bug Detection** - Catches issues before they reach production
- **Confident Modifications** - Enables altering system behavior with confidence that expected functionality won't break
- **Living Documentation** - Tests serve as always up-to-date documentation of the codebase
- **Interface Improvement** - Forces developers to consider the system from a client's perspective

## Test Scope

Scope defines the code path under test (system under test - SUT):

**Unit Tests** - Validate behavior of single components like classes. Should work with public interfaces only, test state changes rather than action sequences, and focus on behaviors.

**Integration Tests** - Verify correct integration with external components. Narrow integration tests exercise code paths communicating with external dependencies. Broad integration tests span multiple services.

**End-to-End Tests** - Validate user-facing scenarios across multiple services. Run in shared environments and are slower, more brittle, but necessary for validating complete user journeys.

## Test Size

Test size determines computing resources needed:

- **Small** - Single process, no I/O, fast and deterministic
- **Medium** - Single node with local I/O, some delays possible  
- **Large** - Multiple nodes, more non-determinism and delays

The pyramid principle suggests many unit tests, fewer integration tests, and minimal end-to-end tests to balance coverage with maintainability.

---
*Related: [[Software Maintainability]], [[Test Doubles]], [[Formal Verification]], [[Contract Testing]]*
