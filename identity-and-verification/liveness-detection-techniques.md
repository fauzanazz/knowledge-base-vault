---
title: "Liveness Detection Techniques"
category: identity-and-verification
summary: "Active vs passive liveness detection, 2D vs 3D technical differences, and Presentation Attack Detection (PAD) per ISO 30107 standard."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Liveness Detection Techniques

> Active vs passive liveness detection, 2D vs 3D technical differences, and Presentation Attack Detection (PAD) per ISO 30107 standard.

Liveness detection (also called **anti-spoofing**) is a security technique that verifies a biometric sample comes from a living, physically present person — not a photograph, pre-recorded video, 3D mask, or AI-generated deepfake.

### 1.1 Active Liveness Detection

Active liveness requires the **user to perform a prompted gesture or action** in real time. The system challenges the user and evaluates their response.

#### Common Active Challenges

| Challenge Type | Description | Spoof Resistance |
|---|---|---|
| **Eye Blink** | Detecting natural blink timing (rate: 15–20 blinks/min) | Medium — videos can fake blinks |
| **Head Turn** | Left/right or up/down head rotation | Medium-High |
| **Smile / Expression** | Specific facial expression on command | Medium |
| **Mouth Open/Close** | Jaw movement detection | Medium |
| **Look at Corners** | Gaze direction tracking | High (when randomized) |
| **Nodding** | Up/down head oscillation | Medium |
| **Touchscreen Tap** | Tap gesture synchronized with visual cue | Medium |

#### How Active Liveness Works
```
1. System presents randomized challenge prompt
2. Camera captures real-time video stream
3. AI analyzes:
   - Temporal continuity of the requested motion
   - Natural movement physics (acceleration, jitter)
   - Facial landmark trajectories (68–468 keypoints)
   - Skin reflectance changes during motion
4. Challenge/response correlation scored
5. Binary pass/fail with confidence score
```

#### Advantages
- High resistance to static photo attacks
- Provides immediate, verifiable human interaction
- Detectable via motion physics — hard to fake with basic videos
- Standard for high-security environments (banking vaults, government systems)

#### Disadvantages
- **UX friction**: Users must perform specific, sometimes uncomfortable actions
- **Accessibility barriers**: People with motor impairments or facial palsy may struggle
- **Predictability risk**: Deterministic challenges (always blink) are vulnerable; randomization is essential
- **Replay attacks**: Advanced adversaries can pre-record the specific requested gesture
- **Network dependency**: Real-time video streaming adds latency

---

### 1.2 Passive Liveness Detection

Passive liveness operates **transparently in the background** — the user does nothing special. The system analyzes a single image or short video captured during the normal selfie-taking flow.

#### Core Technical Signals Analyzed

| Signal Category | Techniques Used |
|---|---|
| **Texture Analysis** | LBP (Local Binary Patterns), HOG, CNN-based micro-texture features — paper/screen surfaces have different texture profiles than skin |
| **Specular Reflection** | Screens produce flat, uniform specular highlights; real faces have non-uniform, curved reflections |
| **Moiré Pattern Detection** | Printed photos and LCD displays create interference fringes detectable in frequency domain |
| **Skin Tone Consistency** | Real faces have sub-surface scattering; flat media does not |
| **Depth Estimation** | Software-based 3D face reconstruction from 2D image using DNN |
| **Micro-Motion Analysis** | Subtle breathing, pulse-induced skin color variation (rPPG), involuntary micro-saccades |
| **Optical Flow** | Natural vs. unnatural motion patterns across video frames |
| **Frequency Domain** | FFT artifacts from screens, printers, and video codecs |
| **Ambient Light Interaction** | How light wraps around 3D facial geometry vs. flat surface |

#### How Passive Liveness Works (Single-Frame)
```
Input: Single selfie image (3MP+)
├── Pre-processing: Face detection, alignment, normalization
├── Feature Extraction Pipeline (parallel):
│   ├── Texture CNN: Skin vs. paper/screen surface classifier
│   ├── Reflection analysis: Specular highlight mapping
│   ├── Frequency analysis: FFT/DCT for moiré/codec artifacts
│   ├── Depth estimation: Monocular 3D depth prediction
│   └── Semantic consistency: Face region geometry coherence
├── Fusion: Ensemble model combining all feature scores
└── Output: Liveness score [0.0–1.0] + binary decision
```

#### Advantages
- **Zero user friction**: Completes in the same time as a normal selfie (~1 second)
- Higher completion rates (fewer drop-offs)
- Accessible to users with physical limitations
- Lower latency for high-volume onboarding pipelines

#### Disadvantages
- More susceptible to high-quality deepfake injection attacks
- Performance degrades in poor lighting conditions
- Requires robust training data to generalize to unseen spoof types
- Typically requires iBeta Level 2 certification to defend against 3D mask attacks

---

### 1.3 Hybrid Liveness Detection

Combines passive and active methods in an **adaptive, risk-based cascade**:

```
Session Start
│
├── Passive Check (always runs, user unaware)
│   ├── PASS (confidence > threshold) → Proceed
│   └── LOW CONFIDENCE → Escalate to Active
│
├── Active Challenge (triggered on suspicion)
│   ├── PASS → Proceed with elevated assurance
│   └── FAIL → Block / flag for manual review
│
└── Injection Attack Detection (server-side, always runs)
    └── Camera metadata + frame integrity validation
```

**Use case**: Fintech apps often start passive (frictionless for legitimate users) and escalate to active challenges for suspicious sessions (VPN, rooted device, anomalous signals).

---

## 2. 2D vs. 3D Liveness Detection

### 2.1 2D Liveness Detection

**2D systems** work exclusively with standard RGB camera input — the typical front-facing camera on any smartphone or webcam.

#### Technical Approach
- **Input**: Standard 2D image or video frames (RGB color space)
- **Analysis methods**:
  - Texture feature extraction (LBP, CNN-based)
  - Frequency domain analysis (FFT, DCT)
  - Color space transformations (HSV, YCbCr)
  - Optical flow and temporal consistency across frames
  - Deep learning binary classifiers trained on spoof vs. live datasets

#### Hardware Requirements
- **Standard 0.3MP+ front camera** — no special hardware
- Works on any web browser or mobile device
- Deployable globally without hardware constraints

#### Strengths
- Universal device compatibility
- Lower cost — no specialized sensor required
- Fast inference (sub-100ms on modern hardware)
- Works via WebRTC in browsers

#### Weaknesses
- Susceptible to **high-quality 3D silicone masks** (without additional signals)
- Struggles with extreme lighting conditions (overexposure, deep shadows)
- Cannot directly measure depth — must infer it algorithmically
- More vulnerable to high-resolution print attacks without strong texture models

---

### 2.2 3D Liveness Detection

**3D systems** capture depth information in addition to color, enabling true geometric facial analysis.

#### Hardware Technologies

| Technology | How It Works | Example Device |
|---|---|---|
| **Structured Light** | Projects an infrared dot or line pattern onto the face; deformation of pattern reveals depth | Apple Face ID (TrueDepth), iPad Pro |
| **Time-of-Flight (ToF)** | Emits infrared light pulses and measures round-trip time per pixel to calculate depth | Samsung Galaxy (ToF sensor), Intel RealSense |
| **Stereo Vision** | Two cameras at known separation; disparity between left/right images yields depth via triangulation | Dual-lens camera arrays |
| **Near-Infrared (NIR)** | Infrared illumination reveals sub-surface vasculature and eliminates ambient light variation | Many enterprise kiosks |
| **Software 3D Estimation** | Pure-software monocular depth estimation via neural network (no special hardware) | FaceTec ZoOm, most mobile SDKs |

#### 3D Pipeline (e.g., Apple Face ID)
```
1. Dot projector emits 30,000 IR dots onto face
2. IR camera captures deformed dot pattern
3. Flood illuminator enables IR selfie capture
4. Neural network processes depth map + IR image
5. Face geometry locked into mathematical model (FaceMap)
6. Match against stored encrypted FaceMap template
7. Neural Engine processing: ~120ms per unlock
```

#### 3D Software Approach (FaceTec ZoOm)
```
1. User moves face forward toward camera (ZoOm in)
2. Camera captures 100+ frames over 2–3 seconds
3. AI analyzes perspective distortion changes across frames
4. Stereo-equivalent depth inferred from motion parallax
5. 3D FaceMap® constructed from any standard 0.3MP+ camera
6. Certified liveness: impossible to fake perspective distortion with flat media
```

#### 2D vs. 3D Comparison Table

| Attribute | 2D Liveness | 3D Liveness |
|---|---|---|
| **Hardware requirement** | Any camera | Depth sensor OR software motion capture |
| **Spoof resistance (photo)** | High | Very High |
| **Spoof resistance (video)** | Medium | High |
| **Spoof resistance (3D mask)** | Low–Medium | High (with real depth) |
| **Spoof resistance (deepfake)** | Low (passive) / Medium (active) | Medium (requires injection detection too) |
| **FAR typical** | 0.1–1.0% | 0.0001–0.01% |
| **User friction** | Minimal | Low (ZoOm-in gesture) or Zero (hardware) |
| **Device compatibility** | Universal | Limited (hardware) / Universal (software) |
| **Cost** | Low | Medium (software) / High (hardware sensor) |
| **Deployment** | Web + Mobile | Primarily Mobile / Kiosk |

---

## 3. Presentation Attack Detection (PAD) — ISO 30107

### 3.1 Standard Overview

**ISO/IEC 30107** is the international standard series governing biometric Presentation Attack Detection, published by ISO/IEC JTC 1/SC 37.

| Part | Title | Status |
|---|---|---|
| **ISO/IEC 30107-1:2023** | Framework — defines terms and taxonomy | Published (Ed. 2, Aug 2023) |
| **ISO/IEC 30107-2** | Biometric presentation attack detection – Part 2: Data formats | Published |
| **ISO/IEC 30107-3:2023** | Testing and reporting — evaluation methodology | Published (Ed. 2, Jan 2023) |
| **ISO/IEC 30107-4** | Profile for mobile applications | Published |

### 3.2 Key Terminology (ISO 30107-1)

| Term | Definition |
|---|---|
| **Presentation Attack (PA)** | Presentation to a biometric capture subsystem with the intent of interfering with the operation of the biometric system |
| **Presentation Attack Instrument (PAI)** | Biometric characteristic or object used in a presentation attack (e.g., printed photo, video on screen, silicone mask) |
| **PAD Mechanism** | Automated determination of whether the biometric data being captured is from a bona fide presentation |
| **Bona Fide Presentation** | Interaction with a biometric system by a live, cooperative subject |
| **Species** | Category of PAI sharing the same material properties (e.g., printed photo = 1 species, silicone mask = another) |

### 3.3 Attack Species Classification (ISO 30107-3 Annex A)

| Level | Species | Examples |
|---|---|---|
| **Level 1** (Basic) | Printed artifact | Flat printed photo on paper |
| **Level 1** | Screen replay | Video of target face on tablet/phone screen |
| **Level 1** | Photo with hole | Printed face with eye holes cut out |
| **Level 2** (Advanced) | 2D mask | Paper/cardboard shaped mask |
| **Level 2** | Rigid 3D face | Resin/latex rigid 3D cast |
| **Level 2** | Flexible 3D mask | Silicone/latex flexible 3D mask |
| **Level 2** | Realistic dummy | Articulated mannequin head |

### 3.4 Key Metrics (ISO 30107-3)

| Metric | Full Name | Meaning |
|---|---|---|
| **APCER** | Attack Presentation Classification Error Rate | % of presentation attacks incorrectly classified as bona fide (= spoof acceptance rate) |
| **BPCER** | Bona Fide Presentation Classification Error Rate | % of genuine presentations incorrectly classified as attack (= false rejection rate for liveness) |
| **ACER** | Average Classification Error Rate | (APCER + BPCER) / 2 — overall balance metric |
| **IAPAR** | Imposter Attack Presentation Accept Rate | iBeta-specific metric for % of PAIs accepted |

### 3.5 iBeta Testing Levels

iBeta Quality Assurance (accredited by NIST NVLAP) conducts ISO 30107-3 conformance testing:

| Level | Time Limit | Artifact Source | Expertise Required | Key Requirement |
|---|---|---|---|---|
| **Level 1** | 8 hours per species | Consumer equipment, cooperative subject, home/office environment | None — unskilled attacker | BPCER ≤ 15%, APCER → 0% |
| **Level 2** | 24–48 hours per species | Professional lab equipment, skilled attacker | Forensic/technical expertise | BPCER ≤ 10%, APCER → 0% |

> **Level 2** covers **3D silicone/latex/resin masks, realistic dolls, and digitally synthesized faces**. Achieving **0% IAPAR** at Level 2 is the gold standard for biometric liveness.

### 3.6 CEN/TS 18099 — Injection Attack Detection Standard

A newer European Technical Specification specifically addressing **digital injection attacks** (deepfakes, virtual cameras). iProov was the first vendor to achieve **CEN/TS 18099 High** (Level 4), certified by Ingenium Biometrics. This standard is the baseline document for the forthcoming global ISO/IEC injection attack standard.

---