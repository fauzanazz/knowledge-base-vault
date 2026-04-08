---
title: "Biometric Verification Systems"
category: identity-and-verification
summary: "Fingerprint, face, voice, and behavioral biometric verification — matching algorithms, template storage, multi-modal fusion, and privacy considerations."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Biometric Verification Systems

> Fingerprint, face, voice, and behavioral biometric verification — matching algorithms, template storage, multi-modal fusion, and privacy considerations.

### Fingerprint Verification

**Technical Implementation**:
- **Touch-based sensors**: Capacitive (detects ridges via electrical charge differential) — most common on phones
- **Under-display optical**: Captures 2D image of fingerprint via display pixels acting as lens
- **Under-display ultrasonic**: Sonic pulses create 3D topographic map (Qualcomm Snapdragon Sense ID) — more secure, works with wet/dirty fingers
- **Template matching**: Minutiae points (ridge endings, bifurcations) extracted and compared against stored template. No raw fingerprint image stored — only mathematical representation
- **On-device vs. server**: Most mobile implementations use on-device Secure Enclave (Apple) or Trusted Execution Environment (TEE/Android Keystore) — biometric data never leaves device

**Standards**: ISO/IEC 19794-2 (fingerprint representation), ANSI 378, FIDO CTAP2

**Providers/SDKs**:
- Apple Touch ID → LocalAuthentication framework (iOS)
- Android BiometricPrompt API (unified API for fingerprint, face, iris)
- Identy.io: Touchless fingerprint from rear camera (rear camera biometric SDK)
- Aware Inc. Knomi: Server-side fingerprint authentication
- NIST CRADA standards for touchless fingerprint

### Face Recognition (On-Device)

- **Apple Face ID**: TrueDepth camera system — structured light infrared dot projector creates 30,000 depth points; infrared camera captures depth map; Neural Engine processes; all on-device in Secure Enclave
  - FAR: 1 in 1,000,000 (stated)
  - Works in dark, with glasses, hats, partial masks
- **Android Face Unlock**: Varies from simple 2D (selfie camera, FAR ~1:50,000) to 3D (Huawei 3D face, Samsung Iris+Face) implementations
- **Windows Hello**: Infrared face recognition via specialized camera; TPM chip stores keys
- FIDO2/WebAuthn exposes biometric authentication to web apps via platform authenticator

### Voice Verification

**Technical Implementation**:
- Speaker recognition extracts voiceprint: fundamental frequency, formants, cadence, spectral characteristics
- **Text-dependent**: User repeats specific passphrase (easier to match; more susceptible to replay)
- **Text-independent**: Verify identity from any spoken content (higher friction, more flexible)
- **Anti-spoofing**: Liveness detection for voice — playback detection (recording artifacts), synthetic speech detection
- Templates stored as mathematical models, not audio recordings

**Providers**:
- Aware Inc. Nexa|Voice: Mobile SDK, iOS/Android, static/dynamic passphrase
- Nuance (Microsoft): Enterprise voice biometrics
- Pindrop: Voice authentication + call center fraud detection
- iBeta voice liveness certification

### Security Levels

| Modality | FAR (typical) | Spoofing Risk | Standard |
|----------|--------------|---------------|----------|
| Fingerprint (capacitive) | 1:50,000 | Gummy finger attacks | ISO 19794-2 |
| Fingerprint (ultrasonic) | 1:100,000 | Much harder to spoof | ISO 19794-2 |
| Face 3D (iPhone) | 1:1,000,000 | Very high (structured light) | ISO/IEC 30107-3 |
| Face 2D | 1:10,000–50,000 | Photo/screen attacks possible | ISO/IEC 30107-3 |
| Voice | 1:10,000 | Replay attacks, TTS synthesis | ETSI ES 202 050 |
| Iris | 1:1,200,000 | Hardest to spoof | ISO/IEC 19794-6 |

### UX Friction Level

- On-device biometrics (Face ID, fingerprint): **Very Low** — sub-second, no typing
- Server-side biometric checks: **Low–Medium** — requires capture + network round-trip
- Voice verification: **Medium** — requires speaking, quiet environment needed

### Regulatory Considerations

- **Illinois BIPA**: Explicit informed consent required before collecting biometric identifiers; $1,000–$5,000/violation
- **GDPR Article 9**: Biometric data = special category data requiring explicit consent or legal basis
- **Texas CUBI**: Similar to BIPA
- **CCPA**: Biometrics as "sensitive personal information" requiring opt-in
- On-device processing (Secure Enclave) may exempt from some regulations as data never leaves device

---