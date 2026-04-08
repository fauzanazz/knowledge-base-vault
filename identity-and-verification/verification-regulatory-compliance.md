---
title: "Verification Regulatory & Compliance Framework"
category: identity-and-verification
summary: "Global and Indonesian verification regulations — KYC/AML, GDPR, PSD2/SCA, eIDAS, OJK regulations, UU PDP, and compliance requirements for fintech."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification Regulatory & Compliance Framework

> Global and Indonesian verification regulations — KYC/AML, GDPR, PSD2/SCA, eIDAS, OJK regulations, UU PDP, and compliance requirements for fintech.

### Global Regulations

| Regulation | Jurisdiction | Key Requirements |
|-----------|-------------|-----------------|
| **GDPR** | EU/EEA | Consent/legal basis; data minimization; right to erasure; DPA notification within 72h of breach; biometrics = Article 9 special category |
| **eIDAS 2.0** | EU | Digital identity wallet standard; mutual recognition of national eIDs across EU; age attributes in wallet |
| **4AMLD/6AMLD** | EU | CDD requirements; beneficial ownership; risk-based approach to KYC |
| **AML/BSA** | USA | FinCEN rules; CIP (Customer Identification Program); CTR/SAR filing; 5-year record retention |
| **CCPA/CPRA** | California | Biometrics = sensitive PI; opt-in consent; right to know/delete/correct |
| **BIPA** | Illinois | Explicit written consent for biometrics; $1,000–$5,000/violation; no statute of limitations |
| **COPPA** | USA | Verifiable parental consent for under-13 data collection |
| **PCI DSS v4** | Global (payments) | Multi-factor authentication required for cardholder data access |
| **NIST SP 800-63-3** | USA Federal | AAL1/2/3 standards; restricts SMS OTP to AAL1 |
| **Online Safety Act** | UK | Highly effective age assurance mandatory for pornography; children's access assessments |
| **POPIA** | South Africa | Biometric data as special information; consent required |
| **PDPA** | Thailand/Malaysia | Personal data protection; consent; purpose limitation |
| **PDP Law** | Indonesia | Law No. 27/2022; personal data protection; consent; data controller obligations |
| **PIPL** | China | Strict data localization; sensitive data requires specific processing conditions |
| **APPI** | Japan | Personal information handling; biometric data as sensitive personal information |

### Indonesia-Specific Regulatory Stack

| Law/Regulation | Authority | Scope |
|----------------|-----------|-------|
| **UU PDP (Law 27/2022)** | Komdigi (BRTI) | Comprehensive personal data protection law |
| **POJK 77/2016 + amendments** | OJK | Fintech lending KYC requirements |
| **POJK 12/2021** | OJK | E-KYC for financial services; face-based KYC permitted |
| **PBI 23/6/PBI/2021** | Bank Indonesia | E-money issuer license; KYC standards |
| **PP 82/2012** | Government | Electronic systems and transactions |
| **SIM registration policy** | Komdigi/Telcos | NIK-linked SIM card mandatory since 2018 |
| **Dukcapil API access** | Kemendagri | eKTP data verification; commercial entities can apply for API access |
| **PPATK regulations** | PPATK | STR/CTR filing; AML transaction monitoring |

### Key Compliance Principles for Verification Systems

1. **Data Minimization**: Only collect data strictly necessary for the verification purpose
2. **Purpose Limitation**: Verification data not reused for marketing, profiling, or other purposes without new consent
3. **Retention Limits**: Define and enforce retention periods; automatic deletion
4. **Transparency**: Privacy notice must clearly explain what verification data is collected, why, how long retained, and user rights
5. **Security**: Encryption at rest (AES-256) and in transit (TLS 1.3); access controls; audit logging
6. **Vendor Contracts**: DPA (Data Processing Agreement) with all third-party verification providers; right to audit
7. **Consent Records**: Maintain records of when, how, what consent was obtained
8. **Cross-border**: EU-US Data Privacy Framework; SCCs for international data transfers; Indonesia data localization considerations
9. **Breach Response**: 72-hour notification under GDPR; local law notification requirements vary
10. **Right to Appeal/Dispute**: Mechanism for users to challenge adverse verification decisions

---

## Quick Reference: Verification Method Comparison Matrix

| Method | Security Level | UX Friction | Cost/Check | Regulatory Complexity |
|--------|---------------|-------------|-----------|----------------------|
| SMS OTP | Medium | Low | $0.01–0.10 | Medium |
| Email verification | Low | Low–Medium | <$0.01 | Low |
| Selfie (passive liveness) | High | Very Low | $0.50–2.00 | High (biometric) |
| Gov ID KYC | Very High | Medium–High | $1–8 | Very High |
| On-device biometric | Very High | Very Low | $0 (on-device) | Very High |
| Device fingerprint | Medium | None | $0.01–0.05 | Medium |
| Social login | Low–Medium | Very Low | Free | Low–Medium |
| Address PoA | Medium | High | $0.50–2.00 | Medium |
| Age verification (facial estimate) | Medium | Low | $0.10–0.50 | High |
| CAPTCHA | Low | Low | Free–$0.001 | Low |
| Behavioral analytics | Medium | None | $0.05–0.20 | Medium |
| FIDO2/Passkeys (MFA) | Very High | Very Low | $0 (infra only) | Low |
| Bank statement PoA | High | High | $0.50–2.00 | Medium |
| Video call (live agent) | Very High | Very High | $10–50 | High |
| KBA (dynamic) | Medium | Medium–High | $0.20–1.00 | Medium |

---

## Recommended Verification Stacks by Use Case

### Dating App (e.g., Tinder-like)
```
Mandatory: Phone OTP + Email
Recommended: Photo selfie + passive liveness  
Optional: Gov ID + face match (incentivize with features)
Supplemental: Device fingerprint + behavioral analytics
```

### Ride-hailing Driver Onboarding (e.g., Grab/Gojek)
```
Mandatory: Gov ID + face match + liveness + license + vehicle docs
Continuous: Periodic selfie check (RTIC-style)
Background: Criminal record check + database cross-reference
Indonesian specifics: eKTP via Dukcapil + STNK + SIM
```

### Fintech / E-wallet (e.g., Dana/OVO style)
```
Basic tier: Phone OTP (SIM-linked NIK in Indonesia)
Standard tier: eKTP + selfie + Dukcapil API verification + liveness
Premium tier: All above + bank account linking + source of funds for large amounts
Ongoing: Behavioral analytics + transaction monitoring + AML screening
```

### Crypto Exchange (e.g., Binance-like)
```
Mandatory: Email + phone OTP + Gov ID + selfie + active liveness
MFA: TOTP (Google Authenticator) strongly encouraged; hardware key for high-value
Enhanced: Proof of address + source of funds for large withdrawals
Ongoing: Sanctions screening + behavioral analytics + device trust
```

### E-commerce Marketplace (e.g., Tokopedia)
```
Buyer: Phone OTP + email + behavioral analytics + device fingerprint
Seller: Above + NIK/KTP + bank account verification
Escrow: Platform holds funds; OJK-regulated settlement
High-value sellers: Full KYC + business registration
```

### B2B SaaS / Enterprise Platform
```
Signup: Work email + social login (Google/Microsoft) + phone OTP
Access: FIDO2/Passkeys + TOTP (for admin accounts, hardware key required)
Continuous: Device trust + behavioral analytics + IP allowlisting
Vendor KYC: Business registration + UBO verification + sanctions screening
```

---

*This knowledge base reflects verification technology and regulatory requirements as of early 2026. The field evolves rapidly; deepfake detection, passkey adoption, and national digital identity wallet programs are the most active development areas.*

---

## Summary

**What I did**: Conducted 12+ targeted web searches covering all 15 verification methods, platform-specific implementations, and regulatory frameworks. Synthesized information from technical documentation, provider websites, regulatory guidance (Ofcom, NIST, OJK), news articles, and platform help centers.

**What I found/accomplished**: Produced a comprehensive ~15,000-word knowledge base covering:

1. **All 15 verification methods** with technical depth including: protocols (TOTP RFC 6238, WebAuthn W3C Level 3, OAuth 2.0/OIDC, ISO 30107-3 liveness), provider comparisons, attack vectors with specific mitigations, regulatory implications, and cost data

2. **10 platform profiles** with verified implementation details:
   - Tinder (video selfie + ID + sex offender registry check)
   - Uber (RTIC using Microsoft Azure Face API + human review)
   - Airbnb (4-step ID + liveness + database checks)
   - Stripe (tiered KYC + Stripe Identity product)
   - Wise (auto-verify via bank transfer + full document KYC)
   - Binance (3-tier KYC + active liveness head-rotation)
   - Gojek/Grab (eKTP/Dukcapil + vehicle docs + ongoing RTIC)
   - Dana/OVO/GoPay (NIK-linked SIM registration + Dukcapil API KYC + OJK regulation)
   - Tokopedia (built-in OJK-regulated escrow system + KTP verification)

3. **Indonesia-specific deep-dive**: Dukcapil API, NIK-SIM linkage, POJK 12/2021, Rekber/escrow system mechanics

4. **Regulatory matrix**: GDPR, eIDAS 2.0, BIPA, COPPA, UK Online Safety Act, Indonesian PDP Law, OJK regulations

**No files created** (output provided directly as markdown).

**Issues encountered**: Rate limiting on search API required sequential searching for some topics; information on proprietary internal systems (Gojek's exact technical stack) was inferred from job postings and public documentation rather than official technical specs.