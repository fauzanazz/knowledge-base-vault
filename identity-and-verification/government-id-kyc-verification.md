---
title: "Government ID & KYC Verification"
category: identity-and-verification
summary: "Know Your Customer identity verification using government-issued documents — OCR extraction, document authenticity, face-to-ID matching, and regulatory requirements."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Government ID & KYC Verification

> Know Your Customer identity verification using government-issued documents — OCR extraction, document authenticity, face-to-ID matching, and regulatory requirements.

### How It Works Technically

**Document Capture**:
1. User photographs ID document (passport, national ID, driver's license)
2. OCR (Optical Character Recognition) extracts text:
   - MRZ (Machine-Readable Zone) parsing: Two/three-line code at bottom of passports — encodes name, DOB, nationality, document number with check digits
   - Barcode parsing: PDF417 (US driver's licenses), QR codes, Data Matrix
3. Document type identification from template library (5,000–14,000+ templates for global coverage)

**Document Authenticity Checks**:
- **Visual security features**: Holograms, UV-reactive ink, microprinting, guilloché patterns, color-shifting ink (OVI), laser engraving
- **NFC chip reading**: ePassports and newer eIDs contain RFID chip with digitally signed biographic data; NFC read verifies cryptographic signature and chip authenticity (ICAO 9303 standard)
- **Forensic ML analysis**: Trained models detect signs of:
  - Template alteration (font mismatch, inconsistent spacing)
  - Digital manipulation (clone stamps, Photoshop artifacts)
  - Screen recapture (moiré patterns)
  - Expired documents
  - Micro-printing inconsistencies

**Database Cross-Checks**:
- Name/DOB against credit bureaus (Equifax, Experian, TransUnion for US)
- Sanctions screening (OFAC, UN, EU lists)
- PEP (Politically Exposed Person) database
- Adverse media screening
- Death records
- Stolen/lost document registers (where accessible)

**KYC Tiers** (typical financial industry):
- **KYC Level 1 (Basic)**: Name, DOB, address self-declared + email/phone verified → low transaction limits
- **KYC Level 2 (Standard)**: Government ID document verified + selfie matching → standard limits
- **KYC Level 3 (Enhanced)**: All above + proof of address + source of funds + possibly video call → high limits, institutional access

### Common Providers/SDKs

| Provider | Documents | Countries | Specialty |
|----------|-----------|-----------|-----------|
| **Jumio** | 5,000+ types | 200+ | Enterprise KYC, Identity Graph fraud signals |
| **Onfido (Entrust)** | 14,000+ types | 195+ | Strong EU/APAC, low-friction |
| **Veriff** | 10,000+ types | 190+ | High automation rate, session-based |
| **Sumsub** | 14,000+ types | 220+ | Multi-jurisdiction compliance |
| **iDenfy** | 8,000+ types | 200+ | AML + liveness integrated, 4.9 rating |
| **Trulioo** | GlobalGateway | 195+ | Data-based verification, no document needed for some markets |
| **Socure** | US-focused | USA | AI identity graph, 2,700+ data sources |
| **Stripe Identity** | 50+ countries | 50+ | Integrated with Stripe platform |
| **Persona** | Custom workflows | Global | Highly configurable |
| **VIDA** | Indonesian focus | Indonesia | Mobile-first, cloud-based, Dukcapil integrated |
| **TrustDecision** | 14,000+ types | 240+ | Strong APAC coverage, AML |
| **Argos Identity** | 4,000+ types | Global | Africa, MENA, Philippines focus |

**Indonesia-specific**:
- **Dukcapil API**: Direct integration with Indonesia's Department of Population and Civil Registration for eKTP (electronic national ID) verification — real-time database lookup
- **BSrE (BSSN)**: Digital signature verification for government documents

### Accuracy/Security Level

- Auto-approval rates: 70–95% depending on document quality and user cooperation
- False acceptance rate target: <0.1% for financial KYC
- Human review fallback for uncertain cases (5–30% depending on provider)
- NFC-verified documents: >99.9% accuracy (cryptographic proof)

### UX Friction Level

- **Medium–High**: Requires document handling, good lighting, patience
- 60–90 seconds typical for full automated flow
- 15–30% abandonment rate at document step
- Mobile-optimized SDKs reduce friction vs. web upload

### Cost Considerations

- Per-check: $0.50–$5.00 for basic ID verification
- Full KYC (ID + liveness + database): $1.50–$8.00+
- AML screening: Additional $0.10–$0.50/check
- Volume enterprise deals can reduce to $0.30–$1.00/check

### Common Attack Vectors & Mitigations

| Attack | Mitigation |
|--------|------------|
| **Fake/forged documents** | Template analysis, security feature verification, NFC chip validation |
| **Identity theft (genuine doc, wrong person)** | Liveness detection + face matching to document photo |
| **Document substitution** | Binding document check to liveness session via encrypted session token |
| **Synthetic identity** | Cross-referencing multiple data sources; behavioral signals |
| **Account farming** | Identity Graph linking multiple verifications; device fingerprinting |

---