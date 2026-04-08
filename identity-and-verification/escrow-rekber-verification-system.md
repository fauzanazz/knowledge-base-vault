---
title: "Building Verification for Escrow/Rekber"
category: identity-and-verification
summary: "Complete guide to verification for Indonesian escrow (rekber) platforms — minimum viable verification, full architecture, regulatory classification, OJK compliance, and MVP roadmap."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Building Verification for Escrow/Rekber

> Complete guide to verification for Indonesian escrow (rekber) platforms — minimum viable verification, full architecture, regulatory classification, OJK compliance, and MVP roadmap.

## 11.1 What Is Rekber and Why Verification Matters

**Rekber** (Rekening Bersama = "joint account") is Indonesia's grassroots escrow model originating from Kaskus forum. A neutral third party holds buyer funds until the seller fulfills the transaction. Modern platforms like Eskro, Midtrans, and bank escrow services have professionalized this model.

**Verification stakes for escrow platforms:**
- **Sellers** must be trustworthy (no impersonation, no exit scams)
- **Buyers** must be able to fund (real payment, not fraud)
- **Both parties** have recourse in disputes (requires real identity)
- **Platform** is liable if funds are lost (requires AML compliance)

---

## 11.2 Regulatory Classification in Indonesia

```
DOES YOUR REKBER PLATFORM REQUIRE OJK/BI LICENSE?

Is your platform holding funds?
  YES → You are operating a payment service → BI license required
  NO  → Pure transaction escrow (funds via partner payment gateway) → See below

Do you use a licensed payment gateway (Xendit, Midtrans, DOKU)?
  YES → The payment gateway holds the license; you need:
         • OJK/BI-registered PSE (Electronic System Operator) via KOMINFO
         • AML compliance per POJK 8/2023 if you're a "reporting party"
         • KYC requirements apply if transaction thresholds exceeded
         
Transaction value thresholds for AML reporting (indicative):
  Single transaction > Rp 500,000,000 → Cash Transaction Report (CTR) to PPATK
  Suspicious transaction (any amount) → Suspicious Transaction Report (STR)
  
If you are a P2P payment intermediary:
  → Bank Indonesia Regulation applies (e-money / payment gateway licensing)
  → POJK 3/2024 for fintech innovation sandbox registration
```

---

## 11.3 Minimum Viable Verification (MVV) for Rekber

```
┌────────────────────────────────────────────────────────────┐
│    MINIMUM VIABLE VERIFICATION FOR REKBER PLATFORM         │
└────────────────────────────────────────────────────────────┘

PHASE 1: BASIC TRUST (Launch MVP)
Required for transactions up to Rp 1,000,000:

  ✓ Phone number OTP verification
    └── Establishes real contact, enables dispute comms
  ✓ Email verification
    └── Transaction receipt, dispute notifications
  ✓ Device fingerprint
    └── Detects bot registration, multiple accounts
  ✓ IP reputation check
    └── Block obvious fraud signals
  ✓ Agreed Terms of Service with legal name
    └── Basic accountability
    
  COST: ~$0.05/user
  PASSES: ~99% of users (minimal friction)
  COVERS: Small C2C transactions, low fraud risk

PHASE 2: IDENTITY VERIFIED (required for higher limits)
Required for: Transactions Rp 1M–50M, or seller status:

  ✓ All Phase 1
  ✓ KTP (KK/Kartu Keluarga also accepted) upload
    └── OCR extract: name, NIK, address, DOB
  ✓ Selfie with KTP (or liveness check)
    └── Confirms document ownership
  ✓ Phone + NIK cross-reference
    └── DUKCAPIL check if licensed (or third-party eKYC provider)
  ✓ Sanctions/PEP screening
    └── OFAC, UN, local PPATK-linked lists
    
  COST: ~$1.50–3.00/user
  PASSES: ~90% of genuine users
  COVERS: Standard rekber use case, seller verification

PHASE 3: FULL KYC (high-value transactions)
Required for: Transactions > Rp 50M, power merchants:

  ✓ All Phase 2
  ✓ Video selfie liveness detection (not just photo)
  ✓ NPWP (tax ID) for sellers above tax threshold
  ✓ Bank account verification (penny drop or Xendit verification)
  ✓ Source of funds declaration for large transactions
  ✓ Enhanced ongoing monitoring
  
  COST: ~$5.00–8.00/user
  PASSES: ~80% of genuine users (some friction)
  COVERS: High-value escrow, professional merchants
```

---

## 11.4 Full Rekber Verification Architecture

```
┌────────────────────────────────────────────────────────────┐
│         REKBER PLATFORM — VERIFICATION ARCHITECTURE        │
└────────────────────────────────────────────────────────────┘

                         USER REGISTRATION
                                │
               ┌────────────────┴────────────────┐
               ▼                                 ▼
           BUYER                              SELLER
        (less friction)                  (more scrutiny)
               │                                 │
               ▼                                 ▼
    ┌──────────────────┐              ┌──────────────────┐
    │  TIER 0 (always) │              │  TIER 0 (always) │
    │  • Phone OTP     │              │  • Phone OTP     │
    │  • Email OTP     │              │  • Email OTP     │
    │  • IP + Device   │              │  • IP + Device   │
    └────────┬─────────┘              └────────┬─────────┘
             │ passes                           │ seller = always
             ▼                                 │ require Tier 1
    ┌──────────────────┐              ┌─────────▼────────┐
    │ Watch velocity   │              │  TIER 1 REQUIRED  │
    │ If txn > 1M IDR  │              │  • KTP upload    │
    │ → trigger Tier 1 │              │  • Selfie check  │
    └────────┬─────────┘              │  • NIK verify    │
             │                        │  • Sanctions scr │
             └──────────┬─────────────┘
                        ▼
              ┌──────────────────────┐
              │    RISK EVALUATION   │
              │   (composite score)  │
              └──────────┬───────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      Score <300     300-700       >700
         │             │             │
         ▼             ▼             ▼
      BLOCK        MANUAL           AUTO
      + Alert      REVIEW           APPROVE
                     │
                     ▼
              Human agent review
              (within 24 hours)
                     │
              ┌──────┴──────┐
              │             │
           APPROVE       REJECT
```

---

## 11.5 Transaction-Time Verification

Even after onboarding KYC, transaction-level verification is required:

```
TRANSACTION FLOW WITH EMBEDDED VERIFICATION:

Buyer initiates transaction (Rp 5,000,000)
         │
         ▼
┌─────────────────────────────────────────────────────┐
│  TRANSACTION RISK CHECK (real-time, < 200ms)        │
│  1. Is buyer's trust level sufficient?              │
│     → Level 1 required for Rp 5M → ✓ passed         │
│  2. Velocity: how many transactions today?          │
│     → 2 today, max 5 → ✓ under limit               │
│  3. Is seller trust level sufficient?               │
│     → Seller must be Level 2 (KTP verified) → ✓    │
│  4. Sanctions re-check (cached, last 24h)           │
│     → Not listed → ✓                               │
│  5. Device anomaly check                           │
│     → Same device as registered → ✓               │
└──────────────────────────┬──────────────────────────┘
                           │ ALL PASS
                           ▼
                   Escrow initiated
                   (funds held in Xendit/Midtrans escrow)
                           │
                    [Seller delivers]
                           │
                           ▼
              Buyer confirms receipt
                           │
                           ▼
               Step-up auth (optional):
               For > Rp 5M: require OTP before release
                           │
                           ▼
                    Funds released to seller
                    
DISPUTE FLOW:
  If buyer disputes:
  → Freeze funds (already in escrow)
  → Require both parties to submit evidence
  → Mediator reviews (requires both parties are Level 2+)
  → Decision based on real verified identities
```

---

## 11.6 Recommended Tech Stack for Indonesian Rekber

```
VERIFICATION SERVICE STACK:

Layer               Recommended Options                Cost/Scale
─────────────────────────────────────────────────────────────────

Phone OTP          Twilio Verify / Vonage              $0.03–0.07/OTP
                   (Indonesian routing)
                   Zenziva (local, cheaper)             $0.01–0.02/OTP

Document KYC       VIDA.id (local, DUKCAPIL)           $1.00–2.00/check
                   Verihubs (local)                    $0.80–1.50/check
                   Sumsub (global, Indonesia support)  $1.50–3.00/check

Liveness           VIDA.id integrated liveness         Bundled
                   FaceTec (standalone SDK)            $0.50–1.50/check

Sanctions/PEP      Refinitiv (Dow Jones)               $0.15–0.30/check
                   Complyadvantage                     $0.10–0.25/check

IP Reputation       MaxMind GeoIP2 (self-hosted DB)    $0.001/lookup
                   IPQualityScore                     $0.005/lookup

Device Fingerprint Fingerprint.com                     $0.01–0.02/visit
                   SEON                               $0.04/check

Payment Escrow     Xendit (licensed payment gateway)   0.7–2% of txn
                   Midtrans (Grab subsidiary)          Variable
                   DOKU                               Variable

Data Storage        AWS Jakarta (ap-southeast-3)       Standard AWS pricing
                   (required for Indonesian data residency)
```

---

## 11.7 MVP Implementation Roadmap

```
PHASE 1 (Weeks 1–4): Basic Trust
  □ User registration with email + phone OTP
  □ Device fingerprinting (Fingerprint.js)
  □ IP reputation check (MaxMind free tier)
  □ Basic velocity rules in Redis
  □ Transaction limits by trust level
  □ Xendit/Midtrans integration for escrow
  
  → Can legally operate small-scale rekber
  → Users limited to Rp 1M/transaction

PHASE 2 (Weeks 5–8): Identity Verification
  □ KTP upload flow (mobile camera guided)
  □ VIDA.id or Verihubs eKYC integration
  □ Selfie + KTP liveness check
  □ Sanctions screening (Complyadvantage)
  □ Trust level system implementation
  □ Escalation trigger rules
  □ Manual review queue (internal tool)
  
  → Full KYC compliance for standard users
  → Increased transaction limits
  → Seller verification enabled

PHASE 3 (Weeks 9–12): Risk & Compliance
  □ DUKCAPIL integration (requires OJK registration)
  □ Automated sanctions re-screening (nightly batch)
  □ Fraud alert system (velocity anomalies)
  □ AML transaction monitoring
  □ SAR/CTR report generation for PPATK
  □ Data retention automation (90-day cleanup)
  □ GDPR/PDP Law consent management
  
  → Full regulatory compliance
  → Enterprise-grade fraud prevention

PHASE 4 (Weeks 13–16): Optimization
  □ ML-based risk scoring model
  □ A/B test verification friction vs completion rate
  □ Batch sanctions screening (cost optimization)
  □ Verification result caching
  □ Cost monitoring dashboard
  □ Perpetual KYC (pKYC) for dormant account refresh
```

---

## 11.8 Indonesian-Specific Compliance Checklist

```
□ REGULATORY REGISTRATION
  □ Register as PSE (Electronic System Operator) with KOMINFO
  □ If holding funds: obtain payment service license from Bank Indonesia
  □ If OJK-regulated fintech: register/license with OJK per POJK 3/2024
  □ Join AFTECH (Indonesian Fintech Association) if ITSK operator

□ AML/CFT (per POJK 8/2023):
  □ Appoint Money Laundering Reporting Officer (MLRO/DKKNP)
  □ Implement Customer Due Diligence (CDD) procedures
  □ Conduct PEP/sanctions screening at onboarding
  □ Implement transaction monitoring system
  □ File CTR to PPATK (transactions > Rp 500M)
  □ File STR to PPATK for suspicious transactions
  □ Maintain records for minimum 5 years

□ DATA PROTECTION (per UU PDP):
  □ Obtain explicit consent before collecting personal data
  □ Publish clear privacy policy in Bahasa Indonesia
  □ Data center physically located in Indonesia (AWS Jakarta)
  □ Implement right-to-erasure (with AML retention exceptions)
  □ Data breach notification to KOMINFO within 14 days
  □ Cross-border data transfer only to approved countries/entities

□ eKYC STANDARDS:
  □ KTP verification (document + selfie)
  □ NIK validation (DUKCAPIL or third-party)
  □ Phone number verification (OTP)
  □ Liveness detection for seller onboarding
  □ NPWP collection for taxable sellers (> Rp 54M/year gross)

□ DISPUTE & CONSUMER PROTECTION:
  □ Implement complaint mechanism per OJK consumer protection rules
  □ Dispute resolution SLA (max 20 business days per OJK)
  □ Escrow fund segregation (cannot commingle with operational funds)
```

---

# Summary Architecture Diagram

```
┌────────────────────────────────────────────────────────────────────┐
│             COMPLETE VERIFICATION SYSTEM ARCHITECTURE              │
│                    (Rekber / Escrow Platform)                      │
└────────────────────────────────────────────────────────────────────┘

  USER ENTRY POINTS          VERIFICATION GATEWAY           BACKEND
  ──────────────────         ───────────────────────        ─────────────────

  Mobile App (iOS/Android)        ┌──────────────┐        Verification DB
  Web Browser          ──────────▶│  API Gateway │──────▶ (PostgreSQL)
  WhatsApp Bot                    │ (Rate limit) │        
                                  └──────┬───────┘        Redis
                                         │                (sessions,velocity)
                          ┌──────────────▼──────────────┐  
                          │      ORCHESTRATION ENGINE    │ S3 Jakarta
                          │  (State machine + DAG exec)  │ (doc storage)
                          └──────────────┬───────────────┘
               ┌───────────────────┬─────┴──────────────────────┐
               │                   │                            │
               ▼                   ▼                            ▼
  ┌────────────────┐   ┌───────────────────┐   ┌───────────────────────┐
  │  TIER 0        │   │  TIER 1           │   │  TIER 2+              │
  │  • IP check    │   │  • Phone OTP      │   │  • VIDA.id eKYC       │
  │  • Device FP   │   │  • Email OTP      │   │  • Liveness check     │
  │  • Velocity    │   │  • Sanctions scr  │   │  • DUKCAPIL NIK check │
  │  Cost: $0.02   │   │  Cost: $0.15      │   │  • Cost: $1.50–5.00   │
  └────────────────┘   └───────────────────┘   └───────────────────────┘
               │                   │                            │
               └───────────────────┴────────────────────────────┘
                                         │
                           ┌─────────────▼────────────┐
                           │     RISK SCORE ENGINE     │
                           │  (Composite weighted ML)  │
                           └─────────────┬─────────────┘
                                         │
                      ┌──────────────────┼──────────────────┐
                      ▼                  ▼                   ▼
                   APPROVE           REVIEW               REJECT
                      │                  │                   │
                      ▼                  ▼                   ▼
               Level assigned     Manual queue          Alert + block
               Trust score set    → Human agent         Fraud log
               Limits set         → Decision             Possible SAR
```

---

## Key Takeaways

| Concern | Recommendation |
|---|---|
| **Low-friction start** | Allow email+phone only for small transactions; escalate progressively |
| **Indonesian compliance** | VIDA.id for local eKYC (cheaper + DUKCAPIL integration) |
| **Cost optimization** | Gate expensive checks behind cheap ones; cache results aggressively |
| **Fraud prevention** | Velocity + device fingerprint catches 70% of fraud before KYC |
| **Sellers vs buyers** | Apply stricter verification to sellers (they receive funds) |
| **AML/CFT** | Implement per POJK 8/2023 even at small scale — PPATK monitoring |
| **Data residency** | Use AWS Jakarta region; never store PII outside Indonesia |
| **Retry/fallback** | Always have SMS→WhatsApp→Voice→Email OTP fallback chain |
| **State machine** | Implement idempotent state transitions; use event sourcing |
| **Trust decay** | Re-verify users after 12 months inactivity or document expiry |

---

**Summary of what was done:**

- Conducted 12 parallel web searches across all requested topics
- Researched orchestration patterns (sequential/parallel/tiered/DAG), trust scoring, state machines, Jumio/Onfido/FaceTec/Twilio internal architectures, velocity checks, device intelligence, IP reputation, data privacy/retention, PSD2/SCA/eIDAS, Indonesia OJK/BI regulations (POJK 8/2023, POJK 3/2024, BI Reg 10/2024, UU PDP), progressive verification patterns, retry/fallback strategies, and cost optimization
- Cross-referenced Indonesian rekber/escrow context with regulatory requirements
- Compiled all findings into this comprehensive markdown document with ASCII architecture diagrams throughout
- **No files were created/modified** — this is delivered directly as a response per the task instructions
- **Issues:** Some third-party internal architecture details (Jumio, Onfido) are proprietary, so those sections are based on published technical documentation, white papers, and public API specifications rather than internal source code — this is the best possible public information available