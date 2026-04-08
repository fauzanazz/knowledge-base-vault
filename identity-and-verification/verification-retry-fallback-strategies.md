---
title: "Verification Retry & Fallback Strategies"
category: identity-and-verification
summary: "Retry and fallback patterns for verification — exponential backoff, channel fallback chains, circuit breakers, and retry UX patterns."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification Retry & Fallback Strategies

> Retry and fallback patterns for verification — exponential backoff, channel fallback chains, circuit breakers, and retry UX patterns.

## 9.1 Retry Strategy Framework

```
┌────────────────────────────────────────────────────────────┐
│              RETRY DECISION TREE                           │
└────────────────────────────────────────────────────────────┘

Verification attempt FAILS
         │
         ├── Is it a TRANSIENT error? (network timeout, 5xx)
         │         YES → Exponential backoff retry
         │                └── Delay = base × 2^attempt ± jitter
         │                    Attempt 1: 1s  ±0.5s
         │                    Attempt 2: 2s  ±1s
         │                    Attempt 3: 4s  ±2s
         │                    Attempt 4: 8s  ±4s
         │                    Attempt 5: 16s ±8s → CIRCUIT BREAK
         │
         ├── Is it a USER error? (wrong code, bad photo)
         │         YES → User-facing retry with guidance
         │                └── "Your ID photo was blurry, try again"
         │                    Max 3 user attempts per session
         │                    After 3: REQUIRES_RETRY state → cooldown
         │
         ├── Is it a DEFINITIVE failure? (OFAC hit, document fraud)
         │         YES → NO RETRY → REJECTED state
         │                └── Log for manual review
         │                    Alert fraud team
         │                    Restrict account
         │
         └── Is it a PROVIDER error? (third-party API down)
                   YES → FALLBACK to alternative provider
```

## 9.2 Channel Fallback Chain

```
OTP VERIFICATION FALLBACK CHAIN:

Primary:   SMS OTP
           ↓ (fails: SIM not available, SMS blocked)
Fallback1: WhatsApp OTP (same phone number)
           ↓ (fails: no WhatsApp)
Fallback2: Voice call OTP (robot reads 6-digit code)
           ↓ (fails: no phone signal)
Fallback3: Email OTP (verified email)
           ↓ (fails: no email verified)
Fallback4: TOTP (authenticator app, if enrolled)
           ↓ (fails: not enrolled)
Fallback5: Manual verification (human agent call)

DOCUMENT VERIFICATION FALLBACK:

Primary:   Automated AI (Jumio/Onfido)
           ↓ (fails: low confidence score < 70%)
Fallback1: Retry with image quality guidance
           ↓ (fails after 2 retries)
Fallback2: Alternative document type (passport vs KTP)
           ↓ (still fails)
Fallback3: Manual review queue (human agent)
           ↓ (queue overflow)
Fallback4: Video KYC (live agent call for verification)
```

## 9.3 Circuit Breaker Pattern

Prevents cascading failures when a third-party provider is degraded:

```
┌────────────────────────────────────────────────────────────┐
│                    CIRCUIT BREAKER                         │
└────────────────────────────────────────────────────────────┘

States:
  CLOSED ──[failure threshold exceeded]──▶ OPEN ──[timeout]──▶ HALF-OPEN
  CLOSED ◀──────────────────────────────── HALF-OPEN ◀──[success]──┘

CLOSED (normal operation):
  • All requests pass through to provider
  • Track failure count in rolling 60s window
  • If failures > 50%: → OPEN

OPEN (provider is down):
  • All requests immediately fail-fast (no API call)
  • Use cached last-known-good response if available
  • Route to fallback provider automatically
  • Wait 30 seconds → HALF-OPEN

HALF-OPEN (testing recovery):
  • Route 10% of traffic to provider
  • If success: → CLOSED
  • If still failing: → OPEN (reset timer)

IMPLEMENTATION:
  Per provider: CircuitBreaker('jumio', { threshold: 0.5, timeout: 30_000 })
  Per channel: CircuitBreaker('sms_otp', { threshold: 0.3, timeout: 60_000 })
```

## 9.4 Retry UX Patterns

```
USER-FACING RETRY GUIDANCE:

Failure Type          UX Message                          Next Action
─────────────────────────────────────────────────────────────────────
Blurry document       "Please retake in better lighting"  Re-capture
Partial document      "Ensure all corners are visible"    Re-capture
Face not matching     "Please remove glasses/hat"         Re-selfie
OTP expired           "Code expired, send a new one"      Resend OTP
OTP wrong (1–2x)      "Incorrect code, X attempts left"   Re-enter
OTP wrong (3x)        "Please request a new code"         Rate-limited
Phone unavailable     "Try WhatsApp or email instead"     Channel switch
Doc type unsupported  "Please use KTP or Passport"        Doc switch
System error          "Temporarily unavailable, retry..."  Auto-retry
Definitive rejection  "We couldn't verify. Contact support" Human agent
```

---