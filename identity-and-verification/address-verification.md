---
title: "Address Verification"
category: identity-and-verification
summary: "Proof of address verification — utility bill verification, postal verification, geolocation cross-referencing, and regulatory requirements for address proof."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Address Verification

> Proof of address verification — utility bill verification, postal verification, geolocation cross-referencing, and regulatory requirements for address proof.

### How It Works Technically

**Document-Based Verification**:
1. User uploads document (utility bill, bank statement, government letter)
2. **OCR extraction**: Name, address, date, issuer extracted
3. **Cross-matching**:
   - Name vs. account name (fuzzy matching with transliteration support)
   - Address vs. provided address (component-by-component: street, city, postal code, country)
   - Date validation: must be within 3 months (some providers: 90 days)
4. **Document authenticity**: Format analysis, logo verification, template consistency
5. **Address normalization**: Standardize address format (USPS CASS, Royal Mail PAF, UN/LOCODE)

**Address Verification Service (AVS)**:
- Real-time database lookup for address existence
- **USA**: USPS address validation API, USPS CASS certification
- **UK**: Royal Mail Postcode Address File (PAF)
- **Global**: Google Maps Address Validation API, Loqate, SmartyStreets
- Returns standardized format + deliverability flag

**Postcard Verification**:
- Physical postcard with unique code sent to address
- User enters code online = proves mailbox access at stated address
- Used by: Google (Google My Business), some financial institutions
- 1–2 week delay; high assurance

**Non-Document Methods**:
- **Open Banking/Bank statement pull**: OAuth-based bank data access (Plaid, Nordigen, TrueLayer) can confirm address on file with bank
- **Electoral roll lookup**: UK, Australia — verify against voter registration
- **Credit bureau check**: Cross-reference address with credit file (requires consent)

**Acceptable Documents** (per Stripe, Wise standards):
- Utility bills (gas, electric, water, landline) — NOT mobile phone
- Bank/credit card statements
- Government-issued letters
- Council tax bills (UK)
- Vehicle registration documents
- Tenancy/lease agreements
- Not accepted: Screenshots, Wise/Revolut/PayPal statements (digital-only), PO Box

### Common Providers/SDKs

| Provider | Coverage | Specialty |
|----------|----------|-----------|
| **Veriff PoA** | Global | OCR + name/address matching, fraud signals |
| **Onfido PoA** | 190+ countries | Integrated with ID verification flow |
| **IDWise** | MENA + Global | Multi-language including Arabic, Cyrillic |
| **Argos Identity** | Global | AI-powered OCR, 13+ document types |
| **Shufti Pro** | 240+ countries | 99.77% accuracy, real-time |
| **Socure** | USA | Address RiskScore, 91.7% name-address correlation |
| **SmartyStreets** | USA | USPS CASS certified, real-time AVS |
| **Loqate (GBG)** | Global | Address lookup, geocoding |
| **Google Address Validation API** | Global | Recent addition, high accuracy |

### Accuracy/Security Level

- **Automated matching**: 85–95% for clean, legible documents
- **Forgery detection**: Moderate — sophisticated forged utility bills can pass basic checks
- Enhanced by cross-referencing with ID document name and government databases

### UX Friction Level

- **High**: Requires finding, photographing, and uploading a recent document
- 20–40% drop-off at address verification step in some funnels
- Pre-fill from banking data (Open Banking) dramatically reduces friction

### Regulatory Considerations

- **GDPR**: Utility bills contain sensitive utility usage data; minimize processing
- **AML/KYC regulations**: BSA (USA), 4AMLD (EU) require address verification for certain transaction thresholds
- USA PATRIOT Act Section 326: Requires address verification for financial account opening
- **Retention**: BSA requires 5-year retention of address verification records

---