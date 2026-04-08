---
title: "Liveness Detection Vendor Implementations"
category: identity-and-verification
summary: "How FaceTec, iProov, Jumio, and Veriff implement liveness detection — processing pipelines, 3D face mapping, and certification levels."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Liveness Detection Vendor Implementations

> How FaceTec, iProov, Jumio, and Veriff implement liveness detection — processing pipelines, 3D face mapping, and certification levels.

### 4.1 FaceTec

**Approach: Software-based 3D Liveness via ZoOm Technology**

FaceTec, founded in 2013, invented the **3D FaceMap®** — a software-derived 3D biometric modality from any standard 2D camera.

#### ZoOm Technology Deep-Dive
```
Capture Process:
├── User centers face in oval guide (12 inches away)
├── User slowly zooms face toward camera (7–8 inches)
├── Camera captures 100+ video frames over 2–3 seconds
├── AI measures perspective distortion across frames:
│   ├── Nose tip displacement vector
│   ├── Eye corner parallax shift
│   ├── Cheekbone geometry change
│   └── Mouth corner trajectory
├── Multi-frame interpolation constructs 3D FaceMap®
└── 3D FaceMap timestamped and non-reusable per session
```

#### Why Perspective Distortion = Proof of 3D
A flat 2D artifact (photo, screen, printed mask) cannot produce the **progressive geometric perspective shift** of a real 3D face moving toward a camera. The nose tip advances faster than the ears; the eye sockets reveal depth. FaceTec's AI is trained on billions of genuine 3D captures to identify when this physics is absent or spoofed.

#### Certifications & Claims
- **iBeta Level 1 & Level 2**: 0% IAPAR achieved
- **$600,000 public Spoof Bounty Program** (active since October 2019, as of 2026 unclaimed)
- Independent testing by Ingenium, Bixelab, Praetorian
- **FAR**: 1/125,000,000 (3D:3D matching); 1/500,000 (3D:2D ID scan matching)
- **FRR**: <1% at published FAR
- Processes **3.7 billion liveness checks annually**
- No specialized hardware required — works on any 0.3MP+ camera

#### Additional Features
- **UR Codes**: Secure, encrypted QR-like identity tokens
- **OCR + NFC**: Document scanning integrated into ZoOm flow
- **Age Estimation**: Anonymous age verification without ID
- **3D Face Matching**: 1:1 matching using stored 3D FaceMaps

---

### 4.2 iProov

**Approach: Science-Based Passive Challenge-Response with Patented Flashmark™ Illumination**

iProov (UK, founded 2011) specializes in **Genuine Presence Assurance** — verifying the user is the right person, a real person, and authenticating *right now*.

#### Product Tiers

| Product | Mechanism | Use Case | Security Level |
|---|---|---|---|
| **Express Liveness** | Multi-frame passive analysis | Rapid conversion, lower-risk flows | Standard |
| **Dynamic Liveness** | Flashmark™ illumination + passive | High-assurance identity verification, deepfake defense | Mission-Critical |

#### Flashmark™ Technology — How It Works
```
Dynamic Liveness Session:
├── Server generates unique, one-time color illumination sequence
├── Device screen displays rapid, randomized color flash pattern
│   (visible to user as subtle background color changes)
├── Camera captures face while illumination changes
├── AI analyzes:
│   ├── Expected vs. actual color reflected off facial skin
│   ├── 3D facial surface light response (cannot be replicated by flat display)
│   ├── Temporal synchronization of reflection with server-known sequence
│   └── Multi-frame biometric consistency
├── One-time biometric: session token cannot be replayed
└── Any injection of pre-recorded/deepfake content fails:
    the attacker cannot predict the future color sequence
```

#### Why Flashmark Defeats Injection Attacks
- The color sequence is **generated server-side, milliseconds before display**
- A deepfake video or replay does **not know the sequence** in advance
- The reflected colors off real skin follow known physics — a screen displaying a face cannot mimic this
- Even if an attacker intercepts the sequence, latency prevents synthesis in real-time

#### Performance & Certifications
- **iBeta Level 1 & Level 2**: 0% APCER
- **CEN/TS 18099 High (Level 4)** for Injection Attack Detection — first globally certified
- **FIDO Alliance certified** — first biometric product certified for face verification
- **NIST SP 800-63-4** — first vendor to meet new U.S. Digital Identity Guidelines
- **eIDAS 2 Level of Assurance High** (equivalent to IAL3/AAL3)
- FRR: 0.14% (Ingenium independent testing, 10,000+ tests, zero successful attacks)
- Completion rate: **98% in production**; average 1.08–1.22 attempts to pass
- Verification time: **1.08–1.22 seconds**
- Scale: **1 million+ daily verifications** (surpassed in 2025)

#### iSOC — Security Operations Center
iProov operates a dedicated biometric threat intelligence unit that continuously monitors attacks globally, feeds findings into model updates, and publishes annual Threat Intelligence Reports (2024, 2025 editions).

---

### 4.3 Jumio

**Approach: Multi-Layer AI-Driven Defense with Passive Liveness + Injection Detection**

Jumio (US, founded 2010) has evolved from document OCR to a comprehensive identity intelligence platform.

#### Liveness Product Lines

| Product | Capabilities |
|---|---|
| **Jumio Liveness** (launched Oct 2024) | Passive liveness, injection attack detection, deepfake AI models; beyond traditional presentation attacks |
| **Jumio Liveness Premium** (launched 2025) | Adds patented **randomized color sequence** + AI analysis for real-time human presence confirmation; deepfake detection layer |

#### Technical Architecture
```
Jumio Multi-Layer Defense:
├── Layer 1: Automated Image Quality Gate
│   └── Blur, glare, occlusion detection; user guidance for optimal capture
├── Layer 2: Passive Liveness (AI model ensemble)
│   ├── Texture, reflection, moiré analysis
│   └── 2D/3D spoof classifier
├── Layer 3: Injection Attack Detection
│   ├── Camera metadata validation
│   ├── Virtual camera fingerprinting
│   └── Frame integrity analysis
├── Layer 4: Deepfake Detection (Premium)
│   ├── Randomized illumination color sequence (patented)
│   ├── GAN/diffusion artifact classifiers
│   └── AI-generated face forensics
├── Layer 5: Behavioral / Context Signals
│   ├── Multiple persons in frame detection
│   ├── Coercion indicator (forced verification)
│   ├── Unconscious subject detection
│   └── Network intelligence (IP reputation, device fingerprint)
└── Layer 6: Continuous Anomaly Monitoring
    └── Real-time traffic analysis → model retraining pipeline
```

#### Certifications & Scale
- **1 billion+ transactions** processed across 200+ countries
- **5,000+ document types** supported
- **300+ issued patents** spanning ~100 patent families
- iBeta Level 1 certified (Level 2 in progress as of 2025)
- Active legal dispute with FaceTec over liveness IP (2024–2025)
- Case study: LATAM fintech client caught **30% more sophisticated fraud** after Liveness Premium deployment

---

### 4.4 Veriff

**Approach: AI-Native Video-Based Verification with Passive Liveness + Behavioral Analysis**

Veriff (Estonia/US, founded 2015) built around **video-native identity verification** processed in an average of **6 seconds**.

#### Core Technology: Verification Station
```
Veriff "Station" Guided Flow:
├── Step 1: Document Capture
│   ├── Automated document type detection (11,500+ types, 230+ countries)
│   ├── NFC chip reading (ePassport)
│   └── OCR + MRZ extraction
├── Step 2: Biometric Capture
│   ├── Passive liveness detection (no user prompts)
│   ├── Face matching: selfie vs. document photo (1:1)
│   └── Age estimation
├── Step 3: Fraud Signals (Fraud Protect)
│   ├── Injection attack detection
│   ├── Deepfake content classification
│   ├── Device/network signals
│   └── Cross-session pattern analysis (connected intelligence)
└── Step 4: Decision Engine
    ├── Automated approval (~95% of sessions)
    └── Manual review queue (edge cases)
```

#### Key Capabilities
- **Passive liveness** — no user action required; runs during normal selfie capture
- **iBeta Level 2 certified** (achieved September 2024): 0% IAPAR on Level 2 presentation attacks
- **Verification speed**: Average 6 seconds end-to-end
- **100% detection rate** of synthetic identity documents (IDNet benchmark, January 2026)
- **Completion rate**: Higher than competitors due to Station guided flow
- **Fraud Protect**: Combines liveness, deepfake detection, behavioral signals, database checks

#### Veriff's AI Stack
- Continuous ML model training on production fraud signals
- Human-in-the-loop feedback for edge cases → model improvement
- Behavioral biometrics: session patterns, interaction timing, device signals

---