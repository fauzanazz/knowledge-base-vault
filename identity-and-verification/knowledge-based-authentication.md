---
title: "Knowledge-Based Authentication (KBA)"
category: identity-and-verification
summary: "Knowledge-based verification — static vs dynamic KBA, out-of-wallet questions, credit bureau integration, and limitations/alternatives."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Knowledge-Based Authentication (KBA)

> Knowledge-based verification — static vs dynamic KBA, out-of-wallet questions, credit bureau integration, and limitations/alternatives.

### How It Works Technically

**Static KBA** (pre-set security questions):
- At account setup: User selects from question list and provides answers
- Answers stored hashed (bcrypt/PBKDF2) in database
- At verification: User answers subset of questions
- Common questions: mother's maiden name, first pet name, childhood street
- **Major weakness**: Answers often publicly available on social media, data breaches, genealogy sites

**Dynamic KBA** (out-of-wallet questions):
- Questions generated in real-time from personal data aggregated from:
  - Credit bureau records (previous addresses, loan amounts, vehicle registrations)
  - Public records (court records, property records)
  - Utility/phone records
  - Voter registration
- Questions about specific details: "Which of these was a monthly payment for your auto loan in 2019?"
- Multiple choice format: Correct answer + plausible distractor options
- One-time use; questions change each session
- Data sourced through: LexisNexis, Equifax, Experian, Transunion, Neustar

**Technical Flow (Dynamic KBA)**:
```
1. User provides PII (name, DOB, SSN/last 4, address)
2. KBA provider queries data broker APIs in real-time
3. Generate 3–5 multiple-choice questions with ~4 options each
4. User answers within time limit (typically 2–5 minutes)
5. Score: Correct answers required (e.g., 3 of 4 to pass)
6. Session bound to IP + device; limited retries (typically 3)
```

**Identity Graph for Dynamic KBA**:
- Network of linked data: Same person likely has connected address history, vehicle registrations, past employers
- ML model selects most discriminative questions (hard for fraudster to answer, easy for legitimate user)

### Common Providers

| Provider | Type | Notes |
|----------|------|-------|
| **Experian CrossCore** | Dynamic KBA | Credit bureau data, USA |
| **LexisNexis InstantID** | Dynamic KBA | Public records + credit data |
| **Equifax Precise ID** | Dynamic KBA | Credit bureau source |
| **TransUnion TruValidate** | Dynamic KBA | Risk-based authentication |
| **Socure** | Dynamic KBA replacement | AI identity graph (KBA scoring) |
| **Auth0** | Static KBA | Built-in security questions |

### Security Analysis

| Threat | Static KBA | Dynamic KBA |
|--------|-----------|-------------|
| **Social engineering** | Very vulnerable (friends/family know answers) | Moderate (harder to research) |
| **Data breach exposure** | High (security Q&A in breach dumps) | Low (questions generated fresh) |
| **AI/search-assisted attack** | High (social media mining) | Moderate (requires data broker access) |
| **Synthetic identity** | Low (fake person may not have credit history) | Low (thin file = no questions possible) |
| **Authorized user attack** | High | High (family member with shared history) |

**NIST SP 800-63-3**: Recommends against using KBA for recovery in high-assurance contexts; acceptable only for AAL1

### UX Friction Level

- **Static KBA**: Low for setup; Medium for recall (users forget answers)
- **Dynamic KBA**: High — questions can be confusing or unanswerable for legitimate users with limited credit history (immigrants, young adults, thin-file consumers)
- "Locked out" rate: 10–20% for dynamic KBA for thin-file users

### Regulatory Considerations

- **GLBA (USA)**: Financial institutions must use reasonable authentication; KBA alone insufficient for high-risk
- **PCI DSS**: KBA not acceptable for payment card data access
- **GDPR**: Dynamic KBA queries credit/public records; requires legal basis
- **CCPA**: Personal data used must be disclosed in privacy policy

---