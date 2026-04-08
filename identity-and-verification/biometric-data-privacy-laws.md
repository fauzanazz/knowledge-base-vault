---
title: "Biometric Data Privacy Laws"
category: identity-and-verification
summary: "Biometric data regulations worldwide — BIPA (Illinois), GDPR Article 9, Indonesia UU PDP, and compliance requirements for biometric verification systems."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Biometric Data Privacy Laws

> Biometric data regulations worldwide — BIPA (Illinois), GDPR Article 9, Indonesia UU PDP, and compliance requirements for biometric verification systems.

### 11.1 Illinois Biometric Information Privacy Act (BIPA)

**Enacted**: 2008 | **Jurisdiction**: Illinois, USA | **Status**: The world's most litigated biometric privacy law

#### Covered Data
- Biometric **identifiers**: retina/iris scans, fingerprints, voiceprints, hand/face geometry scans
- Biometric **information**: any info derived from biometric identifiers used for identification

#### Core Requirements (Section 15)

| Obligation | Requirement |
|---|---|
| **Written Policy** | Public-facing written retention and destruction schedule |
| **Informed Written Consent** | Before any collection — purpose, duration, and method must be disclosed |
| **No Profit** | Absolute prohibition on selling, leasing, or trading biometric data |
| **No Disclosure** | No third-party sharing without consent (except transaction completion, legal requirement) |
| **Security Standards** | "Reasonable standard of care" — at least as protective as other sensitive data |
| **Destruction** | Delete when purpose fulfilled OR within 3 years of last interaction (whichever first) |

#### Penalties (as amended by SB 2979, August 2024)
```
Per-violation:
├── Negligent violation: $1,000 per person (not per scan — 2024 amendment)
├── Intentional/reckless: $5,000 per person
└── Plus attorneys' fees (private right of action — no regulatory agency needed)

Notable settlements:
├── Facebook / Meta: $650 million (2021)
├── BNSF Railway: $228 million (2023)
├── Google: $100 million (2022)
└── TikTok: $92 million (2021)
```

#### 2024 Amendment (SB 2979) Key Changes
- **Accrual rule**: Single violation per individual (not per scan), reversing the catastrophic *Cothron v. White Castle* ruling that made per-scan liability existentially threatening
- **Electronic consent**: Digital signatures (checkbox, e-signature) explicitly satisfy "written release" requirement

#### BIPA Compliance Checklist for Biometric Verification Systems
```
□ Published biometric data policy (retention schedule + destruction guidelines)
□ Separate written notice before first biometric collection
□ Affirmative written/electronic consent (not buried in ToS)
□ Data processing agreement (DPA) with biometric vendor
□ Vendor prohibited from selling/profiting from biometric data
□ Deletion process: automated destruction within 3 years / on purpose completion
□ Security controls: encryption at rest + in transit, access controls, audit logs
□ Illinois-specific legal review if any Illinois residents are users
```

---

### 11.2 GDPR Article 9 — Special Categories of Personal Data

**Enacted**: May 2018 | **Jurisdiction**: EU + EEA + extraterritorial (processing EU residents' data) | **Enforcer**: National Data Protection Authorities (DPAs)

#### Biometric Data Classification
Under **Article 4(14)**, biometric data means:
> *"Personal data resulting from specific technical processing relating to the physical, physiological or behavioural characteristics of a natural person, which allow or confirm the unique identification of that natural person, such as facial images or dactyloscopic data."*

**Key distinction from Indonesian law**: GDPR requires **technical processing for identification** — a raw photograph is NOT biometric data under GDPR unless processed through facial recognition algorithms. The processing transforms it into biometric data.

#### Article 9 Processing Framework

```
Article 9(1): Default Prohibition
"Processing of biometric data for the purpose of uniquely 
identifying a natural person is PROHIBITED"

Article 9(2): Exceptions (must satisfy at least one):
├── (a) Explicit consent of data subject
├── (b) Necessary for employment/social security obligations
├── (c) Vital interests when subject cannot consent
├── (d) Legitimate activities of not-for-profit bodies
├── (e) Data manifestly made public by data subject
├── (f) Legal claims establishment/defense
├── (g) Substantial public interest (must have EU/Member State law basis)
├── (h) Health/medical purposes (professional secrecy)
├── (i) Public health (epidemics, safety)
└── (j) Archiving/scientific/historical/statistical research
```

#### GDPR Requirements for Biometric Verification Systems

| Requirement | Implementation |
|---|---|
| **Lawful basis** | Typically 9(2)(a) explicit consent for KYC onboarding |
| **Data minimization** | Process only what's necessary; do not retain raw images if hash/template suffices |
| **Purpose limitation** | Biometric data collected for identity verification cannot be used for marketing |
| **Storage limitation** | Delete biometric data when verification purpose is complete |
| **Data subject rights** | Right to access, rectification, erasure (RTBF), portability, objection |
| **DPIA (Art 35)** | Mandatory Data Protection Impact Assessment for large-scale biometric processing |
| **Records of processing** | Maintain Art 30 processing records |
| **DPO** | Required if large-scale biometric processing is a core activity |
| **Cross-border transfers** | Standard Contractual Clauses (SCCs) or adequacy decision required |

#### Key GDPR Enforcement Cases (Biometric)
- **Clearview AI**: €20M+ fines across multiple EU member states (unlawful biometric scraping)
- **Worldline**: €5.1M (CNIL) — face recognition without proper legal basis
- **Amazon Rekognition police use**: Multiple DPA investigations across EU

---

### 11.3 Indonesia — UU PDP (Undang-Undang Perlindungan Data Pribadi)

**Law No. 27 of 2022** | **Fully Effective**: October 17, 2024 | **Model**: GDPR-inspired

#### Biometric Data Classification

Under **Article 4(2)(b) UU PDP**, biometric data is classified as **Specific Personal Data** (*Data Pribadi yang Bersifat Spesifik*):
> *"Data related to physical, physiological, or behavioral characteristics of an individual that enable unique identification, such as facial images or dactyloscopic data. This includes but is not limited to fingerprints, retina scans, and DNA samples."*

**Critical difference from GDPR**: Indonesia's definition is **broader** — even a regular photograph can qualify as biometric data if used for identification purposes (e.g., on a KTP / national ID card). No "specific technical processing" threshold required.

#### Processing Requirements

| Requirement | UU PDP Provision | Practical Implication |
|---|---|---|
| **Explicit Consent** | Article 20(2) | Mandatory before processing specific personal data |
| **Transparency** | Article 21 | Disclose: purpose, duration, data types, rights, legal basis |
| **Data Subject Rights** | Articles 5–16 | Access, correction, deletion, portability, objection rights |
| **Cross-Border Transfer** | Article 56 | Only to countries with equivalent protection or with safeguards |
| **Penalties** | Article 67 | Administrative: up to IDR 2% of annual revenue; Criminal: up to 6 years imprisonment |

#### UU PDP vs. GDPR vs. BIPA Comparison

| Dimension | BIPA (Illinois) | GDPR (EU) | UU PDP (Indonesia) |
|---|---|---|---|
| **Scope** | Private entities in Illinois | All processing of EU residents' data | All processing of Indonesian residents' data |
| **Biometric definition** | Functional (scan/measurement) | Requires technical processing for ID | Broad — includes raw photo for ID use |
| **Legal basis options** | Consent + limited exceptions | Multiple bases (Art 9(2)) | Consent + contractual + legal obligation + vital interest + public interest |
| **Private right of action** | **Yes** — individuals can sue | No (DPA enforcement) | No (regulatory enforcement) |
| **Penalties** | $1,000–$5,000/violation | 4% global annual turnover or €20M | 2% annual revenue; 6 years imprisonment |
| **Enforcement maturity** | Very high (massive litigation) | High (active DPA enforcement) | Developing (Lembaga PDP not yet operational as of 2026) |
| **Cross-border transfer** | Not addressed | Adequacy/SCCs required | Adequacy or equivalent safeguards |

#### OJK AI Governance (Banking Sector — April 2025)
Indonesia's financial regulator OJK published AI Governance guidelines for banking (April 29, 2025), specifically addressing:
- Biometric authentication in digital banking
- AI-based fraud detection compliance
- Customer consent for biometric data processing in fintech

---

### 11.4 Global Biometric Privacy Law Summary

| Jurisdiction | Law | Biometric Category | Consent Basis | Penalty |
|---|---|---|---|---|
| **Illinois, USA** | BIPA (2008, amended 2024) | Special — private right of action | Written, explicit, pre-collection | $1K–$5K per person |
| **Texas, USA** | CUBI Act (2009) | Special | Written consent | AG enforcement only |
| **Washington, USA** | My Health My Data (2023) | Special | Affirmative, separate | AG enforcement |
| **California, USA** | CPRA/CCPA (2020/2023) | Sensitive — opt-out right | Not required, but opt-out | $2,500–$7,500 per violation |
| **EU/EEA** | GDPR Art 9 (2018) | Special — default prohibited | Explicit consent or Art 9(2) exception | 4% global revenue or €20M |
| **Indonesia** | UU PDP (2022) | Specific — strict controls | Explicit consent | 2% revenue; imprisonment |
| **India** | DPDP Act (2023) | Sensitive — enhanced protection | Explicit consent | ₹250 crore (~$30M) |
| **Brazil** | LGPD Art 11 (2020) | Sensitive | Explicit consent or legal basis | 2% revenue, up to R$50M |
| **China** | PIPL (2021) | Sensitive | Separate explicit consent | ¥50M or 5% revenue |

---

### 11.5 Privacy-By-Design Principles for Biometric Systems

```
Biometric System Privacy Architecture:
├── Data Minimization:
│   ├── Store mathematical templates, NOT raw images (where possible)
│   ├── Irreversible one-way hash of biometric embeddings
│   └── Separate biometric template from identity data (unlinkability)
├── Purpose Limitation:
│   ├── Biometric data isolated in purpose-specific database
│   └── Technical controls prevent repurposing
├── Security Architecture:
│   ├── Biometric templates encrypted at rest (AES-256)
│   ├── Key management: HSM or cloud KMS with envelope encryption
│   ├── Template in transit: TLS 1.3 + additional payload encryption
│   └── Access control: Least privilege, audit logging for all access
├── Retention Management:
│   ├── Automated deletion on session expiry (liveness check only)
│   ├── Template lifecycle management system
│   └── Deletion attestation for regulatory compliance
├── Consent Management:
│   ├── Granular, revocable consent per purpose
│   ├── Consent audit trail immutably logged
│   └── Self-service consent withdrawal mechanism
└── Cross-Border Governance:
    ├── Data residency controls (region-specific storage)
    ├── Standard Contractual Clauses with processors
    └── Transfer Impact Assessments where required
```

---

## Appendix: Technology Readiness Overview (2026)

```
Liveness Detection Threat/Defense Matrix:

Attack Type          │ Basic PAD │ ISO 30107-3 L2 │ Injection Detection │ Cryptographic Challenge
─────────────────────┼───────────┼────────────────┼─────────────────────┼────────────────────────
Printed photo        │    ✅     │       ✅       │         N/A         │          N/A
Screen replay        │    ✅     │       ✅       │         ✅          │          ✅
2D mask             │    ✅     │       ✅       │         N/A         │          N/A
Rigid 3D mask        │    ⚠️     │       ✅       │         N/A         │          N/A
Silicone 3D mask     │    ❌     │       ✅       │         N/A         │          N/A
Virtual camera       │    ❌     │       ❌       │         ✅          │          ✅
Deepfake injection   │    ❌     │       ❌       │         ✅          │          ✅
Replay injection     │    ❌     │       ❌       │         ✅          │          ✅
Real-time deepfake   │    ❌     │       ❌       │    ⚠️ Emerging      │          ✅

✅ = Effective defense  ⚠️ = Partial defense  ❌ = Not addressed
```

---

*Document compiled April 2026. Standards, vendor certifications, and legal requirements evolve rapidly. Verify current certification status with iBeta (ibeta.com), ISO (iso.org), and relevant regulatory authorities.*

---

## Summary

**What I did**: Conducted 9 targeted web searches across all 11 required topic areas, extracting technical details from primary sources including ISO standards pages, vendor documentation (FaceTec, iProov, Jumio, Veriff), academic papers (arxiv, IEEE, MDPI), legal resources (BIPA, GDPR, UU PDP), and industry reports.

**What I found and produced**: A comprehensive 5,000+ word technical markdown document covering:

1. **Liveness detection types** — Active (blink/gesture challenges), Passive (texture/frequency/depth), and Hybrid cascade architectures with detailed signal analysis
2. **2D vs. 3D technical differences** — Hardware technologies (structured light, ToF, stereo), software 3D estimation (FaceTec ZoOm perspective distortion method), and a comprehensive comparison table
3. **ISO 30107 PAD standard** — All parts explained, attack species taxonomy, APCER/BPCER/IAPAR metrics, iBeta Level 1/2 requirements, and the new CEN/TS 18099 injection attack standard
4. **Vendor implementations** — Deep technical dives into FaceTec ZoOm (3D FaceMap pipeline), iProov Flashmark™ (cryptographic challenge-response), Jumio multi-layer defense, and Veriff Station flow — with certifications and performance benchmarks
5. **Deepfake detection** — GAN artifact signatures, FFT/DCT frequency analysis, temporal inconsistency detection (TALL, LFGDIN, pixel-wise temporal frequency), and diffusion model detection challenges
6. **Injection attack prevention** — All attack vectors documented, client-side hardening (App Attest, Play Integrity), server-side frame forensics, and cryptographic defenses
7. **Face matching algorithms** — 1:1 verification and 1:N identification pipelines, embedding architectures (ArcFace, CosFace, AdaFace), threshold tuning, and NIST FRTE benchmark data
8. **Accuracy metrics** — FAR/FRR/EER/TAR/APCER/BPCER definitions, benchmark comparison table across vendors, dataset descriptions
9. **Mobile vs. web** — Architecture comparison, iOS/Android-specific APIs, browser limitations (no hardware attestation, no depth data), compatibility matrix, performance benchmarks
10. **Edge cases** — Identical twins (FRS vulnerability, iris mitigation), aging (template drift rates), cosmetic surgery (procedure-by-procedure impact), lighting conditions (failure modes and mitigations), plus table of additional edge cases
11. **Privacy laws** — BIPA (full requirements, 2024 amendment, penalty schedule), GDPR Art 9 (classification, Art 9(2) exceptions, enforcement cases), Indonesia UU PDP (definition differences, OJK banking guidelines), global law comparison table, and privacy-by-design architecture

**Files created**: None (output delivered as inline markdown per task specification)

**Issues encountered**: None significant — all data sources were accessible and current (2024–2026). Some vendor FAR/FRR figures are not publicly disclosed; noted in the benchmark table with flags.