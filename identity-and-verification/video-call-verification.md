---
title: "Video Call Verification"
category: identity-and-verification
summary: "Live agent video verification — real-time identity interviews, liveness confirmation, document presentation, and hybrid human-AI verification pipelines."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Video Call Verification

> Live agent video verification — real-time identity interviews, liveness confirmation, document presentation, and hybrid human-AI verification pipelines.

### How It Works Technically

**Session Setup**:
1. User schedules or joins on-demand video call
2. Secure, encrypted video session (WebRTC-based with E2E encryption)
3. Identity of agent verified; session recorded (with consent, where required)

**Verification Steps During Call**:
1. **Document presentation**: User holds ID to camera; agent captures still frame
2. **OCR + NFC** (if device supports): Automated data extraction during call
3. **Liveness + face matching**: AI system running in background matches live face to ID photo
4. **Security feature verification**: Agent visually inspects hologram, microprinting, document thickness/flexibility
5. **Q&A verification**: Agent asks questions to verify identity (name, DOB, address, reason for account)
6. **Biometric capture**: AI captures biometric template for future authentication
7. **Registry checks**: Real-time sanctions/PEP screening while call progresses
8. **AML risk assessment**: Transaction history discussion for enhanced due diligence

**Regulatory-Driven Requirements**:
- **Germany BaFin Video Ident**: Specific requirements for financial identity via video (used by N26, Deutsche Bank digital)
- **India V-CIP**: RBI (Reserve Bank of India) Video Customer Identification Process requirements — mandatory in-call OTP verification
- **EU AMLD**: Video verification accepted in many EU states as equivalent to in-person
- **Recording**: Must be stored for AML compliance; typical retention 5–7 years

**Asynchronous Video** (emerging):
- User records video selfie with ID on their own time
- Agent reviews recording asynchronously
- AI assists with initial screening before human review
- Lower cost; less scheduling friction

### Common Providers

| Provider | Specialty |
|----------|-----------|
| **Ondato** | Live agent + automated hybrid; banks/fintech |
| **KYCAID** | Agent-assisted; outsourced agent service available |
| **IDnow** | Germany-focused; BaFin Video Ident certified; used by N26 |
| **iDenfy** | Hybrid: Automated + agent escalation |
| **Jumio** | Automated-first; human review fallback |
| **Acuant** | Used by financial institutions |
| **VerifiNow** | Zoom/Teams integration for identity binding during video calls |

### Accuracy/Security Level

- **Highest assurance level**: Direct human verification + AI assistance
- Agent can detect social engineering, coercion, nervous behavior
- AI assists with document forensics and liveness
- Approximate 100% accuracy for willing, cooperative legitimate users
- Attack vectors: Deepfakes shown to camera, accomplice coaching, coercion

### UX Friction Level

- **Very High**: Scheduling, waiting for agent, 5–15 minute call
- Agent availability → queuing required (or async)
- User must have working camera + microphone
- Best reserved for high-value onboarding (large financial accounts, regulated access)

### Cost Considerations

- Highest-cost verification method: $10–$50+ per verification (agent time)
- Outsourced agent services: Lower cost but quality variable
- Automated-first with agent fallback reduces cost to $3–$8/verification

---