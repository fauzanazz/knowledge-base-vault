---
title: "Verification State Machines & Lifecycle Management"
category: identity-and-verification
summary: "State machine design for verification flows — core states, per-step states, lifecycle events, idempotent transitions, and verification lifecycle management."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification State Machines & Lifecycle Management

> State machine design for verification flows — core states, per-step states, lifecycle events, idempotent transitions, and verification lifecycle management.

## 3.1 Core Verification State Machine

Every user's verification record transitions through well-defined states:

```
┌────────────────────────────────────────────────────────────┐
│            VERIFICATION STATE MACHINE                      │
└────────────────────────────────────────────────────────────┘

                     ┌──────────┐
                     │  NONE /  │
                     │ ANONYMOUS│
                     └─────┬────┘
                           │ user initiates registration
                           ▼
                     ┌──────────┐
                     │INITIATED │◄─────────────────┐
                     └─────┬────┘                  │ session timeout
                           │ user submits data      │ / restart
                           ▼                        │
                     ┌──────────┐                   │
                     │ PENDING  │                   │
                     │PROCESSING│                   │
                     └─────┬────┘                   │
              ┌────────────┼────────────┐            │
              │            │            │            │
              ▼            ▼            ▼            │
       ┌──────────┐  ┌──────────┐ ┌──────────┐      │
       │  PASSED  │  │  FAILED  │ │ REQUIRES │──────┘
       │(VERIFIED)│  │(REJECTED)│ │  RETRY   │
       └─────┬────┘  └─────┬────┘ └──────────┘
             │             │
             │ time passes │ appeal / re-apply
             ▼             ▼
       ┌──────────┐  ┌──────────┐
       │REVERIFIED│  │ MANUAL   │
       │(periodic)│  │ REVIEW   │
       └─────┬────┘  └─────┬────┘
             │             │
             │             ├── Approved → PASSED
             │             └── Denied  → REJECTED
             │
             │ document expires / inactivity decay
             ▼
       ┌──────────┐
       │ STALE /  │◄─── trigger re-verification flow
       │ EXPIRED  │
       └──────────┘
             │
             │ account closed
             ▼
       ┌──────────┐
       │TERMINATED│ (data retention policy applies)
       └──────────┘
```

## 3.2 Per-Step State Machine

Each individual verification check within a session has its own state:

```
STEP STATES:
  initiated → submitted → processing → passed
                                    └──→ failed
                                    └──→ requires_retry → initiated (loop, max N)
                                    └──→ cancelled
                                    └──→ skipped (risk threshold not met)
```

## 3.3 Lifecycle Events & Triggers

| Event | Trigger | Action |
|---|---|---|
| `verification.initiated` | User starts onboarding | Create session, start TTL timer |
| `verification.document.submitted` | Document uploaded | Run OCR + authenticity pipeline |
| `verification.liveness.passed` | Face match > 95% | Update trust score |
| `verification.failed` | Critical check fails | Notify, log, optional manual review queue |
| `verification.expired` | Session TTL exceeded | Clean up temp data, require restart |
| `verification.stale` | Document expiry or inactivity | Trigger re-verification notification |
| `verification.terminated` | Account deletion | Schedule PII purge per retention policy |

## 3.4 Idempotent State Transitions

State machines in distributed systems must be idempotent — retrying the same event must not cause double-transitions:

```javascript
class VerificationStateMachine {
  static TRANSITIONS = {
    'INITIATED':   ['PENDING_PROCESSING', 'CANCELLED'],
    'PENDING':     ['PASSED', 'FAILED', 'REQUIRES_RETRY'],
    'REQUIRES_RETRY': ['INITIATED'],  // back to start for retry
    'PASSED':      ['STALE', 'TERMINATED'],
    'FAILED':      ['MANUAL_REVIEW', 'TERMINATED'],
    'MANUAL_REVIEW': ['PASSED', 'FAILED'],
    'STALE':       ['INITIATED', 'TERMINATED'],
    'TERMINATED':  [],  // terminal state
  };

  async transition(userId, currentState, event, newState) {
    const allowed = this.TRANSITIONS[currentState] || [];
    if (!allowed.includes(newState)) {
      throw new InvalidTransitionError(`${currentState} → ${newState} not allowed`);
    }
    // Use optimistic locking or DB CAS to prevent race conditions
    await this.db.updateStateAtomic(userId, currentState, newState, event);
    await this.emitEvent(userId, event, { from: currentState, to: newState });
  }
}
```

---