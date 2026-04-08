---
title: "Platform Verification Case Studies"
category: identity-and-verification
summary: "How major platforms implement verification — Tinder, Uber, Airbnb, Stripe, Wise, Binance, Gojek, Dana, Tokopedia, and escrow platforms."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Platform Verification Case Studies

> How major platforms implement verification — Tinder, Uber, Airbnb, Stripe, Wise, Binance, Gojek, Dana, Tokopedia, and escrow platforms.

### 🔴 Tinder

**Verification Stack**:
1. **Phone verification**: SMS OTP at signup (mandatory)
2. **Photo Verification** (optional, blue camera icon):
   - Video selfie captured in-app
   - Facial geometry extracted (biometric data per BIPA in Illinois)
   - Compared against all profile photos
   - Liveness detection to prevent static image submission
3. **ID + Photo Verification** (optional, blue checkmark):
   - Government-issued ID upload (driver's license, passport)
   - OCR extracts DOB → age verification (must be 18+)
   - Face matching: ID photo vs. video selfie vs. profile photos
   - **US only**: Sex offender registry (RSO) check using ID data via third-party
4. **Profile photos**: Moderated for explicit content; age assessment ML
5. **Email**: Optional additional signal

**Data partners**: Third-party ID verification provider (not publicly named)
**ID retention**: Only redacted screenshot of ID front kept; no full ID stored
**Biometric data**: Retained only for fraud prevention; deleted on account deletion

**Outcome**: ID-verified users see 67% increase in matches (social incentive for adoption)

---

### 🟡 Uber

**Rider Verification**:
- Phone + email verification at signup
- Optional ID verification in select markets (Jumio, Veriff, Socure as named partners)
- ID + Selfie: Liveness detection + face match in some countries
- Selfie-only: Anti-impersonation check

**Driver/Delivery Partner Verification**:
- **Onboarding**: Full document KYC — ID card/passport, driver's license, vehicle registration, insurance proof, background check
- **Real-Time ID Check (RTIC)**:
  - Randomly prompts drivers to take selfie during active sessions
  - Microsoft Azure Face API compares selfie to stored profile photo
  - **Match**: Proceed; **No match**: Human review by 3 independent reviewers
  - 99%+ pass rate
  - GPS movement detection: Prompts paused while vehicle is moving (safety)
  - EU/UK: Optional face-matching; alternative review process offered due to biometric laws
  - Periodic check: Every 14 days minimum identity re-verification

**Third-party partners**: Jumio, Veriff, Socure (riders); Microsoft Azure (RTIC)

---

### 🟢 Airbnb

**Verification Stack**:
1. **Basic info**: Full legal name, DOB, address, email, phone
2. **Government ID**: Passport, national ID, driver's license, residence permit — clear photos of all 4 corners required
3. **Selfie + liveness**: Real-time selfie captured; liveness detection (not static image); face matched to ID photo
4. **Database cross-checks**: Public records verification in some countries
5. **Verified badge**: Displayed on profile after completion
6. **Automated + human review**: AI first pass; human escalation for edge cases
7. **Background checks** (hosts in some markets): Criminal record check

**KYC trigger**: Required for certain bookings/hosts based on risk assessment

---

### 🔵 Stripe

**Business (Merchant) Verification**:
- **Business identity**: Legal entity name, EIN/TIN, business address
- **Individual verification**: Identity for key company representatives, beneficial owners (UBO)
- **Documents accepted by country**: Passport, national ID, driver's license (country-specific)
- **Address verification**: Utility bill, bank statement, government letter (max 12 months old)
- **NFC MRZ verification**: For supported documents
- **Business docs**: Certificate of incorporation, IRS EIN confirmation letter (SS-4)

**Stripe Identity** (for Stripe customers to verify their own users):
- Veriff-powered (partial) + proprietary
- ID verification + liveness + face match
- Supports 50+ countries

**Stripe's own KYC**: EDGAR filing cross-check, credit bureau, sanctions screening

---

### 🟣 Wise (TransferWise)

**Individual Verification**:
- **Identity**: Passport (photo page), national ID (both sides), photo driving license
- **Selfie/Live photo**: Face photo compared to ID; cannot upload existing photo — must be taken in-moment
- **Address**: Utility bill, bank statement, council tax, rental agreement — within 3 months (exceptions: yearly council tax, government letters within 12 months)
- **Enhanced verification trigger**: Large transfers, high-risk jurisdictions, amount thresholds (15,000 EUR equivalent triggers stronger verification)
- **Auto-verification**: First bank transfer from name-matched bank account can auto-verify identity (< 15,000 EUR certain currencies)
- **Source of funds**: For large transfers; may require employment contract, property sale evidence

**Process**: Automated OCR → automated face match → automated liveness → human review if flagged (2 business day SLA for manual review)

---

### 🟠 Binance

**KYC Tiers**:

| Level | Requirements | Withdrawal Limit |
|-------|-------------|-----------------|
| Unverified | Email + password only | Cannot use (since 2021) |
| Basic (Verified) | Name + DOB + address + government ID + selfie facial recognition | ~$1,000,000 USD/day crypto |
| Advanced (Verified Plus) | Basic + proof of address document | Higher limits |
| Enterprise | Business KYC documents | Institutional |

**Verification Process**:
1. Personal details form
2. Government ID selection (passport, national ID, driver's license)
3. Document photo upload (front + back for ID cards)
4. Facial recognition: Live video selfie; user prompted to follow on-screen movements (head rotation)
5. Automated AI verification → typically minutes to hours
6. Manual review for edge cases

**Biometric**: Active liveness (head rotation prompt) during selfie
**Anti-fraud**: Identity Graph cross-checking; device binding; behavioral analytics

---

### 🟤 Gojek / Grab

**Driver/Delivery Partner Onboarding**:

*Gojek (Indonesia)*:
- eKTP verification via Dukcapil API integration (real-time Indonesian national ID database)
- SIM (Surat Izin Mengemudi) — driving license upload
- STNK (Surat Tanda Nomor Kendaraan) — vehicle registration
- SKCK (Surat Keterangan Catatan Kepolisian) — police clearance certificate
- Vehicle photos (front, back, side)
- Selfie + ID face match
- Vehicle inspection at service center for some categories
- KYC Form submission with complete personal data

*Grab (Southeast Asia)*:
- KYC Form with full identity info
- IC/KTP/passport based on country
- Driver's license
- Vehicle registration
- Insurance certificate
- Background check (CTOS in Malaysia, equivalent in other countries)
- Selfie verification during registration
- Profile photo for passenger-facing display
- Ongoing: Periodic selfie re-verification (similar to Uber RTIC)

**Passenger Verification**:
- Phone number + OTP (primary)
- Email optional
- Payment method linking (credit card, e-wallet) adds fraud signal

---

### 🔵 Dana / OVO / GoPay (Indonesian Fintech)

**Regulated by**: Bank Indonesia (BI), OJK (Otoritas Jasa Keuangan), PPATK (Financial Intelligence Unit)

**DANA Indonesia**:
- Phone number registration (SIM card tied to NIK per Indonesian regulation)
- **DANA Basic**: Phone-only, low transaction limits (IDR 1M/month)
- **DANA Premium/Verified**:
  - eKTP upload (both sides)
  - Face recognition (selfie + eKTP photo matching) via Dukcapil API integration
  - Passive liveness detection
  - Sometimes address confirmation via utility bill
  - NIK (Nomor Induk Kependudukan) cross-reference with Dukcapil database
  - Processing: 24–48 hours (mix of automated + human review)
- **Integration with Dukcapil**: Primary verification source — 50% reduction in processing time after integration (per DANA KYC team data)

**OVO (Lippo Group)**:
- Phone + SIM registration (mandatory for all e-wallets per BI regulation)
- **OVO Basic**: IDR 2,000,000/month transaction limit
- **OVO Premium**: Enhanced KYC — eKTP + selfie + data cross-reference; IDR 10,000,000/month
- OVO Club and Premier: Higher tiers with bank account linking

**GoPay (GoTo/Gojek)**:
- Embedded in Gojek super-app
- Phone + Gojek account (driver/passenger)
- **GoPay Basic**: OTP-verified
- **GoPay KYC**: eKTP + selfie + liveness; Dukcapil cross-reference
- **Bank account linking**: Additional trust signal

**Key Indonesia-specific Context**:
- **NIK (National ID number)**: 16-digit number in eKTP tied to Dukcapil database; foundation of digital KYC
- **SIM card real-name registration (2018)**: All SIM cards must be registered with NIK; enables phone-to-identity linkage
- **POJK 12/2021**: OJK regulation mandating e-KYC for financial technology providers
- **BI PBI No. 23/6/PBI/2021**: E-money issuer license requirements including KYC standards
- **Anti-money laundering (PPATK)**: STR (Suspicious Transaction Report) thresholds at IDR 500M+

---

### 🔵 Tokopedia / TikTok Shop Indonesia

**Buyer Verification**:
- Phone OTP (mandatory)
- Email optional
- Payment method verification (bank transfer confirmation, credit card)
- **Full KYC**: Required for Tokopedia-financial products (TokoModal, etc.)

**Seller Verification**:
- Phone OTP
- NIK + eKTP for verified seller badge
- Bank account (must match KTP name)
- **Official Store / Power Merchant**: Higher tiers require full KYC

**Escrow/Rekber System (Built-in)**:
- Tokopedia operates as built-in escrow provider (regulated by OJK)
- Buyer pays → Tokopedia holds funds → Seller ships → Buyer confirms receipt → Tokopedia releases funds
- Dispute resolution: 3-day window after delivery; Tokopedia mediates
- Compliant with OJK e-commerce regulations
- Rating/review system: Social trust layer alongside financial escrow
- Fraud protection: Purchase Protection program for qualified transactions

**Personal Rekber (Community/P2P)**:
- Used in Facebook/WhatsApp/Kaskus/KBLI buy-sell groups
- Trusted individual acts as intermediary
- High risk — identity verification of rekber depends on community reputation, not formal process
- No OJK oversight
- Attack vector: Fake rekber "penipuan rekber" scam — impersonating trusted rekbers

---