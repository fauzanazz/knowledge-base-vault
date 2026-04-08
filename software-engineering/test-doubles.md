---
title: "Test Doubles"
category: software-engineering
summary: "Test doubles are lightweight implementations that replace external dependencies in tests to avoid increasing test size and complexity. They include fakes, stubs, and mocks with varying levels of confidence."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:14:19.397Z
---

# Test Doubles

> Test doubles are lightweight implementations that replace external dependencies in tests to avoid increasing test size and complexity. They include fakes, stubs, and mocks with varying levels of confidence.

# Test Doubles

Test doubles are techniques to avoid increasing test size by replacing external dependencies with lightweight implementations during testing.

## Types of Test Doubles

**Fake** - A lightweight implementation of an external dependency that behaves similarly to the real system. Example: an in-memory version of a database.

**Stub** - A function that always returns the same value regardless of input. Provides predictable responses for testing specific scenarios.

**Mock** - Has expectations on how it should be called and is used to test interactions between objects. Verifies that components communicate correctly.

## Trade-offs

Test doubles don't behave exactly like real systems, resulting in lower confidence compared to using actual implementations. The confidence hierarchy from highest to lowest:

1. **Real Implementation** - When fast, deterministic, and has few dependencies
2. **Fake** - Maintained by the same developers as the real implementation
3. **Stub/Mock** - Last resort options offering lowest confidence

## Contract Testing

For integration tests, contract testing provides a good compromise. A contract test defines:
- Request format for an external dependency
- Expected result

The test uses this contract to mock the dependency, while the dependency validates that its system behaves according to the contract. This ensures both sides maintain compatibility.

## Best Practices

Use the smallest possible test for the desired scope while minimizing test doubles. When external dependencies are fast and reliable, prefer real implementations. Reserve stubbing and mocking for scenarios where real implementations are impractical or too slow.

---
*Related: [[Software Testing]], [[Integration Testing]], [[Contract Testing]]*
