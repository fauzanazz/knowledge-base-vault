---
title: "Verification Regulatory Frameworks"
category: identity-and-verification
summary: "KYC/AML (FATF), PSD2/SCA (EU), eIDAS, and Indonesia OJK regulations — compliance requirements, eKYC standards, and fintech-specific rules."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification Regulatory Frameworks

> KYC/AML (FATF), PSD2/SCA (EU), eIDAS, and Indonesia OJK regulations — compliance requirements, eKYC standards, and fintech-specific rules.

## 7.1 KYC / AML (Global FATF Framework)

**FATF 40 Recommendations** form the global baseline:

```
CDD (Customer Due Diligence) Requirements:

SIMPLIFIED CDD (low-risk customers):
  └── Name + one ID document
  └── No ongoing monitoring required at high frequency

STANDARD CDD (medium-risk):
  └── Full name, DOB, address, occupation
  └── Government-issued photo ID
  └── Beneficial ownership (for businesses)
  └── Source of funds for large transactions
  └── Ongoing monitoring

ENHANCED DUE DILIGENCE - EDD (high-risk, PEPs):
  └── All standard CDD requirements
  └── Senior management approval to onboard
  └── Source of wealth documentation
  └── Enhanced ongoing monitoring
  └── Regular review (quarterly for high-risk)

KEY FATF PRINCIPLES:
  • Risk-Based Approach (RBA) — proportionate controls
  • Beneficial Ownership — identify ultimate owners
  • PEP Screening — enhanced checks for politically exposed persons
  • Sanctions Screening — OFAC, UN, EU consolidated list
  • Suspicious Activity Reports (SAR) — file with financial intelligence unit
  • Record Keeping — minimum 5 years
```

---

## 7.2 PSD2 / SCA (EU Payment Services Directive 2)

```
┌────────────────────────────────────────────────────────────┐
│              PSD2 STRONG CUSTOMER AUTHENTICATION           │
└────────────────────────────────────────────────────────────┘

SCA requires AT LEAST 2 of 3 INDEPENDENT factors:

  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
  │  KNOWLEDGE      │  │  POSSESSION     │  │  INHERENCE      │
  │  (Something     │  │  (Something     │  │  (Something     │
  │   you KNOW)     │  │   you HAVE)     │  │   you ARE)      │
  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤
  │ • PIN           │  │ • Hardware token│  │ • Fingerprint   │
  │ • Password      │  │ • OTP via phone │  │ • Face biometric│
  │ • Security Q&A  │  │ • Smart card    │  │ • Voice pattern │
  │ • Pattern       │  │ • Mobile app    │  │ • Iris scan     │
  └─────────────────┘  └─────────────────┘  └─────────────────┘

SCA EXEMPTIONS (allow frictionless flow):
  • Transaction < €30 AND cumulative < €100 → Low value
  • Trusted beneficiary (whitelisted payee) → Trust list
  • Corporate payment → B2B card exemption
  • TRA (Transaction Risk Analysis):
      - PSP fraud rate < 0.01% + transaction < €100 → exempt
      - PSP fraud rate < 0.06% + transaction < €250 → exempt
      - PSP fraud rate < 0.13% + transaction < €500 → exempt

IMPLEMENTATION — EMV 3DS2 Flow:
  Merchant checkout → 3DS Server → Directory Server → Issuer ACS
      │                                                      │
      └──────── sends 100+ data points (device, IP, ─────────┘
                 history, browser) for frictionless check
```

---

## 7.3 eIDAS (EU Electronic Identification and Authentication Services)

```
EIDAS TRUST LEVELS:

LOW:          Self-asserted identity, minimal verification
              → Suitable for low-risk services

SUBSTANTIAL:  Remote verification with government-issued ID
              → Requires document check + address confirmation
              → Comparable to standard KYC

HIGH:         In-person or equivalent; strong cryptographic binding
              → Government eID cards, Qualified Electronic Signatures
              → NFC chip verification, biometric comparison
              → Suitable for banking, tax, legal services

EIDAS 2.0 / EUDI WALLET (2025+):
  • EU-wide digital identity wallet (mandatory for member states)
  • Holds "Person Identification Data" (PID) — government-verified
  • Can be used for SCA in PSD3 flows
  • Enables selective disclosure: share only needed attributes
  • Replaces fragmented country-by-country eID systems

KEY CONCEPT — Qualified Electronic Signature (QES):
  • Legally equivalent to handwritten signature in all EU member states
  • Requires Qualified Trust Service Provider (QTSP)
  • Used for contracts, legal agreements in financial services
```

---

## 7.4 Indonesia OJK Regulations for Fintech

### Regulatory Bodies

| Authority | Domain |
|---|---|
| **OJK** (Otoritas Jasa Keuangan) | Non-payment fintech: P2P lending, crowdfunding, BNPL, digital banking, insurtech |
| **Bank Indonesia (BI)** | Payment services: e-wallets, e-money, payment gateways, remittance |
| **PPATK** | Financial Intelligence Unit — AML/CFT reporting |
| **KOMINFO** | Data protection, Electronic System Operators registration |

### Key Regulations

```
FINTECH VERIFICATION REGULATORY STACK (Indonesia):

┌─────────────────────────────────────────────────────┐
│  LAW NO. 8/2010 — Prevention of Money Laundering   │
│  Foundation of all AML obligations in Indonesia    │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  POJK 8/2023 — AML/CFT/WMD Prevention Program      │
│  • Mandatory CDD for all OJK-regulated entities     │
│  • Risk-based approach required                     │
│  • Enhanced Due Diligence (EDD) for high-risk       │
│  • Sanctions screening mandatory                    │
│  • SAR filing to PPATK                              │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  POJK 3/2024 — Financial Technology Innovation      │
│  • Regulatory sandbox framework                     │
│  • ITSK licensing requirements                      │
│  • AML/CFT mandatory for all registered ITSK        │
│  • Data center in Indonesia required                │
│  • Member of AFTECH (Indonesian Fintech Assoc.)     │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  BI REGULATION 10/2024 — AML/CFT for BI-regulated  │
│  • eKYC and biometric verification mandatory        │
│  • Third-party eKYC delegation permitted            │
│  • Data quality + integrity requirements            │
│  • Technical interoperability between institutions  │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  UU PDP (Personal Data Protection Law, 2022)        │
│  • Explicit consent required                        │
│  • Purpose limitation                               │
│  • Right to erasure (with AML exceptions)           │
│  • Cross-border transfer restrictions               │
│  • Data breach notification within 14 days          │
└─────────────────────────────────────────────────────┘
```

### eKYC Requirements in Indonesia

For financial services regulated by OJK/BI, eKYC must include:
1. **Identity data**: Full name, NIK (Nomor Induk Kependudukan — national ID), address, DOB, nationality
2. **Document**: KTP (Kartu Tanda Penduduk — national ID card) scan
3. **Selfie with KTP**: Photo of user holding their KTP (or liveness check)
4. **Phone verification**: OTP to mobile number
5. **DUKCAPIL integration**: Verification against Indonesia's civil registry database (for banks/licensed fintechs)

**DUKCAPIL** (Direktorat Jenderal Kependudukan dan Pencatatan Sipil) is Indonesia's civil registry. Licensed entities can query it to verify NIK + name + DOB matches national records — this is the gold standard for Indonesian eKYC.

---