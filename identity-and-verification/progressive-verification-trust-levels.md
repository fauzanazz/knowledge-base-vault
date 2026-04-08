---
title: "Progressive Verification & Trust Levels"
category: identity-and-verification
summary: "Risk-based progressive verification — trust level tiers, escalation triggers, UX patterns, and dynamic verification requirements based on transaction risk."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Progressive Verification & Trust Levels

> Risk-based progressive verification — trust level tiers, escalation triggers, UX patterns, and dynamic verification requirements based on transaction risk.

## 8.1 The Trust Level Architecture

```
┌────────────────────────────────────────────────────────────┐
│            PROGRESSIVE TRUST LEVEL SYSTEM                  │
└────────────────────────────────────────────────────────────┘

LEVEL 0 — ANONYMOUS
┌────────────────────────────────────────────────────┐
│ Who: Unregistered visitors                         │
│ Verification: None                                 │
│ Capabilities: Browse only, no transactions         │
│ Risk: N/A                                          │
└────────────────────────────────────────────────────┘
                         │ user registers
                         ▼
LEVEL 1 — EMAIL/PHONE VERIFIED
┌────────────────────────────────────────────────────┐
│ Who: Registered users with basic contact verify    │
│ Verification: Email OTP + Phone OTP                │
│ Capabilities:                                      │
│   • View listings                                  │
│   • Send/receive messages                          │
│   • Create transactions up to Rp 1,000,000        │
│ Transaction limit: Rp 1,000,000/transaction        │
│ Monthly limit: Rp 5,000,000                       │
└────────────────────────────────────────────────────┘
                         │ exceeds limit OR risk trigger
                         ▼
LEVEL 2 — KTP VERIFIED (Standard KYC)
┌────────────────────────────────────────────────────┐
│ Who: Users with verified national ID               │
│ Verification: KTP upload + selfie + liveness       │
│ Optional: DUKCAPIL NIK check                       │
│ Capabilities:                                      │
│   • All Level 1 features                           │
│   • Transactions up to Rp 50,000,000              │
│   • Can be a seller                                │
│   • Dispute resolution eligible                   │
│ Transaction limit: Rp 50,000,000/transaction       │
│ Monthly limit: Rp 100,000,000                     │
└────────────────────────────────────────────────────┘
                         │ high-value or high-risk
                         ▼
LEVEL 3 — ENHANCED KYC
┌────────────────────────────────────────────────────┐
│ Who: Power users, high-volume merchants            │
│ Verification: + Address proof + NPWP (tax ID)      │
│ + Video KYC or in-person verification              │
│ Capabilities:                                      │
│   • All Level 2 features                           │
│   • Transactions up to Rp 500,000,000             │
│   • Premium merchant status                        │
│   • Priority dispute resolution                   │
│ Transaction limit: Rp 500,000,000/transaction      │
│ Monthly limit: Unlimited                           │
└────────────────────────────────────────────────────┘
                         │ business account
                         ▼
LEVEL 4 — BUSINESS VERIFIED (KYB)
┌────────────────────────────────────────────────────┐
│ Who: Companies, professional merchants             │
│ Verification: Business registration (NIB/SIUP)    │
│ + NPWP perusahaan + director KYC                  │
│ + Bank account verification                        │
│ Capabilities: Enterprise features, API access      │
└────────────────────────────────────────────────────┘
```

## 8.2 Trust Level Escalation Triggers

```javascript
const ESCALATION_TRIGGERS = [
  // Amount-based triggers
  { condition: 'transaction_amount > 1_000_000',      escalate_to: 2 },
  { condition: 'monthly_cumulative > 5_000_000',      escalate_to: 2 },
  { condition: 'transaction_amount > 50_000_000',     escalate_to: 3 },
  
  // Risk-based triggers
  { condition: 'risk_score > 600',                    escalate_to: 2 },
  { condition: 'risk_score > 750',                    escalate_to: 3 },
  { condition: 'fraud_signal_detected',               escalate_to: 3 },
  
  // Behavioral triggers
  { condition: 'country_mismatch',                    escalate_to: 2 },
  { condition: 'device_change',                       escalate_to: 2 },
  { condition: 'pep_name_fuzzy_match',               escalate_to: 3 },
  
  // Regulatory triggers
  { condition: 'user_is_seller',                      escalate_to: 2 },
  { condition: 'user_requests_withdrawal',            escalate_to: 2 },
];
```

## 8.3 UX Pattern for Progressive Escalation

The key is to escalate **in-context**, not interrupt the user cold:

```
User wants to transact Rp 2,000,000 but is Level 1 (limit: Rp 1,000,000):

┌────────────────────────────────────────────────┐
│  💡 Almost there!                              │
│                                                │
│  To complete this Rp 2,000,000 transaction,   │
│  we need to verify your identity.              │
│                                                │
│  ✓ Takes only 2 minutes                        │
│  ✓ You'll unlock higher limits                 │
│  ✓ Your account becomes more trusted           │
│                                                │
│  [Verify Now]    [Do It Later]                │
│                                                │
│  "Do It Later" limits transaction to Rp 500K  │
└────────────────────────────────────────────────┘

→ If "Verify Now": inline verification flow (no full page reload)
→ If "Do It Later": allow partial transaction, flag for verification
```

---