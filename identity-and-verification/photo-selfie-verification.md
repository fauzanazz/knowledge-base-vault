---
title: "Photo & Selfie Verification"
category: identity-and-verification
summary: "Photo-based identity verification — selfie matching, real-time capture, profile photo comparison, and integration with liveness detection systems."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Photo & Selfie Verification

> Photo-based identity verification — selfie matching, real-time capture, profile photo comparison, and integration with liveness detection systems.

### How It Works Technically

**Selfie Verification**:
1. User captures photo/video via device camera
2. **Face Detection**: ML model detects face landmarks (eyes, nose, mouth) using CNNs (e.g., MTCNN, RetinaFace, MediaPipe)
3. **Face Matching**: Deep learning embeddings (FaceNet, ArcFace, VGGFace) compare facial geometry between selfie and reference photo (stored ID or profile photo). Match threshold tuned for FAR/FRR trade-off
4. **Quality Checks**: Blur, lighting, occlusion (glasses, mask), face angle detection

**Active Liveness Detection** (challenge-response):
- User prompted to: blink, smile, turn head left/right, open mouth
- System verifies response corresponds to prompt (can't be a static photo)
- Vulnerable to sophisticated deepfakes that simulate head movements

**Passive Liveness Detection** (texture/physics-based):
- Single image or short video analyzed for:
  - **Texture analysis**: Distinguishes skin texture from printed paper or screen
  - **Light reflection patterns**: Real 3D faces reflect light differently than 2D printouts
  - **Depth cues**: Even from monocular camera, subtle depth signals extracted
  - **Micro-movements**: Involuntary physiological signals (micro-saccades, skin blood-flow signals via rPPG)
  - **Frequency analysis**: Screen pixel patterns detectable in photos of screens
- No user action required; sub-second processing
- More resistant to deepfakes due to sub-dermal signal detection

**3D Liveness** (highest security):
- Uses structured light or time-of-flight (ToF) depth sensors (iPhone Face ID, newer Android flagship cameras)
- Creates accurate 3D face map; impossible to spoof with flat image
- Limited to specific hardware

**ISO/IEC 30107-3 Standard**:
- International standard for Biometric Presentation Attack Detection (PAD)
- **APCER** (Attack Presentation Classification Error Rate): % of attacks incorrectly classified as genuine
- **BPCER** (Bonafide Presentation Classification Error Rate): % of genuine presentations rejected
- **iBeta Level 1 & 2 certification**: Third-party testing lab validating liveness algorithm against standardized attack scenarios (Level 2 includes 3D masks, silicone faces)

### Common Providers/SDKs

| Provider | Liveness Type | Certifications | Notes |
|----------|--------------|----------------|-------|
| **Jumio** | Passive + 3D | iBeta Level 2 | Integrated with full KYC platform |
| **Onfido (Entrust)** | Passive + Active | iBeta Level 2 | Strong EU/NA presence |
| **FaceTec** | 3D ZoOm® (zoom-in action) | iBeta Level 1 & 2 | Patented depth capture without depth sensor |
| **iProov** | Passive + illumination challenge | iBeta Level 2 | Pioneered "Genuine Presence Assurance" |
| **Sumsub** | Passive + Active | iBeta Level 2 | Combined ID+liveness workflow |
| **Veriff** | Passive | iBeta Level 2 | Session-based analysis |
| **Microblink** | AI-powered passive | ISO 30107-3 | On-device processing, privacy-first |
| **Kairos/Amazon Rekognition** | Basic face matching | - | Lower-tier, no dedicated liveness |
| **Microsoft Azure Face** | Basic + limited liveness | - | Used by Uber's RTIC |
| **Apple Face ID** | 3D structured light | Apple internal | Hardware-level, on-device only |

### Accuracy/Security Level

- **Face matching accuracy**: Modern systems achieve >99.5% TAR at 0.001% FAR (NIST FRVT benchmarks)
- **Passive liveness**: APCER <5% at BPCER 1% (iBeta Level 2)
- **Deepfake vulnerability**: Growing concern; injection attacks (inserting deepfake at API level, bypassing camera) are an emerging threat
- **Injection attack mitigations**: Certificate pinning, encrypted session tokens, hardware attestation, anomaly detection

### UX Friction Level

- **Passive liveness**: Very Low — just take a selfie
- **Active liveness**: Low–Medium — 1–3 actions required, 5–30 seconds
- **3D liveness**: Low (if hardware supports it) — same as Face ID unlock
- Failure rate for poor lighting or non-frontal poses: 5–15%

### Cost Considerations

- Per-verification: $0.50–$3.00 for combined ID + liveness from enterprise providers
- Volume discounts at 100k+ verifications/month
- On-device models (Microblink, iDenfy): Reduce per-API cost, privacy benefits

### Common Attack Vectors & Mitigations

| Attack | Description | Mitigation |
|--------|-------------|------------|
| **Print Attack** | Photo printed and held to camera | Texture analysis, reflectance detection |
| **Screen Replay** | Video played on screen | Moiré pattern detection, screen pixel analysis |
| **3D Mask** | Silicone/latex mask of victim | iBeta L2 certified models; thermal + depth sensors |
| **Deepfake injection** | Generated video injected into API stream | Certificate pinning, session integrity, hardware attestation, camera frame signing |
| **Adversarial examples** | Specially crafted images to fool ML models | Ensemble models, continuous model updates |
| **Identical twins** | Biometric similarity | FAR tuning; supplemental factors required |

### Real-World Examples

- **Tinder**: Video selfie + profile photo matching (photo verification)
- **Uber**: Real-Time ID Check — periodic selfie vs. stored profile using Microsoft Azure Face API; human review fallback
- **Airbnb**: Selfie + liveness during ID verification
- **Binance**: Facial recognition after ID upload for KYC
- **Wise**: Live photo (selfie) + ID document verification

---