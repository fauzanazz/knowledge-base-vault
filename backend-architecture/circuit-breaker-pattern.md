---
title: "Circuit Breaker Pattern"
category: backend-architecture
summary: "A resilience pattern that detects failing downstream dependencies and 'opens' to fail fast — preventing cascading failures by stopping calls to unhealthy services and periodically probing for recovery."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Circuit Breaker Pattern

> A resilience pattern that detects failing downstream dependencies and 'opens' to fail fast — preventing cascading failures by stopping calls to unhealthy services and periodically probing for recovery.

## Overview

Named after electrical circuit breakers that cut power when a fault is detected, the software Circuit Breaker (introduced to software by Michael Nygard in *Release It!* 2007) monitors calls to remote services. When failure exceeds a threshold, it "trips" — subsequent calls immediately return a fallback without hitting the failing service. This protects your service from waiting on timeouts and prevents cascading failures across the system.

Netflix popularized this pattern with **Hystrix** (now in maintenance mode); the modern standard is **Resilience4j** (Java) and similar libraries.

## The Three States

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   CLOSED ─────[failures exceed threshold]────► OPEN    │
│     ▲                                           │       │
│     │                                  [after wait      │
│     │                                   timeout]        │
│     │                                           │       │
│   [success]                                     ▼       │
│     └─────────────────────────── HALF-OPEN      │       │
│                                   │             │       │
│                              [probe fails]      │       │
│                                   └────────────►┘       │
└─────────────────────────────────────────────────────────┘
```

### CLOSED (Normal Operation)
Requests pass through to the downstream service. Failures are counted. When the failure rate exceeds the threshold, the circuit **trips** to OPEN.

### OPEN (Fail Fast)
All calls immediately return a fallback (exception, cached response, or default value). The downstream service is not called. After a configured **wait duration**, the circuit transitions to HALF-OPEN.

### HALF-OPEN (Probing)
A limited number of probe requests are allowed through. If they succeed, the circuit closes. If they fail, the circuit re-opens and the timer resets.

## Implementation: Resilience4j (Java)

```java
// Circuit breaker configuration
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)              // Open when 50% of calls fail
    .slowCallRateThreshold(80)             // Also trip for slow calls
    .slowCallDurationThreshold(Duration.ofSeconds(2))
    .minimumNumberOfCalls(10)              // Need 10 calls before evaluating
    .slidingWindowSize(20)                 // Rolling window of last 20 calls
    .waitDurationInOpenState(Duration.ofSeconds(30))  // Stay open 30s
    .permittedNumberOfCallsInHalfOpenState(5)         // 5 probe calls
    .automaticTransitionFromOpenToHalfOpenEnabled(true)
    .build();

CircuitBreaker circuitBreaker = CircuitBreaker.of("paymentService", config);

// Decorate the call
CheckedFunction0<PaymentResult> decoratedCall = CircuitBreaker
    .decorateCheckedSupplier(circuitBreaker,
        () -> paymentServiceClient.charge(orderId, amount));

// Execute with fallback
Try<PaymentResult> result = Try.of(decoratedCall)
    .recover(CallNotPermittedException.class,
        ex -> PaymentResult.queued(orderId))      // Circuit OPEN fallback
    .recover(Exception.class,
        ex -> PaymentResult.failed(orderId, ex)); // Other failures
```

### Count-Based vs. Time-Based Sliding Window

```java
// Count-based: evaluate last N calls
CircuitBreakerConfig.custom()
    .slidingWindowType(SlidingWindowType.COUNT_BASED)
    .slidingWindowSize(10)   // last 10 calls
    .build();

// Time-based: evaluate calls in last N seconds
CircuitBreakerConfig.custom()
    .slidingWindowType(SlidingWindowType.TIME_BASED)
    .slidingWindowSize(10)   // last 10 seconds
    .build();
```

## Implementation: Python (with tenacity + custom)

```python
import time
from enum import Enum
from threading import Lock

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.state = "CLOSED"
        self.last_failure_time = None
        self._lock = Lock()

    def call(self, func, *args, fallback=None, **kwargs):
        with self._lock:
            if self.state == "OPEN":
                if time.time() - self.last_failure_time > self.recovery_timeout:
                    self.state = "HALF_OPEN"
                else:
                    return fallback() if fallback else None  # Fail fast

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            if fallback:
                return fallback()
            raise

    def _on_success(self):
        with self._lock:
            self.failure_count = 0
            self.state = "CLOSED"

    def _on_failure(self):
        with self._lock:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
```

## Netflix Hystrix (Historical Reference)

Hystrix (now deprecated, in maintenance mode) pioneered circuit breakers at scale:

```java
// Hystrix command pattern (legacy)
public class GetUserCommand extends HystrixCommand<User> {
    private final String userId;

    public GetUserCommand(String userId) {
        super(HystrixCommandGroupKey.Factory.asKey("UserService"));
        this.userId = userId;
    }

    @Override
    protected User run() {
        return userServiceClient.getUser(userId);
    }

    @Override
    protected User getFallback() {
        return User.anonymous(); // Return cached/default when open
    }
}
```

Netflix ran Hystrix across 800+ services. The Hystrix Dashboard provided real-time circuit state visualization. Now replaced by Resilience4j in modern Spring Boot applications.

## Fallback Strategies

The quality of fallback determines user experience when circuits are open:

```java
// Fallback hierarchy (try best available option)
public ProductDetails getProductDetails(String productId) {
    return circuitBreaker.executeSupplier(() -> productService.get(productId))
        // Level 1: Try cache
        .recover(e -> cache.get(productId))
        // Level 2: Return degraded response
        .recover(e -> ProductDetails.minimal(productId, "Details temporarily unavailable"))
        // Level 3: Propagate if nothing works
        .get();
}
```

**Fallback types**:
- **Cached response**: Return last known good data (with staleness indicator)
- **Stub/default**: Return a safe default value
- **Alternative service**: Call a secondary/read replica
- **Fail fast with clear error**: Return 503 immediately rather than timing out

## Observability

Monitor circuit breaker state as a key operational metric:

```
Metrics to track:
- circuit_breaker_state{service="payment"} — 0=CLOSED, 1=HALF_OPEN, 2=OPEN
- circuit_breaker_calls_total{outcome="success|failure|not_permitted"}
- circuit_breaker_failure_rate{service="payment"}
- circuit_breaker_slow_call_rate{service="payment"}

Alert: circuit_breaker_state == 2 for > 5 minutes
```

## Real-World Examples

**Netflix**: All service calls wrapped in circuit breakers. When a recommendation service fails, users see "Top Picks for You" with cached/fallback recommendations rather than a blank page. 99.99% availability maintained even when individual services degrade.

**Uber**: Circuit breakers protect trip dispatch from slow mapping or ETA calculation services. If ETA calculation is degraded, trips can still be dispatched with "ETA unavailable" rather than blocking the ride request.

**Amazon**: Product pages use circuit breakers for all non-critical widgets (recommendations, reviews, sponsored ads). Each widget can individually fail gracefully — the page renders without it rather than returning an error.

## Circuit Breaker + Bulkhead

Combine for defense in depth:
```
Request → [Bulkhead: limit concurrency]
              → [Circuit Breaker: fail fast if service unhealthy]
                  → Remote Service
```
- **Bulkhead** prevents resource exhaustion from slow services
- **Circuit Breaker** prevents calls to known-failed services
- Together: no request waits long for an unhealthy downstream

## Trade-offs

| Pros | Cons |
|------|------|
| Prevents cascading failures | Adds latency overhead (~microseconds per call) |
| Fail fast reduces user wait time | False positives: may open circuit on transient failures |
| Automatic recovery detection | Fallback design requires careful thought |
| System stability during partial outages | Configuration tuning required per service |
| Provides visibility into service health | Doesn't fix the underlying failing service |

## Anti-Patterns

- **Circuit breaker with no fallback**: Opens the circuit but returns errors to users anyway — adds state complexity with no UX benefit
- **Too-sensitive thresholds**: Opening on the first failure causes constant tripping; use sliding windows with minimum call counts
- **Wrapping internal (non-network) calls**: Circuit breakers are for remote calls, not in-memory operations
- **Ignoring circuit state in monitoring**: Circuit open = service incident; must alert

## When to Use

✅ **Use when**: Any synchronous call to a remote service (HTTP, gRPC, DB); systems requiring graceful degradation; microservice meshes where cascades are a risk.

❌ **Avoid when**: Internal in-process calls; idempotent operations where retrying is safe and preferred; when the fallback is indistinguishable from the real response anyway.

---
*Related: [[Bulkhead Pattern]], [[Service Mesh]], [[API Gateway Pattern]], [[Shared Nothing Architecture]]*
