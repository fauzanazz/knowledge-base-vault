---
title: "Software Testing"
category: software-engineering
summary: "Software testing verifies system behavior and enables confident modifications by catching bugs early. Tests serve as documentation and improve system interfaces, though they cannot predict all possible application states."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T19:11:26.757Z
---

# Software Testing

> Software testing verifies system behavior and enables confident modifications by catching bugs early. Tests serve as documentation and improve system interfaces, though they cannot predict all possible application states.

# Software Testing

Software testing verifies that parts of an application work correctly, catching bugs early in development. The longer it takes to detect a bug, the more expensive it becomes to fix.

## Benefits of Testing

- **Early Bug Detection** - Catches issues before they reach production
- **Confident Modifications** - Enables altering system behavior with confidence
- **Living Documentation** - Tests serve as always up-to-date system documentation  
- **Interface Improvement** - Forces developers to consider client perspectives

## Test Scope

Scope defines the code path under test (System Under Test - SUT):

**Unit Tests** - Validate single component behavior (e.g., a class). Should work with public interfaces only, test state changes rather than action sequences, and focus on behaviors given specific inputs and states.

**Integration Tests** - Verify correct integration with external components. Narrow integration tests exercise code paths communicating with external dependencies. Broad integration tests span multiple services.

**End-to-End Tests** - Validate user-facing scenarios across multiple services. Run in shared environments and are slower, more brittle, but necessary for validating complete user journeys.

## Test Size

Size determines computing resources needed:

- **Small** - Single process, no I/O, fast and deterministic
- **Medium** - Single node with local I/O, some delays possible
- **Large** - Multiple nodes, high non-determinism and delays

## Test Doubles

To avoid increasing test size:

- **Fake** - Lightweight implementation behaving similarly to real dependency
- **Stub** - Function returning same value regardless of input
- **Mock** - Has expectations on how it should be called

Test doubles reduce confidence compared to real implementations. Contract tests provide a good compromise for integration testing.

---
*Related: [[Software Maintainability]], [[Continuous Delivery]], [[Formal Verification]]*
