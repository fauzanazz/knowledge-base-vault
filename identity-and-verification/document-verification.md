---
title: "Document Verification (Beyond ID)"
category: identity-and-verification
summary: "Non-ID document verification — bank statements, utility bills, employment letters, academic certificates, and automated document classification."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Document Verification (Beyond ID)

> Non-ID document verification — bank statements, utility bills, employment letters, academic certificates, and automated document classification.

### Types of Non-ID Documents Verified

**Bank Statements**:
- Most common PoA document
- Verification checks:
  - Bank logo/brand authenticity
  - Template consistency with known bank formats
  - OCR: Account name, account number (partial), address, issue date, bank name
  - Balance data (if income verification needed)
  - Statement period validation
  - Digital manipulation detection (font inconsistency, pixel analysis)
- **Bank data alternative**: Plaid, TrueLayer, Nordigen pull live statement data via Open Banking API — eliminates document fraud entirely

**Utility Bills**:
- Gas, electricity, water, landline (NOT mobile/cell)
- Checks: Provider logo, billing period, consumption data format, address formatting

**Pay Stubs / Employment Verification**:
- Income verification for lending
- Employer name, salary figure, pay frequency
- Cross-reference with employer databases (The Work Number, Equifax workforce)
- **Alternative**: Open Banking salary inflow analysis

**Tax Documents**:
- W-2, 1099 (USA); SA302 (UK); equivalent in other countries
- Used for income verification
- HMRC PAYE data integration (UK Gov.uk)
- IRS Income Verification Express Service (IVES, USA)

**Insurance Documents**:
- Proof of insurance for gig economy onboarding (Uber, Grab drivers)
- Policy number, coverage dates, insured vehicle/person details

**Vehicle Registration (STNK in Indonesia)**:
- Required for Gojek/Grab driver registration
- OCR of plate number, owner name, vehicle make/model/year
- Cross-reference with Samsat (Indonesia vehicle registration database) or equivalent

**Business Registration Documents** (KYB — Know Your Business):
- Articles of incorporation, Certificate of Good Standing
- Ultimate Beneficial Owner (UBO) documentation
- Issued by: Companies House (UK), SEC (USA), AHU (Indonesia)

**Academic Credentials**:
- Degree certificates for verification (LinkedIn Education Verification)
- National student ID verification

### Technical Verification Process

1. **Capture**: User uploads PDF, JPEG, PNG, or uses camera SDK
2. **Classification**: ML model identifies document type and issuer
3. **OCR Extraction**: Structured data extraction using computer vision + NLP
4. **Field Validation**: 
   - Date parsing (within acceptable window)
   - Name normalization (fuzzy match with account data)
   - Address component matching
5. **Authenticity Signals**:
   - Metadata analysis (PDF creation date, software used)
   - Pixel-level forensics for signs of editing
   - Template matching against known bank/utility formats
6. **Cross-reference**: Match data against account-provided information

### Leading Providers

| Provider | Specialty |
|----------|-----------|
| **Veriff PoA** | Full suite: bank statements, utility bills, with address matching |
| **IDWise** | 15+ document categories; Arabic/Cyrillic support |
| **Argos Identity** | Medical invoices, rental agreements, insurance |
| **Shufti Pro** | 240+ countries, 99.77% accuracy |
| **Plaid/Nordigen** | Live bank data extraction (eliminates document) |
| **The Work Number (Equifax)** | Employment/income verification |
| **Persona** | Highly configurable document verification flows |

---