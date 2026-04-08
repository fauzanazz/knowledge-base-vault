---
title: "Behavioral Analytics & Risk Scoring"
category: identity-and-verification
summary: "User behavior analysis for fraud detection — typing patterns, navigation behavior, velocity checks, IP reputation, and ML-based risk scoring."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Behavioral Analytics & Risk Scoring

> User behavior analysis for fraud detection — typing patterns, navigation behavior, velocity checks, IP reputation, and ML-based risk scoring.

### How It Works Technically

**Data Collection Layer**:
- Keystroke dynamics: Inter-key timing, key hold duration, typing rhythm
- Mouse/touch behavior: Movement trajectory, click precision, scroll patterns, pressure (mobile)
- Form interaction: Field entry order, time-in-field, copy-paste detection, hesitation patterns
- Navigation: Page path, time-on-page, back/forward navigation, tab switching
- Session metadata: Time of day, session duration, referrer, device/OS

**Behavioral Baseline**:
- Per-user profile built over time (typically 2–4 weeks)
- Statistical model of "normal" behavior for this user
- Continuous update with new sessions

**Anomaly Detection**:
- Real-time comparison of current session vs. baseline
- Deviation triggers risk score increase
- Example red flags:
  - Login from impossible geographic distance in short time
  - Sudden change in typing speed (another person at keyboard)
  - Unusual transaction size or frequency
  - Access outside normal hours
  - Rapid account detail changes

**Risk Scoring Model**:
```
Risk Score = f(
  device_trust_score,       # New/known device
  location_score,           # Consistent with history
  behavioral_score,         # Matches baseline patterns  
  velocity_score,           # Actions per time unit
  network_score,            # IP reputation, VPN use
  account_score,            # Account age, history
  transaction_score         # Amount, recipient, frequency
)
```

- Outputs: 0–100 or 0.0–1.0 risk score
- Thresholds: Low (<30) = pass; Medium (30–70) = step-up auth; High (>70) = block/review

**Machine Learning Approaches**:
- Supervised learning (labeled fraud vs. legitimate) for fraud classification
- Unsupervised anomaly detection (isolation forest, autoencoders) for novel patterns
- Federated learning: Models trained across customer consortium without raw data sharing
- Real-time scoring: Apache Kafka + Flink for stream processing; <100ms latency targets

### Common Providers

| Provider | Specialty |
|----------|-----------|
| **NeuroID** | Top-of-funnel form behavioral analytics; 90% fraud detection at <1% false positive |
| **Sardine** | Fintech-focused; device + behavioral + velocity integrated |
| **Sumsub Behavioral Biometrics** | KYC-integrated behavioral monitoring |
| **SEON** | Email/phone/IP intelligence + behavioral |
| **Kount (Equifax)** | E-commerce fraud, AI decisioning |
| **PayPal's internal ML** | Hundreds of signals; device + email + behavioral |
| **BioCatch** | Banking-focused behavioral biometrics |
| **ThreatMetrix (LexisNexis)** | 5B+ device network; real-time risk scoring |

### Use Cases

- **Account opening fraud**: Detect bots and fraudulent applications at onboarding
- **Account takeover (ATO)**: Detect when a legitimate account is being used by unauthorized person
- **Payment fraud**: Anomalous transaction patterns
- **Synthetic identity fraud**: PII entered hesitantly (copied from notes) vs. naturally typed

**PayPal's Implementation Example**:
- Hundreds of signals: Device info, email checks, identity scores, session data, enrollment details
- Behavioral signals include scroll behavior, typing speed, device tilt
- Machine learning models updated continuously
- Risk scores trigger additional verification steps

---