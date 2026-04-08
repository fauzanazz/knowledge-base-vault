---
title: "Third-Party Verification Service Architecture"
category: identity-and-verification
summary: "How Jumio, Onfido, FaceTec, and Twilio Verify work internally — processing pipelines, AI models, API patterns, and integration architecture."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Third-Party Verification Service Architecture

> How Jumio, Onfido, FaceTec, and Twilio Verify work internally — processing pipelines, AI models, API patterns, and integration architecture.

## 4.1 How Jumio Works (Internal Pipeline)

Jumio's KYX Platform processes identity verification through a multi-layer AI pipeline:

```
┌────────────────────────────────────────────────────────────┐
│                    JUMIO KYX PLATFORM                      │
└────────────────────────────────────────────────────────────┘

Client SDK (iOS/Android/Web)
         │
         │ Encrypted: ID images + selfie video
         ▼
┌─────────────────────────────────────────────────────┐
│                  INTAKE & PRE-PROCESSING             │
│  • Image quality assessment (blur, glare, coverage)  │
│  • Document type classification (ML model)           │
│  • Region/country detection                          │
└──────────────────────────┬──────────────────────────┘
                           │
         ┌─────────────────┼──────────────────┐
         ▼                 ▼                  ▼
┌──────────────┐  ┌───────────────┐  ┌──────────────────┐
│  OCR Engine  │  │  Authenticity │  │  Biometric Engine│
│ • MRZ parse  │  │  • Hologram   │  │  • Face extract  │
│ • Barcode    │  │  • UV/IR sigs │  │  • 3D FaceMap    │
│ • Text fields│  │  • Micro-print│  │  • Liveness det. │
│ • Expiry chk │  │  • Font checks│  │  (via FaceTec)   │
└──────┬───────┘  └───────┬───────┘  └──────────┬───────┘
       │                  │                      │
       └──────────────────┼──────────────────────┘
                          ▼
             ┌───────────────────────────┐
             │    ML DECISION ENGINE     │
             │ • 10,000+ fraud signals   │
             │ • Cross-reference DB      │
             │ • Anomaly detection       │
             │ • Synthetic ID detection  │
             └─────────────┬─────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
    APPROVED (auto)             MANUAL REVIEW QUEUE
    → webhook to client         → Jumio analyst portal
    → confidence score          → Human decision
    → extracted data            → 24/7 global centers
```

**Key Jumio differentiators:**
- **PCI DSS Level 1 compliance** — highest financial data security standard
- **AES-256 encryption** at rest and in transit
- **3D Face Mapping** via FaceTec integration: 100× more biometric data than 2D
- **Continuous risk monitoring**: not just onboarding — ongoing transaction-level checks
- **170M+ identities verified**: model trained on massive fraud dataset

---

## 4.2 How Onfido / Atlas AI Works

Onfido's "Atlas AI" uses a **micro-model architecture** — over 10,000 specialized ML models rather than one generalized model:

```
┌────────────────────────────────────────────────────────────┐
│                  ONFIDO ATLAS AI ENGINE                    │
└────────────────────────────────────────────────────────────┘

Document submitted
         │
         ▼
┌─────────────────────────────────────────────────────┐
│              MICRO-MODEL ORCHESTRATION               │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ Model A  │ │ Model B  │ │ Model C  │ │ ...10K+│ │
│  │(font     │ │(hologram │ │(MRZ      │ │ models │ │
│  │ analysis)│ │ pattern) │ │ checksum)│ │        │ │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬───┘ │
│       │             │             │              │    │
│       └─────────────┴─────────────┴──────────────┘   │
│                             │                        │
│                    ┌────────▼────────┐               │
│                    │ Ensemble Merger │               │
│                    │ (weighted vote) │               │
│                    └────────┬────────┘               │
└─────────────────────────────┼───────────────────────┘
                              ▼
                  ┌───────────────────────┐
                  │   Authenticity Score  │
                  │   + Fraud Markers     │
                  └───────────────────────┘
```

**Atlas AI advantages:**
- Detects **50% more document fraud** than generalized models
- Trained on **2,500+ document types** from 195+ countries
- Passive liveness detection: no user gestures needed, analyzes micro-movements & skin texture
- Fraud Lab: dedicated internal red team testing adversarial attacks (deepfakes, 3D prints, silicone masks)

---

## 4.3 How FaceTec Works

FaceTec specializes in **3D face liveness** — the strongest biometric layer:

```
USER ACTION: 2-second video selfie
         │
         ▼
┌─────────────────────────────────────────────────────┐
│               FACETEC 3D PROCESSING                  │
│                                                     │
│  1. PERSPECTIVE DISTORTION ANALYSIS                 │
│     └── Proves face is 3D, not flat photo/screen    │
│                                                     │
│  2. 50+ HUMAN LIVENESS TRAITS                       │
│     ├── Micro-muscle movements                      │
│     ├── Skin texture frequency analysis             │
│     ├── Blood flow detection (rPPG)                 │
│     ├── Specular reflection patterns                │
│     └── Depth map consistency                       │
│                                                     │
│  3. 3D FACEMAP GENERATION                           │
│     └── Encrypted proprietary biometric template   │
│         stored in cloud (not raw image)             │
│                                                     │
│  4. COMPARISON                                      │
│     ├── FaceMap vs ID photo → match score           │
│     └── FaceMap vs stored enrollment → auth score  │
└─────────────────────────────────────────────────────┘
         │
         ▼
 NIST/iBeta Level 1 & 2 Anti-Spoofing Certified
 → Only software to achieve BOTH levels
```

**Key technical detail**: FaceMaps are mathematical representations, NOT photos. Even if leaked, they cannot reconstruct the user's face image — privacy-by-design.

---

## 4.4 How Twilio Verify Works

Twilio Verify handles **phone/OTP verification** with multi-channel fallback:

```
┌────────────────────────────────────────────────────────────┐
│                  TWILIO VERIFY ARCHITECTURE                │
└────────────────────────────────────────────────────────────┘

Your App                      Twilio Verify API
    │                               │
    │─── POST /Verifications ───────▶│
    │    { to: "+62812xxx", channel: "sms" }
    │                               │
    │                    ┌──────────▼──────────┐
    │                    │  OTP GENERATION      │
    │                    │  • Cryptographically │
    │                    │    secure random     │
    │                    │  • TOTP-compatible   │
    │                    │  • 6-digit default   │
    │                    └──────────┬──────────┘
    │                               │
    │                    ┌──────────▼──────────┐
    │                    │  CHANNEL ROUTING     │
    │                    │  • SMS (default)     │
    │                    │  • WhatsApp          │
    │                    │  • Voice call        │
    │                    │  • Email             │
    │                    │  • TOTP (app-based)  │
    │                    │  • Silent network    │
    │                    └──────────┬──────────┘
    │                               │
    │◄── { status: "pending" } ─────┘
    │
    │  [User receives OTP, enters in your app]
    │
    │─── POST /VerificationCheck ────▶
    │    { to: "+62812xxx", code: "123456" }
    │                               │
    │                    ┌──────────▼──────────┐
    │                    │  VERIFICATION CHECK  │
    │                    │  • Compare codes     │
    │                    │  • Check expiry (10m)│
    │                    │  • Rate limit check  │
    │                    │  • Max attempts (5)  │
    │                    └──────────┬──────────┘
    │◄── { status: "approved" } ────┘

FRAUD GUARD (built-in):
  • Detects SMS pumping attacks
  • Blocks high-cost country flooding
  • Anomaly detection on verification patterns
  • Rate limits: per-phone, per-IP, per-service
```

**Twilio Verify internals:**
- OTP generated using cryptographically secure PRNG (not TOTP by default, but TOTP-compatible)
- OTP stored **hashed** (bcrypt/SHA-256), never in plaintext
- Auto-invalidation: 10-minute expiry, 5-attempt max, then OTP burned
- Fraud Guard: monitors for SMS pumping (attackers triggering mass SMS sends to premium-rate numbers)

---