---
title: "Verification Orchestration Patterns"
category: identity-and-verification
summary: "Architecture patterns for multi-step verification — sequential, parallel, tiered/progressive pipelines, DAG-based orchestration, state machines, and retry strategies."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification Orchestration Patterns

> Architecture patterns for multi-step verification — sequential, parallel, tiered/progressive pipelines, DAG-based orchestration, state machines, and retry strategies.

Verification orchestration defines *how* and *when* individual verification checks are executed relative to each other. The three foundational patterns—sequential, parallel, and tiered—each have distinct trade-offs in latency, cost, and failure tolerance.

## 1.1 Sequential (Pipeline) Orchestration

Each verification step is gated on the success of the previous one. The output of step N is the input to step N+1.

```
┌────────────────────────────────────────────────────────────┐
│              SEQUENTIAL VERIFICATION PIPELINE              │
└────────────────────────────────────────────────────────────┘

User Input
    │
    ▼
┌─────────┐   PASS   ┌──────────┐   PASS   ┌──────────┐   PASS   ┌─────────┐
│  Email  │─────────▶│  Phone   │─────────▶│ Document │─────────▶│ Liveness│
│ Verify  │          │   OTP    │          │   OCR    │          │ Check   │
└─────────┘          └──────────┘          └──────────┘          └─────────┘
    │ FAIL               │ FAIL                │ FAIL                │ FAIL
    ▼                    ▼                     ▼                     ▼
  Reject             Reject/Retry           Reject/Retry          Reject/Retry

    Result: VERIFIED ──────────────────────────────────────────────────▶
```

**When to use:**
- Each step has a hard dependency on the prior step (e.g., can't do face match without document OCR)
- Regulatory requirement for ordered proof (e.g., must verify email before issuing OTP)
- Early-exit optimization: fail fast without wasting API budget on later steps

**Drawbacks:**
- Total latency = sum of all step latencies
- A slow step blocks everything downstream

**Implementation snippet (pseudocode):**
```
async function sequentialVerify(userId, steps) {
  for (const step of steps) {
    const result = await executeStep(step, userId);
    if (result.status === 'FAIL') {
      if (result.retryable) await scheduleRetry(userId, step);
      else return { status: 'REJECTED', failedAt: step.name };
    }
    await persistStepResult(userId, step.name, result);
  }
  return { status: 'VERIFIED' };
}
```

---

## 1.2 Parallel (Fan-Out / Fan-In) Orchestration

Multiple independent checks fire simultaneously; results are aggregated by a combiner function.

```
┌────────────────────────────────────────────────────────────┐
│              PARALLEL VERIFICATION FAN-OUT/IN              │
└────────────────────────────────────────────────────────────┘

                        ┌──────────────────┐
                        │   Orchestrator   │
                        └────────┬─────────┘
               ┌─────────────────┼──────────────────┐
               │                 │                  │
               ▼                 ▼                  ▼
        ┌────────────┐   ┌─────────────┐   ┌──────────────┐
        │  Sanctions │   │  PEP Check  │   │ Device Finger│
        │  Screen    │   │  (OFAC,UN)  │   │  print Score │
        └──────┬─────┘   └──────┬──────┘   └──────┬───────┘
               │                │                  │
               └────────────────┼──────────────────┘
                                ▼
                    ┌───────────────────────┐
                    │   Result Aggregator   │
                    │  (Voting / Weighted)  │
                    └───────────┬───────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
              ALL_PASS = ACCEPT        ANY_FAIL = REVIEW/REJECT
```

**Aggregation strategies:**
- **Unanimous**: All checks must pass → highest security, highest false-positive rate
- **Majority vote**: ≥50% must pass
- **Weighted score**: Each check contributes to a composite risk score; threshold determines outcome
- **Veto override**: Any single critical check can REJECT (e.g., OFAC hit)

**When to use:**
- Checks are truly independent (sanctions, device, email reputation)
- Latency matters and checks can run concurrently
- Best for enrichment layers (PEP, sanctions, adverse media)

---

## 1.3 Tiered / Progressive Orchestration

Verification depth increases with risk level. Cheap checks run first; expensive checks only trigger based on risk signals from prior tiers.

```
┌────────────────────────────────────────────────────────────┐
│              TIERED VERIFICATION ARCHITECTURE              │
└────────────────────────────────────────────────────────────┘

TIER 0 — FREE / INSTANT (sub-100ms)
┌─────────────────────────────────────────────────────┐
│  IP Reputation │ Device Fingerprint │ Email Format  │
│  Velocity Check│ Geolocation        │ Phone Format  │
└──────────────────────────┬──────────────────────────┘
                           │ Risk Score < 20: proceed to TIER 1
                           │ Risk Score > 70: BLOCK immediately
                           ▼
TIER 1 — LIGHTWEIGHT (1–5s, low cost)
┌─────────────────────────────────────────────────────┐
│  Phone OTP (SMS/WhatsApp) │ Email OTP               │
│  Sanctions/PEP screen     │ Email deliverability    │
└──────────────────────────┬──────────────────────────┘
                           │ Risk Score < 40: proceed to TIER 2
                           │ Risk Score 40–70: enhanced review
                           ▼
TIER 2 — DOCUMENT VERIFICATION (5–30s, medium cost)
┌─────────────────────────────────────────────────────┐
│  Government ID OCR        │ Document Authenticity   │
│  MRZ / Barcode Check      │ Address Verification    │
└──────────────────────────┬──────────────────────────┘
                           │ High-risk users only
                           ▼
TIER 3 — BIOMETRIC (10–60s, high cost)
┌─────────────────────────────────────────────────────┐
│  Liveness Detection       │ Face Match (selfie→ID)  │
│  3D FaceMap               │ Video KYC (human agent) │
└──────────────────────────┬──────────────────────────┘
                           │ Very high risk / regulatory mandate
                           ▼
TIER 4 — ENHANCED DUE DILIGENCE (hours/days, very high cost)
┌─────────────────────────────────────────────────────┐
│  Manual Review Agent      │ Source of Funds Check   │
│  Proof of Address Docs    │ Corporate Registry      │
└─────────────────────────────────────────────────────┘
```

**Key advantage**: ~60–80% of users clear at Tier 0–1 without ever touching expensive biometric APIs, dramatically reducing cost per verified user.

---

## 1.4 DAG-Based Orchestration (Advanced)

For complex platforms, verification steps form a Directed Acyclic Graph (DAG) — some steps can run in parallel, others have dependencies.

```
                 ┌─────────────┐
                 │  Start User │
                 │  Registration│
                 └──────┬──────┘
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
       ┌───────────┐        ┌────────────┐
       │Email Verify│       │Phone Verify│
       └─────┬─────┘        └─────┬──────┘
             │                    │
             └──────────┬─────────┘
                        ▼
              ┌──────────────────┐
              │ Sanctions Screen │ (requires both email+phone)
              └────────┬─────────┘
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       ┌──────────┐      ┌──────────┐
       │  Doc OCR │      │ Device   │
       │  (KYC)   │      │ Forensics│
       └────┬─────┘      └────┬─────┘
            │                 │
            └────────┬────────┘
                     ▼
             ┌───────────────┐
             │ Face Liveness │ (requires DocOCR photo)
             └───────┬───────┘
                     ▼
             ┌───────────────┐
             │ Final Decision│
             └───────────────┘
```

---