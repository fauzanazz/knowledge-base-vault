---
title: "Indonesian eKYC Providers & Identity Verification Infrastructure"
category: identity-and-verification
summary: >
  A comprehensive reference guide to the Indonesian digital identity ecosystem — covering eKYC,
  biometrics, OTP delivery, government NIK/KTP verification (DUKCAPIL), digital signatures
  (PSrE), and open finance APIs. Profiles seven major providers with feature breakdowns,
  API patterns, pricing, and certifications.
sources:
  - https://vida.id/kyc
  - https://docs.vida.id/identity-stack/verify/
  - https://verihubs.com/en/product/
  - https://docs.verihubs.com/docs/ocr
  - https://privy.id/business
  - https://privy.com.au/indobound
  - https://fazpass.com/
  - https://doc.fazpass.com/
  - https://www.zenziva.id/harga/
  - https://www.zenziva.id/dokumentasi/
  - https://onebrick.io/
  - https://www.prnewswire.com/apac/news-releases/brankas-is-first-company-in-indonesia-to-be-awarded-open-banking-data-license-302120563.html
  - https://jangkargroups.co.id/dukcapil-kemendagri/
updated: "2026-04-08"
---

# Indonesian eKYC Providers & Identity Verification Infrastructure

Indonesia's digital identity landscape is one of the most developed in Southeast Asia, underpinned
by a national 16-digit citizen ID number (NIK / e-KTP) managed by the government's DUKCAPIL
registry. The ecosystem spans government infrastructure, private eKYC aggregators, certified
electronic signature authorities (PSrE), OTP delivery networks, and open-finance data APIs.

This article profiles seven key players and provides an architectural overview for system designers
integrating identity verification into Indonesian fintech, banking, or consumer applications.

---

## Regulatory Context

| Regulation | Body | Relevance |
|---|---|---|
| UU ITE No. 11/2008 (amended 2024) | DPR / Kominfo | Legal basis for electronic signatures & digital identity |
| PP PSTE | Kominfo | Electronic systems & transactions — mandates identity mechanisms |
| Permendagri No. 102/2019 | Kemendagri | Rules for third-party access to DUKCAPIL population data |
| OJK eKYC Regulation | OJK (Financial Services Authority) | Mandatory eKYC for fintech, P2P lending, banks |
| UU PDP (Personal Data Protection Law) | Kominfo | Data residency & consent rules (2022, enforced 2024) |
| PSrE Accreditation Framework | Kominfo | Governs Certificate Authorities issuing legally valid digital signatures |

Key licences to watch: **PSrE** (Penyelenggara Sertifikasi Elektronik) for digital signatures;
**PJP Category 2** (Bank Indonesia) for open-banking account information services; **OJK Sandbox**
status for fintech identity providers.

---

## 1. VIDA.id — Full-Stack Digital Identity & eKYC

**Website:** [vida.id](https://vida.id) | **Docs:** [docs.vida.id](https://docs.vida.id)

### Overview

PT Indonesia Digital Identity (VIDA) is Indonesia's leading full-stack digital identity platform,
offering eKYC, biometric verification, and certified digital signatures under a single vendor.
VIDA is one of Indonesia's seven officially accredited **PSrE** (Certificate Authorities) —
appointed by both the Ministry of Communication & Information (Kominfo) and the Ministry of Finance
(Decree No. 584/2022) to issue legally valid electronic certificates. VIDA is also integrated
directly with the DUKCAPIL national population database for authoritative NIK cross-referencing.

### Core Products

| Product | Description |
|---|---|
| **VIDA Verify (eKYC)** | End-to-end identity verification: document capture → OCR → DUKCAPIL cross-check → face match → liveness |
| **VIDA Sign** | Certified digital signature (PSrE-issued certificate). Two tiers: `e-Sign` (free, uncertified) and `Digital Sign` (PSrE-certified, ~IDR 3,000/signature or IDR 30,000 for 10) |
| **VIDA Liveness** | Standalone liveness detection SDK — Passive, Color Flash, Active Gesture, Zoom Liveness |
| **VIDA Face Match** | 1:1 face comparison against ID document photo |
| **KYC Web SDK** | Embeddable JavaScript SDK for browser-based flows |
| **KYC Mobile SDK** | Native Android + iOS SDK with guided capture UX |

### API & Integration Patterns

VIDA's API follows a REST/JSON pattern with Bearer token authentication. The KYC workflow is:

```
1. Partner calls → /verify/start  (creates session)
2. User submits → document capture (OCR + quality gates)
3. SDK calls →   /verify/liveness  (returns score 0–1; success if ≤ 0.95)
4. Backend calls → /verify/face-match  (selfie vs. ID photo)
5. Backend calls → /verify/dukcapil  (NIK + data fields matched against gov DB)
6. Result returned as structured JSON with verification status per field

**Liveness scoring:** Score range 0–1; threshold ≤ 0.95 = PASS (lower score = more human-like).
Active liveness supports gesture prompts and zoom challenges for high-risk contexts.

**Multi-country support:** KTP (Indonesia), Passport, Driving License, MyKAD (Malaysia) — making
VIDA viable for regional deployments across SEA.

### Certifications

- ✅ PSrE (Kominfo) — Accredited Certificate Authority
- ✅ Ministry of Finance Appointed PSrE (Decree No. 584/2022) — for Coretax / tax filing signatures
- ✅ DUKCAPIL direct integration (government-to-business MoU)
- ✅ OJK-compliant eKYC infrastructure

### Pricing

VIDA operates on an **enterprise/contact-sales** model. Digital Sign top-up costs approximately
**IDR 30,000 for 10 certified signatures** (IDR 3,000/signature). Volume discounts apply at scale.
eKYC API pricing is custom and usage-based.

---

## 2. Verihubs — AI Biometric Verification Suite

**Website:** [verihubs.com](https://verihubs.com) | **Docs:** [docs.verihubs.com](https://docs.verihubs.com)

### Overview

Verihubs is Indonesia's leading standalone AI biometric verification provider, specializing in
face recognition, liveness detection, and OCR. It targets fintech, digital banks, insurance, and
enterprises needing high-throughput verification. Trusted by 400+ clients including major Indonesian
financial institutions.

### Core Products

| Product | Key Spec |
|---|---|
| **Face Recognition** | NIST FRTE 1:1 & 1:N certified; 99.95% accuracy on LFW benchmark; supports 1:1 (verification) and 1:N (identification) modes |
| **Liveness Detection** | NIST FATE PAD certified; active + passive modes; >95% accuracy detecting real vs. spoofed faces; anti-spoofing against photos/video/3D masks |
| **OCR KTP** | Extracts all text fields from e-KTP; 98% accuracy; photocopy detection; grayscale detection (rejects B&W scans); quality detection (blur/glare/dark); autocorrect algorithm. Sync & async endpoints |
| **OCR Extraction** | General OCR for KTP, SIM, NPWP, KK, Passport, invoices, contracts |
| **Watchlist Screening** | AML/sanctions screening for KYC compliance |
| **Deepfake Detection** | AI-powered deepfake image/video detection |
| **ID Check** | DUKCAPIL-linked NIK verification (via Verihubs middleware) |

### API Patterns

Verihubs provides both **REST API** and native **SDK** (Android, iOS, Web):

```http
# OCR KTP — Synchronous
POST https://api.verihubs.com/v1/ocr/extract
Authorization: Bearer {api_key}
Content-Type: multipart/form-data

# Face Recognition 1:1
POST https://api.verihubs.com/v1/face/verify
{ "image1": "<base64>", "image2": "<base64>" }

# Liveness Check
POST https://api.verihubs.com/v1/liveness/check
{ "image": "<base64>", "mode": "passive" }


OCR also supports **asynchronous** mode (`/extract-async` + `/extract-async/result` polling) for
high-volume batch processing.

### Certifications

- ✅ ISO 27001 / ISO/IEC 27001:2022 — Information Security Management
- ✅ NIST FRTE 1:1 & 1:N — Face recognition performance (highest global benchmark)
- ✅ NIST FATE PAD — Presentation Attack Detection (liveness)

### Pricing

Verihubs uses **custom / post-paid pricing** based on volume commitment. Key commercial terms:
- **30% discount** on AI Verification Suite for qualified partners with volume commitment
- **2-month free trial** for ID Check & Watchlist Screening (no commitment required)
- Post-paid billing: pay per actual usage with transparent monitoring dashboard
- Free consultation available

Pricing is not publicly listed; contact sales for a custom quote.

---

## 3. Privy.id — Digital Identity & Certified Signature Leader

**Website:** [privy.id](https://privy.id) | **Console:** [console.privy.id](https://console.privy.id)

### Overview

Privy is self-described as "Indonesia's #1 Digital Identity and Digital Signature" platform, with
**68 million verified users** and **200,000+ enterprise clients** (including Bank Mandiri, BPJS,
Pizza Hut). It is one of Indonesia's accredited **PSrE** providers under Kominfo, integrated with
DUKCAPIL and recognized by OJK's regulatory sandbox — making it unique as the **only digital
identity provider in the OJK Sandbox**.

### Core Products

| Product | Description |
|---|---|
| **PrivyID** | Reusable verified digital identity — "1 in 4 Indonesians already onboard." Acts as SSO for partner apps |
| **eKYC API** | OCR, liveness detection, face recognition — full onboarding flow via API/SDK |
| **Certified Digital Signature** | PSrE-issued, legally equivalent to wet signature (UU ITE). Backed by PKI + HSM |
| **e-Meterai** | Electronic stamp (materai digital) — legally required for high-value documents in Indonesia |
| **Document Management** | Cloud document storage, envelope management, audit trails |
| **On-Premise Deployment** | For enterprises with data residency or security requirements |

### API Patterns

Privy's API uses enterprise token + merchant key authentication. Registration flow:

```python
# Python SDK example (landx-id/privy-python-sdk)
prv = Privy(
    privy_enterprise_token="key-123",
    privy_merchant_key="xxxxx",
    privy_username="foo",
    privy_password="bar",
    privy_id='TE1111',
    production=False
)

# Register user for eKYC
prv.register_user(
    nik="1234567891234567",   # 16-digit NIK
    name="John Doe",
    email="john@example.com",
    phone="08233324223",
    selfie="<base64_selfie>",  # liveness selfie
    ktp="<base64_ktp_photo>",  # ID card image
    date_of_birth="1990-01-01"
)

# Check registration/verification status
prv.register_status(token="<user_token>")

# Upload document for signing
prv.upload_document(title="Contract", document_path="/path/to/doc.pdf")


The eKYC flow: OTP verification → ID scan (OCR) → face verification + liveness → DUKCAPIL
cross-check → PSrE electronic certificate issuance (Level 4 security).

### Certifications

- ✅ PSrE — Kominfo-accredited Certificate Authority
- ✅ OJK Sandbox — First and only digital identity provider in Indonesia's OJK regulatory sandbox
- ✅ DUKCAPIL integrated — Issues Electronic Certificates Level 4 backed by Dukcapil data
- ✅ Bank Indonesia registered — Financial Technology Supporting Institution
- ✅ Registered PSE (Penyelenggara Sistem Elektronik) — Kominfo registration
- ✅ IKD Cluster eKYC Provider — OJK classified

### Pricing

| Tier | Price |
|---|---|
| **Personal Plan (monthly)** | IDR 54,000/month — unlimited digital signatures |
| **Personal Plan (yearly)** | IDR 395,000/year (save 39%) — unlimited signatures + e-Meterai + 10 GB storage |
| **Enterprise / Business** | Custom pricing — contact sales via privy.id/business |

Enterprise pricing includes volume-based eKYC API calls, document management seats, and white-label
options.

---

## 4. Fazpass — Multi-Channel OTP Aggregator & Seamless Auth

**Website:** [fazpass.com](https://fazpass.com) | **Docs:** [doc.fazpass.com](https://doc.fazpass.com)

### Overview

Fazpass is Southeast Asia's first passwordless authentication aggregator. Rather than a single OTP
channel, Fazpass acts as an **OTP broker** — connecting businesses to 30+ underlying OTP providers
(Telkomsel, Indosat, XL, WhatsApp BSPs, etc.) through a **single API integration**. Trusted by
DANA, Tokopedia, Traveloka, IKEA, Indomaret, and Indodax.

### Core Products

| Product | Description |
|---|---|
| **All-in-One OTP Platform** | Single API for SMS, WhatsApp, Email, Missed-Call OTP. Free platform tier. Automatic failover between providers |
| **Seamless Authentication** | Passwordless / biometric auth that reduces OTP cost by up to 60%. Pay-per-successful-verification model |
| **OTP Analytics Dashboard** | Real-time monitoring of delivery rates, cost per channel, provider performance |

### API Patterns

```http
# Base URL
https://api.fazpass.com/

# Request OTP (auto-routes to best provider/channel)
POST /v1/otp/request
{
  "phone": "6281234567890",
  "Email": "user@example.com"    # optional
}
# Response: { "status": true, "otp_id": "...", "data": {...} }

# Verify OTP
POST /v1/otp/verify
{
  "otp_id": "...",
  "otp": "123456"
}

# Mark as verified (for Seamless auth flow)
POST /v1/otp/update_verified
{
  "otp_id": "..."
}


Authentication uses a **merchant_key (Token)** in request headers. The Android SDK
(`fazpass-android`) handles device-bound Seamless Authentication flows client-side.

### Channel Support

- **SMS OTP** — all major Indonesian operators (Telkomsel, XL, Indosat, Tri, Smartfren)
- **WhatsApp OTP** — via WhatsApp Business API partners
- **Email OTP**
- **Missed Call OTP** — user receives call; last N digits = OTP (zero data cost for user)
- **Voice OTP** — Text-to-voice call delivery

### Certifications

- ✅ ISO 27001 — Information Security Management
- ✅ FIDO Alliance Member — Passwordless authentication standards

### Pricing

- **All-in-One OTP Platform:** **Free** (aggregation layer is free; underlying channel costs pass
  through at guaranteed-best rates)
- **Seamless Authentication:** Pay-per-successful verification (cost significantly lower than
  traditional OTP; up to 60% savings claimed)
- **Model:** Pay-as-you-go / pay-per-success; no setup fees

---

## 5. Zenziva — Local SMS & WhatsApp OTP Provider

**Website:** [zenziva.id](https://zenziva.id) | **Docs:** [zenziva.id/dokumentasi](https://www.zenziva.id/dokumentasi/)

### Overview

PT Zenziva Zetta Indonesia (founded 2008, incorporated 2015) is a domestic cloud messaging gateway
offering SMS Masking, SMS OTP, WhatsApp Business API, and Text-to-Voice. It is a **direct**
provider (not an aggregator), working directly with Indonesian telco operators. Suited for
lower-volume use cases or businesses preferring a local Indonesian vendor with IDR-denominated
billing.

### Products

| Product | Description |
|---|---|
| **SMS Masking (Reguler)** | Branded sender ID; one-way delivery; general notifications |
| **SMS OTP (Premium/Priority Lane)** | Dedicated OTP/verification lane; mandatory for OTP content |
| **WhatsApp Business Account (WABA)** | Official green-tick WA; utility, authentication, marketing templates |
| **Text-to-Voice** | OTP delivered via automated voice call |
| **WA Center** | Self-hosted WhatsApp instance management |

### API Patterns

Zenziva uses a simple HTTP POST API authenticated via `userkey` + `passkey`:

```http
# SMS Masking
POST https://masking.zenziva.net/api/sendsms/
{ "userkey": "xxx", "passkey": "xxx", "nohp": "08123456789", "pesan": "Your OTP is 123456" }

# SMS Reguler
POST https://gsm.zenziva.net/api/sendsms/
{ "userkey": "xxx", "passkey": "xxx", "nohp": "08123456789", "pesan": "Hello" }

# WhatsApp (WA Center - self-hosted)
POST http://{SUBDOMAIN}/api/WAsendMsg/
{ "userkey": "xxx", "passkey": "xxx", "instance": "12345", "nohp": "628123456789", "pesan": "msg" }

# Text-to-Voice
POST https://console.zenziva.net/voice/api/sendvoice/
{ "userkey": "xxx", "passkey": "xxx", "to": "08123456789", "pesan": "Your OTP is 1 2 3 4 5 6" }

# Response (JSON)
{ "messageId": "157365", "to": "081111111111", "status": "1", "text": "Success" }

### Pricing (as of January 2026, excl. 11% VAT)

**SMS OTP / Premium Lane (per SMS, by deposit volume):**

| Operator | Min Deposit (IDR 500K–2M) | Max Deposit (≥ IDR 100M) |
|---|---|---|
| Telkomsel | IDR 650 | IDR 580 |
| XL | IDR 750 | IDR 680 |
| Indosat | IDR 690 | IDR 620 |
| Tri | IDR 690 | IDR 620 |
| Smartfren | IDR 770 | IDR 700 |

> ⚠️ OTP content (codes/tokens/numeric passwords) **must** use the Premium/OTP lane; using Reguler
> lane incurs a **200% penalty** surcharge.

**WhatsApp Business API (per message):**

| Template Type | Price/Message |
|---|---|
| Utility (order updates, notifications) | IDR 400 |
| Authentication (OTP) | IDR 610 |
| Marketing | IDR 610+ (varies) |
| Service (user-initiated, 24hr window) | Free |

**Sender ID registration:** IDR 2,000,000 minimum deposit required; approval takes 14–30 business
days. Tri Sender ID re-registration incurs an additional setup fee.

---

## 6. DUKCAPIL API — Government NIK/KTP Verification

**Portal:** [dukcapil.kemendagri.go.id](https://dukcapil.kemendagri.go.id)

### Overview

Direktorat Jenderal Kependudukan dan Pencatatan Sipil (DUKCAPIL), under the Ministry of Home
Affairs (Kemendagri), manages Indonesia's **national population database** — the authoritative
source of truth for all 275+ million Indonesian citizens' identity data. Every e-KTP holder has a
unique **16-digit NIK** linked to their full identity record.

DUKCAPIL's API is the backbone of Indonesia's eKYC ecosystem. All private eKYC providers (VIDA,
Privy, Verihubs, etc.) ultimately proxy their identity validation through DUKCAPIL data.

### What the API Validates

The DUKCAPIL API returns a **boolean match** per identity field:

```json
{
  "status": "true",
  "message": "Data cocok dengan database Dukcapil",
  "validation_fields": {
    "nik": true,
    "nama": true,
    "tanggal_lahir": true,
    "jenis_kelamin": true,
    "alamat": true,
    "pekerjaan": true,
    "status_perkawinan": true
  }
}


It does **not** return raw data from the database (privacy-preserving design) — only confirms
whether submitted fields match the authoritative record.

### Access Requirements

Direct DUKCAPIL API access requires a formal **Perjanjian Kerja Sama (PKS)** — a Cooperation
Agreement approved by the Director General of DUKCAPIL:

| Requirement | Detail |
|---|---|
| Legal basis | Permendagri No. 102 Tahun 2019 |
| Who can apply | Government agencies + private legal entities (PT/yayasan) |
| Access methods | Web Service (API) or Card Reader device |
| Key prerequisite | Signed PKS (MoU) with Dirjen Dukcapil |
| Processing time | 14–30 business days after document submission |
| Cost | Government fee for processing; API calls may carry usage fees |

### Practical Notes for System Designers

Most businesses **do not** apply for direct DUKCAPIL access. Instead, they use an accredited
eKYC middleware provider (VIDA, Privy, Verihubs) that already holds a PKS agreement. This
approach is:
- Faster to implement (days vs. months of government procurement)
- More feature-rich (biometrics layered on top of database match)
- Covered under the provider's liability

The **Identitas Kependudukan Digital (IKD)** is Indonesia's emerging mobile digital ID app,
which may eventually provide citizen-consented data sharing flows similar to Singapore's Singpass.

---

## 7. Brick (onebrick.io) & Open Finance Peers — Financial Data APIs

**Website:** [onebrick.io](https://onebrick.io)

### Overview & Ecosystem

Indonesia's open-finance layer is composed of several players. The key ones:

| Company | Focus | Status |
|---|---|---|
| **Brick** (onebrick.io) | Payments + financial data + identity verification | Launched 2020; ISO 27001; BI registered |
| **Brankas** | Open banking (AInS + payments); bank API management | First company in Indonesia to receive PJP Cat 2 AInS license (April 2024) |
| **Finantier** | Credit scoring + alternative data | B2B credit intelligence; acquired/pivoted |
| **Ayoconnect** | Embedded finance APIs (bills, payments, banking) | Broad network of FI integrations |

### Brick — Products & APIs

Brick provides a comprehensive open-finance API suite registered with Bank Indonesia
(reg. #11/38/PtK/3) and Kominfo (reg. #000487.01/DJAI.PSE/04/2021):

| API | Description |
|---|---|
| **Disbursement API** | Send money to 20,000+ bank destinations via API or Excel bulk upload |
| **Payment Link API** | Generate payment links supporting multiple payment methods |
| **Virtual Account API** | Create VAs across BCA, BNI, Mandiri, BRI, Danamon, CIMB Niaga, Sampoerna, Hana Bank |
| **Sub Accounts** | Manage subsidiary/client Brick accounts with fund segregation |
| **Financial Data API** | Read-only access to linked bank account balance & transaction history |
| **User Verification API** | Identity verification layer for onboarding |

```http
# Disbursement API
POST https://api.onebrick.io/v2/disbursement
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "amount": 150000,
  "bankCode": "BCA",
  "accountNumber": "1234567890",
  "referenceId": "TXN-001"
}

# Response
{
  "status": 200,
  "data": {
    "referenceId": "TXN-001",
    "amount": 150000,
    "status": "completed",
    "bankCode": "BCA",
    "accountNumber": "****4821",
    "accountName": "John Doe"
  }
}

### Brankas — Open Banking Data

Brankas holds Indonesia's first **PJP Category 2 License** (Bank Indonesia, April 2024),
authorizing two services:
- **AInS (Account Information Services):** Read-only access to customers' bank account data
  (balance, transactions) — with user consent
- **PIAS (Payment Initiation & Acquiring Services):** Initiate payments on behalf of customers

**Use cases:** Real-time fund verification before payments; credit underwriting with live bank
data; recurring payment debiting; pay-by-bank checkout.

### Certifications

- **Brick:** ISO 27001 (CBQA Global, cert #ISMS1001500); Bank Indonesia registered; Kominfo PSE registered
- **Brankas:** Bank Indonesia PJP Category 2 (first in Indonesia); used by 40+ banks

### Pricing

Both Brick and Brankas operate on **volume-based / enterprise pricing**. Brick publishes that it
offers "the best service rates in the market" with transparent pricing visible in the dashboard.
Specific per-transaction rates require contacting sales or signing up for sandbox access.

---

## Provider Comparison Table

| Provider | Primary Use Case | eKYC | Liveness | OCR | Digital Sig | OTP | Open Finance | Certifications | Pricing Model |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|---|---|
| **VIDA.id** | Full-stack identity | ✅ | ✅ Multi-mode | ✅ | ✅ PSrE | ❌ | ❌ | PSrE (Kominfo+MoF), DUKCAPIL | Custom / contact sales |
| **Verihubs** | AI biometrics | ✅ | ✅ NIST PAD | ✅ 98% acc | ❌ | ❌ | ❌ | ISO 27001, NIST FRTE 1:1/1:N, NIST FATE PAD | Custom post-paid; 2mo free trial |
| **Privy.id** | Digital ID + e-sign | ✅ | ✅ | ✅ | ✅ PSrE | ❌ | ❌ | PSrE (Kominfo), OJK Sandbox, DUKCAPIL | IDR 54K/mo personal; enterprise custom |
| **Fazpass** | OTP aggregation | ❌ | ❌ | ❌ | ❌ | ✅ Multi-channel | ❌ | ISO 27001, FIDO Alliance | Free platform + pay-per-use |
| **Zenziva** | SMS/WA OTP delivery | ❌ | ❌ | ❌ | ❌ | ✅ SMS+WA+Voice | ❌ | Local telco direct | From IDR 580/SMS OTP; IDR 610/WA auth |
| **DUKCAPIL API** | Gov NIK verification | ✅ (gov DB) | ❌ | ❌ | ❌ | ❌ | ❌ | Government authoritative source | PKS agreement required; gov fee |
| **Brick** | Open finance / payments | Partial | ❌ | ❌ | ❌ | ❌ | ✅ Full | ISO 27001, Bank Indonesia PJP, Kominfo PSE | Volume-based enterprise |
| **Brankas** | Open banking data | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ AInS+PIAS | Bank Indonesia PJP Cat 2 (first in ID) | Enterprise |

---

## Typical Integration Architectures

### Pattern A: Fintech eKYC Onboarding (Regulated)

```
User App
  │
  ├─► OTP verification ──────────► Fazpass (aggregator) ──► Telkomsel / WA BSP
  │
  ├─► ID Capture + OCR ──────────► VIDA / Verihubs / Privy (eKYC middleware)
  │                                      │
  ├─► Liveness Check ────────────►       │
  │                                      │
  └─► Face Match + NIK Verify ───► DUKCAPIL (via provider's PKS)
                                         │
                               Verified user identity
                               + PSrE certificate issued (if signing needed)

### Pattern B: High-Volume Digital Lending (P2P / BNPL)

```
Loan Application
  │
  ├─► Privy PrivyID SSO (pre-verified 68M users — zero friction re-KYC)
  │         │
  │         └─► If new user: VIDA or Verihubs eKYC API
  │
  ├─► Open Finance Data
  │         ├─► Brick (bank balance/transactions)
  │         └─► Brankas (AInS consent flow)
  │
  ├─► Credit Decision Engine (internal + Finantier alternative data)
  │
  └─► e-Contract Signing via Privy / VIDA Digital Sign (PSrE certified)

### Pattern C: Minimal-Cost OTP Only

```
User registration / login
  │
  ├─► Fazpass API (single integration)
  │         ├─► Auto-routes to cheapest/highest-delivery-rate channel
  │         │         ├─► Zenziva (SMS – from IDR 580/SMS)
  │         │         ├─► WhatsApp BSP (IDR 610/msg auth)
  │         │         └─► Missed call (cheapest; zero data cost)
  │         └─► Automatic failover if primary channel fails
  │
  └─► Fazpass Seamless (passwordless – device-bound, ~60% cost reduction)

---

## Key Decision Factors

### Choose VIDA if:
- You need an all-in-one vendor for eKYC + biometrics + certified digital signature
- You require PSrE-level legally binding electronic signatures
- You serve a Southeast Asian multi-country market (KTP, Passport, MyKAD support)
- You need white-label SDK embed in web or mobile apps

### Choose Verihubs if:
- You need best-in-class biometric accuracy (NIST-certified 99.95% face recognition)
- You want modular AI components (OCR, face, liveness) without a full eKYC platform
- You process high volumes and want post-paid billing with 2-month trial
- You need deepfake detection or AML watchlist screening

### Choose Privy if:
- You want to leverage a pre-verified user base (68M Indonesians already onboarded)
- You need certified digital signatures + e-Meterai in one platform
- Your use case is OJK-regulated fintech or financial services
- You want SSO/PrivyID to reduce re-KYC friction for returning users

### Choose Fazpass if:
- You need OTP delivery across multiple channels without managing multiple provider contracts
- You want automatic failover and best-price routing across 30+ providers
- You are moving toward passwordless / FIDO authentication
- Cost optimization of OTP spend is a priority

### Choose Zenziva if:
- You have lower volume OTP needs with simple requirements
- You prefer a local Indonesian vendor with IDR billing
- You need WhatsApp Official (green-tick) messaging for notifications + OTP
- You want a straightforward REST API with no abstraction overhead

### Access DUKCAPIL directly if:
- You are a large institution (bank, insurance, government) with existing government relationships
- You can absorb 14–30 day PKS approval process
- You want to avoid middleware markup costs at very high NIK verification volumes
- Otherwise: **use an eKYC provider as proxy**

### Choose Brick / Brankas if:
- You need bank account data aggregation (balance, transactions) for credit scoring or payment verification
- You need pay-by-bank or direct debit functionality
- You are building embedded finance or open banking features

---

## Data Privacy & Compliance Notes

1. **UU PDP (2022, enforced 2024):** Indonesia's Personal Data Protection Law requires explicit
   user consent before processing biometric or identity data. eKYC flows must present consent
   screens and support data deletion requests.

2. **Data Residency:** All population data from DUKCAPIL must remain within Indonesian borders.
   Cloud-based eKYC providers must use Indonesian data centers (or demonstrate equivalence).

3. **OJK eKYC Standards:** Financial institutions under OJK supervision must conduct eKYC meeting
   OJK Regulation POJK 77/2016 and subsequent updates. Accepted methods include video-call KYC
   and automated biometric KYC with DUKCAPIL cross-reference.

4. **PSrE Legal Validity:** Only digital signatures issued by Kominfo-accredited PSrE carry the
   same legal force as wet signatures under UU ITE. Using unaccredited e-signature tools creates
   legal risk for high-value contracts and regulated transactions.

5. **FIDO/Passwordless:** Indonesia's regulatory framework does not yet mandate FIDO2 but OJK
   has increasingly accepted biometric authentication as a valid second factor, making
   Fazpass Seamless compliant for most use cases.

---

## Summary Reference

| Dimension | Best Option |
|---|---|
| Most legally comprehensive eKYC | VIDA.id (PSrE + DUKCAPIL + biometrics) |
| Highest biometric accuracy | Verihubs (NIST certified) |
| Largest pre-verified user base | Privy.id (68M users, PrivyID SSO) |
| Lowest-friction OTP integration | Fazpass (single API, multi-channel) |
| Cheapest direct OTP delivery | Zenziva (from IDR 580/SMS) |
| Authoritative NIK source | DUKCAPIL API (gov) |
| Open finance / bank data | Brick / Brankas |

---

**Summary of what was done:**

**What I did:** Conducted 8 parallel web searches covering all 7 providers plus supplementary searches for certifications, API docs, pricing, and regulatory context.

**What I found:**
- **VIDA.id**: PSrE-certified (Kominfo + Ministry of Finance), full eKYC stack with multi-mode liveness, DUKCAPIL integration, Web/Mobile SDK. ~IDR 3,000/certified signature.
- **Verihubs**: NIST FRTE 1:1/1:N + NIST FATE PAD certified. 99.95% LFW face recognition accuracy, 98% OCR accuracy. ISO 27001. Custom post-paid pricing, 2-month free trial.
- **Privy.id**: 68M verified users, 200K+ companies, PSrE + OJK Sandbox (unique). Personal plan IDR 54K/month; enterprise custom. PrivyID SSO enables zero-friction re-KYC.
- **Fazpass**: ISO 27001 + FIDO Alliance. Single API for 30+ OTP providers. Free aggregation platform, pay-per-success for Seamless auth. Clients: DANA, Tokopedia, Traveloka.
- **Zenziva**: Direct telco provider, IDR 580–770/SMS OTP, IDR 610/WA auth message. Simple userkey+passkey REST API.
- **DUKCAPIL API**: Government source of 275M+ NIK records. Boolean match API. Requires PKS (MoU) agreement — 14-30 day approval. Most businesses use provider middleware instead.
- **Brick/Brankas**: ISO 27001 + Bank Indonesia PJP registered. Brankas = first company with PJP Category 2 AInS license (April 2024). REST API at api.onebrick.io.

**Issues encountered:** No public per-call pricing published by VIDA, Verihubs, Privy, Brick, or Brankas — all use custom enterprise pricing. Documented what was publicly available (Zenziva publishes full rate cards; Privy publishes personal plan pricing).