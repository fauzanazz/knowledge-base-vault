---
title: "Trust Scoring Systems & Risk-Based Verification"
category: identity-and-verification
summary: "Trust score architecture — risk factor matrices, dynamic score updates, trust decay models, and risk-tier based verification escalation."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Trust Scoring Systems & Risk-Based Verification

> Trust score architecture — risk factor matrices, dynamic score updates, trust decay models, and risk-tier based verification escalation.

## 2.1 Trust Score Architecture

A trust score is a dynamic, composite numerical value (typically 0–100 or 0–1000) representing the confidence that a user is who they claim to be and is acting in good faith.

```
┌────────────────────────────────────────────────────────────┐
│                TRUST SCORE COMPUTATION ENGINE              │
└────────────────────────────────────────────────────────────┘

Input Signals
     │
     ├─ Device signals        (weight: 0.15)
     │     └── fingerprint uniqueness, known device, rooted?
     ├─ Network signals       (weight: 0.10)
     │     └── IP reputation, VPN/proxy/Tor, geolocation
     ├─ Email signals         (weight: 0.10)
     │     └── age of domain, deliverability, breach exposure
     ├─ Phone signals         (weight: 0.15)
     │     └── carrier-verified, SIM swap recency, ported?
     ├─ Document signals      (weight: 0.25)
     │     └── authenticity score, MRZ match, tamper detect
     ├─ Biometric signals     (weight: 0.20)
     │     └── face match confidence, liveness score
     └─ Behavioral signals    (weight: 0.05)
           └── typing cadence, session patterns, time-on-task
           
                    │
                    ▼
         ┌──────────────────────┐
         │  Weighted Aggregator │
         │  (ML model or rules) │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │    Composite Score   │
         │      0 – 1000        │
         └──────────┬───────────┘
                    │
         ┌──────────┼──────────┐
         ▼          ▼          ▼
    0–300       300–700     700–1000
   HIGH RISK   MED RISK    LOW RISK
   Block/EDD   Manual/Step  Auto-approve
               Up-Verify
```

## 2.2 Risk Factor Matrix

| Factor | Low Risk | Medium Risk | High Risk |
|---|---|---|---|
| Geography | FATF member state | Non-FATF | FATF blacklist/greylist |
| Transaction volume | < $1K/month | $1K–$10K | > $10K |
| Account age | > 6 months | 1–6 months | < 30 days |
| Device history | Known device | New device | Emulated/rooted |
| IP | Residential, home country | Foreign, datacenter | Tor, known fraud IP |
| Document | Fresh government ID | Slightly expired | Expired, tamper signs |
| PEP/Sanctions | Not listed | Related party | Direct hit |

## 2.3 Dynamic Score Updates

Trust scores must update continuously, not just at onboarding:

```
ONBOARDING SCORE: 720 (Low Risk)
     │
     │─── 3 months later: Normal activity → Score: 730
     │
     │─── Month 4: SIM swap detected on phone → Score: 590 → Require re-verify
     │
     │─── Month 4: Re-verified with face liveness → Score: 680
     │
     │─── Month 6: Unusual withdrawal pattern → Score: 520 → Manual review flag
     │
     │─── Month 6: Cleared by compliance officer → Score: 680
```

### Score Decay Model
```
effective_score = base_score × e^(-λt)

Where:
  λ = decay constant (e.g., 0.002 per day for financial platforms)
  t = days since last verification event

After 365 days without activity: score decays ~50%
This forces periodic re-verification for dormant accounts.
```

## 2.4 Implementing Risk Tiers

```javascript
const RISK_TIERS = {
  LOW:    { minScore: 700, label: 'GREEN',  action: 'auto_approve'    },
  MEDIUM: { minScore: 400, label: 'YELLOW', action: 'step_up_verify'  },
  HIGH:   { minScore: 200, label: 'ORANGE', action: 'manual_review'   },
  BLOCK:  { minScore: 0,   label: 'RED',    action: 'block_and_alert' },
};

function classifyRisk(trustScore) {
  if (trustScore >= RISK_TIERS.LOW.minScore)    return RISK_TIERS.LOW;
  if (trustScore >= RISK_TIERS.MEDIUM.minScore) return RISK_TIERS.MEDIUM;
  if (trustScore >= RISK_TIERS.HIGH.minScore)   return RISK_TIERS.HIGH;
  return RISK_TIERS.BLOCK;
}

// Risk actions trigger verification tier selection:
const TIER_MAP = {
  auto_approve:   'TIER_0_1',  // IP + Phone OTP only
  step_up_verify: 'TIER_2',    // Document verification required
  manual_review:  'TIER_3_4',  // Biometrics + human review
  block_and_alert:'BLOCKED',
};
```

---