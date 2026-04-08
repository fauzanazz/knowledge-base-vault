---

---
title: "WhatsApp-Based Verification & Authentication Patterns for Indonesian Platforms"
category: identity-and-verification
summary: >
  A comprehensive guide to designing WhatsApp OTP authentication systems for
  Indonesian digital platforms. Covers Meta's per-message pricing model,
  WhatsApp Flows for in-chat verification, SMS vs WhatsApp cost trade-offs in
  the Indonesian market, how super-apps like Gojek, DANA, and OVO implement
  multi-channel auth, and the anti-fraud patterns needed to defend against SIM
  swap, OTP phishing, and APK-based malware attacks.
sources:
  - https://whatsbizapi.com/blog/whatsapp-business-api-pricing-2026
  - https://flowcall.co/blog/whatsapp-business-api-pricing-2026
  - https://maxchat.id/blog/perbandingan-biaya-sms-vs-whatsapp-otp/
  - https://www.authgear.com/post/sms-otp-vs-whatsapp-otp
  - https://www.gojek.com/en-id/help/gofoodweb/cara-masuk-ke-akun-gojek
  - https://www.dana.id/help-center/article/how-do-i-verify-my-dana-account-after-registration
  - https://www.ovo.id/cerdasfinansial/literasi-keuangan/modus-penipuan-akun/
  - https://bca.co.id/en/informasi/awas-modus/2023/02/09/09/23/waspadai-modus-kejahatan-sim-swap-jaga-data-pribadimu
  - https://www.onemarketer.net/identity-verification-with-proof-of-life/
  - https://developer.8x8.com/connect/docs/whatsapp/whatsapp-flows-best-practices
  - https://kejarkoding.com/
  - https://taptalk.io/blog/pembajakan-whatsapp-kode-otp-dan-cara-mengatasi
  - https://www.govinfosecurity.com/one-time-passcodes-are-gateway-for-financial-fraud-attacks-a-31341
updated: "2026-04-08"
---

# WhatsApp-Based Verification & Authentication Patterns for Indonesian Platforms

## Why Indonesia Is a WhatsApp-First Authentication Market

Indonesia is one of the largest WhatsApp markets in the world. With over 185 million
active users, WhatsApp is the dominant communication channel — opened dozens of times
per day by the average Indonesian smartphone user. This makes it the highest-reach,
highest-trust channel for delivering one-time passwords (OTPs) and other verification
signals, far surpassing traditional SMS in both delivery reliability and user familiarity.

At the same time, Indonesia presents unique challenges: a fragmented telco landscape with
aggressive A2P SMS carrier filtering, a high incidence of OTP-targeting fraud (SIM swap,
social engineering, fake BTS interception), and a rapidly growing fintech ecosystem
subject to OJK (Otoritas Jasa Keuangan) oversight. Designing authentication flows for
this market therefore requires both a cost-optimised delivery strategy and a defence-in-
depth fraud posture.

---

## 1. WhatsApp Business API — Pricing Model for Indonesia

### 1.1 The Shift to Per-Message Pricing (July 2025)

Meta made a major structural change to its pricing model in 2025, moving from
**conversation-based billing** (charged per 24-hour session window) to
**per-message billing** (charged on every individually delivered template message).
This change took full effect from **July 1, 2025**, and significantly affects how
high-volume OTP senders calculate costs.

Under the new model, every business-initiated template message incurs a charge in one
of three categories:

| Category          | Typical Use Cases                         |
|-------------------|-------------------------------------------|
| **Marketing**     | Promotional campaigns, re-engagement      |
| **Utility**       | Order confirmations, shipping alerts, reminders |
| **Authentication**| OTPs, 2FA codes, login verification codes |

A fourth category — **Service** (replies within a 24-hour customer-initiated window) —
remains free.

### 1.2 Indonesia-Specific Rates (2025–2026)

Pricing is determined by the **recipient's country code**, not where the sending
business is based.

| Category                      | Cost per Message (IDR) | Approx. USD |
|-------------------------------|------------------------|-------------|
| Marketing                     | Rp 350 – 586           | $0.022–0.037 |
| Utility                       | Rp 150 – 357           | $0.009–0.022 |
| Authentication (domestic)     | Rp 250 – 357           | $0.015–0.022 |
| Authentication-International  | Rp 1,940               | ~$0.12      |

> **Critical detail — Authentication-International:** When a business registered
> *outside Indonesia* sends an authentication OTP to an Indonesian phone number,
> Meta applies the **Authentication-International** surcharge (Rp 1,940 per message).
> This rate is roughly **5–8× higher** than the domestic authentication rate. Global
> SaaS products, fintech platforms, and international super-apps entering Indonesia must
> account for this in unit-economics modelling. The cost trap is easy to miss: the sender's
> BSP dashboard may show only base authentication pricing until the first international
> billing cycle.

### 1.3 Volume Tiers and BSP Markups

Meta does not sell WhatsApp Business API access directly to most businesses — access is
mediated through **Business Solution Providers (BSPs)** such as Vonage, Twilio, Infobip,
Wati, Qiscus, and local Indonesian providers like Maxchat and Qontak. BSPs add platform
and support fees on top of Meta's base rates, so the real landed cost per OTP typically
runs **10–40% higher** than the raw Meta price.

High-volume senders (>1M authentication messages/month) can negotiate tiered pricing
directly with BSPs. At scale, Indonesian BSPs can offer authentication templates at
**Rp 250–300** per delivered message, which is cost-competitive with premium A2P SMS.

### 1.4 Free Monthly Credits

Meta offers a limited free tier through the WhatsApp Business Platform:
- Up to **1,000 free service conversations** per month per WABA (WhatsApp Business
  Account)
- Free delivery of **utility templates** sent within the 24-hour active window
  (applicable from April 2025)

These do not apply to authentication messages — OTP delivery is always charged.

---

## 2. WhatsApp Flows for In-Chat Verification

### 2.1 What Are WhatsApp Flows?

**WhatsApp Flows** is Meta's framework for building interactive, multi-screen UI
experiences that run natively inside the WhatsApp chat interface. Instead of redirecting
users to a browser or a separate app, Flows render form components (text inputs, dropdowns,
photo pickers, date selectors) directly in the chat window on Android ≥23.19 and iOS ≥23.19.

For authentication and identity verification, Flows offer a richer alternative to the
classic "send a code, paste it in the app" OTP pattern.

### 2.2 Authentication Template Types on WhatsApp

Meta's authentication templates support three interaction patterns:

| Template Type        | Description | Android/iOS Support |
|----------------------|-------------|---------------------|
| **Copy Code**        | User receives the OTP code and manually copies it into the app | All platforms |
| **One-Tap Autofill** | A button in the WhatsApp message triggers automatic code injection into the requesting app | Android-first; iOS limited |
| **Zero-Tap**         | Code is injected without any user action (requires app-level integration with device hash) | Android only |

For Indonesian Android-heavy user bases (Android market share in Indonesia exceeds 90%),
**One-Tap Autofill** dramatically reduces friction and OTP abandonment rates.

### 2.3 Using Flows for KYC and Identity Verification

Beyond simple 6-digit OTPs, WhatsApp Flows enable full KYC workflows within the chat:


[User initiates registration]
        ↓
[Flow Screen 1: Enter full name + national ID (NIK) number]
        ↓
[Flow Screen 2: Upload KTP photo via PhotoPicker component]
        ↓
[Flow Screen 3: Selfie / liveness capture]
        ↓
[Backend: OCR + face match against NIK record]
        ↓
[Flow Screen 4: Confirmation + PIN creation]


This pattern is increasingly used by Indonesian fintech platforms for **upgraded
account tiers** (basic → verified) required under OJK e-money regulations, which mandate
identity verification above certain transaction thresholds (typically Rp 20 million
balance or Rp 40 million/month transaction value).

**Key implementation notes for Flows-based KYC:**

- **Sensitive data handling:** Use the `PhotoPicker` component for ID document images.
  Images must be uploaded to encrypted storage with access controls; Meta's own guidelines
  prohibit storing raw biometric data beyond what is needed for verification.
- **Validation:** WhatsApp Flows do not natively support regex validation on text inputs.
  Backend endpoint validation (via the Flow Endpoint specification) must be used to reject
  malformed NIK numbers or phone formats and return an `error-message` to the screen.
- **Data retention:** Set a 90-day maximum retention policy on KYC document images,
  aligned with OJK data governance guidelines.
- **Template approval:** Flows used for authentication must still be submitted for Meta
  template approval under the `AUTHENTICATION` category. Business Verification on the
  Meta Business Manager is a prerequisite.

### 2.4 Flow Architecture Pattern


  User's WhatsApp ──── HTTPS ────► Flow Endpoint (your backend)
       │                                    │
  [Renders screens]               [Validates fields]
  [Collects inputs]               [Calls identity API]
  [Shows confirmation]            [Issues session token]
       │                                    │
       └──────── Webhook ──────────────────►│
                                   [Writes to user record]
                                   [Triggers downstream auth]


The Flow Endpoint receives a `decryptedBody` containing screen inputs and must respond
within **10 seconds** to avoid timeout errors visible to the user.

---

## 3. SMS vs WhatsApp OTP — Cost Comparison in Indonesia

### 3.1 Raw Pricing Comparison (Indonesia Market, 2025–2026)

| Factor                  | SMS OTP                      | WhatsApp OTP                |
|-------------------------|------------------------------|-----------------------------|
| Price per message       | Rp 400 – 600                 | Rp 250 – 430 (BSP-dependent)|
| Delivery rate           | 70 – 85%                     | >98% (on active connections)|
| Delivery speed          | 10 – 30 seconds              | <5 seconds                  |
| End-to-end encryption   | No                           | Yes (E2EE)                  |
| Requires internet       | No                           | Yes (WiFi or mobile data)   |
| International sender    | Standard A2P (carrier-routed)| Rp 1,940 (Auth-International)|
| Branding                | Numeric short code           | Verified business name + logo|

> **Source note:** Indonesian SMS OTP rates (Rp 400–600) are quoted from Maxchat and
> Flowkirim, Indonesian BSPs serving the local market. These reflect A2P termination
> rates from major Indonesian carriers (Telkomsel, Indosat, XL Axiata) as of early 2026.

### 3.2 True Cost-Per-Verified-User Calculation

Raw per-message price is misleading when delivery rates differ. The relevant metric is
**cost per successfully delivered and used OTP**:

```
True Cost = (Price per message) ÷ (Delivery rate × Conversion rate)


For a platform sending 1,000,000 OTPs/month in Indonesia:

| Channel          | Price/msg | Delivery Rate | Cost per Delivered OTP | Monthly Total    |
|------------------|-----------|---------------|------------------------|------------------|
| SMS OTP          | Rp 500    | 78%           | Rp 641                 | Rp 641,000,000   |
| WhatsApp OTP     | Rp 357    | 98%           | Rp 364                 | Rp 364,000,000   |
| **Saving (WA)**  |           |               | **Rp 277 / OTP**       | **Rp 277M/month**|

The higher WhatsApp delivery rate (98% vs ~78% for international A2P SMS in Indonesia
due to carrier filtering) means WhatsApp delivers a **~43% lower true cost per delivered
OTP** in this example, despite the headline price being similar or slightly higher in
some BSP tiers.

### 3.3 When SMS Still Makes Sense

WhatsApp OTP is not universally superior in Indonesia:

- **No-internet scenarios:** Rural areas or low-end devices without stable data connections
  cannot receive WhatsApp messages. SMS reaches these users via carrier signalling.
- **Unregistered WhatsApp numbers:** If the target phone number is not registered on
  WhatsApp, the message fails silently. An SMS fallback must be triggered.
- **Elderly or feature-phone user segments:** OVO and Gojek both serve user bases that
  include non-smartphone users.
- **Carrier billing flows:** Some Indonesian microloan and operator-billing products
  require SMS for regulatory or billing-reconciliation reasons.

### 3.4 Recommended Channel Strategy

```
Primary:   WhatsApp OTP (lower true cost, higher trust, E2EE)
           ↓ (if no WA delivery receipt after 15s)
Fallback:  SMS OTP
           ↓ (if SMS fails after 60s)
Secondary: Voice OTP (robocall to phone number)
           ↓ (for account recovery only)
Tertiary:  Email OTP + manual ID check


This waterfall pattern is used by mature Indonesian platforms and reduces total
verification drop-off to under 2%.

---

## 4. How Leading Indonesian Apps Implement WhatsApp Auth

### 4.1 Gojek — Multi-Modal Authentication with WA as Default

Gojek's authentication system (serving ride-hail, food delivery, GoPay fintech, and
GoSend logistics) is one of the most sophisticated in the Indonesian market. Users
authenticate via:

1. **OTP (SMS or WhatsApp, user-selectable):** On the login screen, users explicitly
   choose delivery channel. Gojek defaults to WhatsApp if the number has an active WA
   account, otherwise falls back to SMS. The OTP is 4 digits, delivered to the
   registered phone number.

2. **Login Link (Magic Link via SMS):** A deep-link sent via SMS that auto-opens the
   Gojek app and completes login without manual OTP entry — useful for returning users
   on different devices.

3. **One Tap Login (Trusted Device):** After first login, Gojek marks the device as
   trusted. Subsequent logins on the same device skip OTP entirely, using device
   attestation. This significantly reduces WA/SMS costs for high-DAU users.

4. **Biometric Login (Fingerprint / Face ID):** Activated optionally by the user after
   initial registration; replaces OTP for subsequent sessions on a biometric-capable
   device.

5. **Silent Login / SIM Verification:** Gojek uses silent network authentication —
   the app sends a one-time SMS request and the platform verifies SIM card ownership
   without the user seeing or entering any code. This is the most frictionless path.

6. **Missed Call OTP:** A one-way voice call to the user's number is placed; the caller
   ID of the incoming call is the OTP digits. No data required.

**Security posture:** Gojek's help documentation explicitly warns: *"Gojek never asks
for your personal data, including an SMS containing the OTP code/login link."* This
anti-social-engineering disclaimer is surfaced at the exact moment users receive an OTP.

### 4.2 DANA — WhatsApp as Primary KYC Channel

DANA (PT Espay Debit Indonesia Koe), one of Indonesia's largest licensed e-money
platforms, uses WhatsApp OTP as an **explicit first-choice verification path** in its
account upgrade flow:

> *"Tap Verify via WhatsApp/SMS → Enter the OTP code sent to your registered number."*

DANA's user journey for account verification also involves bank card linkage (requiring
the bank's own authentication code), followed by a WhatsApp or SMS OTP, followed by
PIN creation. This layered approach satisfies OJK's tiered KYC requirements (Tier 1
through Tier 3 account upgrade paths).

For DANA's full-KYC (Tier 3) users who transact at or above Rp 20 million, an
additional in-person or video-call identity check may supplement the WhatsApp-delivered
OTP.

### 4.3 OVO — WhatsApp Channel in Security Alerting

OVO (PT Visionet Internasional) uses WhatsApp not only for OTP delivery but also for
**security notification push** — transaction alerts, suspicious login notifications,
and fraud warnings are dispatched via WhatsApp alongside SMS. Key design choices:

- **Two-way authentication:** OVO actively promotes enabling 2FA (both SMS and
  WhatsApp) to guard against account takeover.
- **OTP non-sharing reminder:** Every OTP message and security alert includes the
  line: *"Jangan bagikan kode OTP/PIN kepada siapapun, termasuk petugas OVO"* (Do not
  share OTP/PIN with anyone, including OVO staff).
- **APK alert integration:** OVO uses WhatsApp broadcast channels to warn users about
  malicious APK files circulating via WhatsApp (see §5 for the threat model).

### 4.4 BCA (Bank Central Asia) — SIM Swap Defence via WA

BCA, Indonesia's largest private bank, has published detailed consumer advisories
about SIM Swap fraud (see §5.1). BCA's mobile banking authentication uses:

- SMS OTP for standard transactions
- WhatsApp notification alerts for all significant transactions (configurable by user)
- A dedicated KlikBCA PIN as a second factor independent of phone-based OTP

BCA's WhatsApp-based transaction alerts serve as a real-time fraud detection trigger:
if a user receives a WA notification for a transaction they did not initiate, they can
immediately block the number via the carrier's call center before further damage occurs.

---

## 5. Anti-Fraud Patterns for WhatsApp-Based Authentication

### 5.1 Threat Landscape in Indonesia

The Indonesian market faces a distinct set of OTP-targeting attack vectors:

#### SIM Swap (Penipuan SIM Swap)
Fraudsters obtain a victim's personal data (often via phishing or data leaks), then
impersonate the victim at a carrier store or via call centre social engineering to swap
the victim's phone number onto a new SIM card. Once activated, all SMS OTPs — for
banking, e-wallets, and WhatsApp itself — are routed to the attacker's device.

BCA's 2023 advisory noted the Kominfo-recorded wave of SIM Swap frauds that cost victims
"billions of Rupiah" in aggregate. The Financial Services Authority (OJK) has since
issued guidance urging banks to add verification steps beyond SIM-based OTP.

> **WhatsApp's partial mitigation:** WhatsApp OTPs delivered to a pre-registered WABA
> number are not affected by SIM swap in the same way SMS is — the attacker would also
> need to log into the victim's WhatsApp account (which requires its own OTP or
> two-factor PIN). However, if the attacker SIM-swaps and WhatsApp re-registration
> is triggered by a new SIM, they can take over the WhatsApp account itself.

#### OTP Phishing via Social Engineering
The most prevalent attack in Indonesia: a fraudster calls the victim, poses as a bank
or e-wallet officer, and claims there is an "urgent security issue" requiring the victim
to read back an OTP they just received. Indonesian platforms have responded by printing
the anti-sharing warning directly inside every OTP message template.

#### Fake BTS (Base Transceiver Station) Attacks
Indonesia's cybersecurity community (as reported by TapTalk.io) has documented cases
of portable fake BTS devices intercepting SMS OTPs by impersonating carrier towers —
particularly targeted at high-value fintech users. WhatsApp OTP's **end-to-end encryption
(E2EE)** is structurally immune to this attack: even if a fake BTS intercepts the data
packet, the ciphertext cannot be decrypted without the recipient's WhatsApp private key.

#### Malicious APK Distribution via WhatsApp
OVO, Bank Mandiri, and CIMB Niaga have all warned users about malicious Android APK files
(`.apk`) circulated via WhatsApp disguised as wedding invitations, courier receipts, or
bank notices. Once installed, these apps contain spyware that reads incoming SMS messages
(including OTPs) and forwards them to attacker-controlled servers. Some variants also
intercept WhatsApp notification content.

**Countermeasure:** Store OTP codes in Android's `ClipboardManager` with a short
expiry (auto-clear after 60 seconds) rather than displaying them in a persistent
notification, to limit spyware exposure windows.

#### OTP Pumping (Toll Fraud)
Bots repeatedly trigger OTP send requests to large numbers of valid phone numbers, forcing
the platform to pay for thousands of WhatsApp authentication messages it never intended
to send. At Rp 357 per message, a 100,000-OTP pumping attack costs the victim platform
~Rp 35 million in a single incident.

### 5.2 Defensive Architecture Patterns

#### Rate Limiting and Cooling Periods

```python
# Pseudocode — OTP rate limiting
MAX_OTP_PER_PHONE_PER_HOUR = 3
MAX_OTP_PER_PHONE_PER_DAY  = 10
LOCKOUT_AFTER_FAILURES     = 5  # failed OTP attempts

def send_otp(phone_number):
    recent_sends = redis.get_count(f"otp:{phone_number}:hourly")
    if recent_sends >= MAX_OTP_PER_PHONE_PER_HOUR:
        raise RateLimitError("Too many OTP requests. Try again in 1 hour.")
    
    # Exponential backoff for resend
    last_send_ts = redis.get(f"otp:{phone_number}:last_send")
    min_resend_gap = 30  # seconds, doubles each resend attempt
    if (now() - last_send_ts) < min_resend_gap:
        raise CooldownError(f"Please wait {min_resend_gap}s before requesting again.")

#### OTP Expiry and Format Best Practices

| Parameter          | Recommended Value       | Rationale                                |
|--------------------|-------------------------|------------------------------------------|
| Code length        | 6 digits                | 5× harder to brute-force than 4 digits   |
| Expiry             | 5 min (login), 10 min (signup) | Limits replay window               |
| Character set      | Numeric only            | Required for WA Auth template format     |
| Invalidation       | Single-use + server-side expiry | Prevents OTP reuse after valid entry |
| Max failed attempts| 5, then 30-min lockout  | Prevents brute-force; balances UX        |
| Anti-sharing copy  | Hardcoded in template body | Platform-level; cannot be removed      |

#### CAPTCHA / Bot Detection Before OTP Send

Place a CAPTCHA challenge (or invisible device fingerprinting) *before* the OTP send
call, not after. This is the primary defence against OTP pumping: if the OTP request
never reaches the WhatsApp API, no charge is incurred.

Indonesian platforms commonly use:
- **Invisible reCAPTCHA v3** (score threshold ≥ 0.7 before sending)
- **Device fingerprint scoring** (new device + new phone number = require CAPTCHA)
- **Phone number pre-validation** (check that the number is a valid Indonesian format
  (+62) and not on a blocklist before calling the WABA send endpoint)

#### SIM Swap Signal Detection

Integrate with **telco SIM change APIs** (available from Telkomsel, Indosat, and XL
Axiata via their partner programmes, or via third-party providers like GSMA Mobile
Connect) to check whether the target SIM card was recently reissued:

```
IF (sim_age_since_last_swap < 24 hours) AND (transaction_value > Rp 500,000):
    → Block transaction, require video verification
    → Send push notification to registered email
    → Log event for fraud review


This prevents a SIM-swapped attacker from immediately draining a wallet even after
capturing the OTP.

#### WhatsApp-Specific: Account Takeover via Re-registration

When WhatsApp detects a re-registration of a number (e.g., attacker tries to register
the victim's number on a new device), Meta sends a re-registration OTP to the victim's
existing device *and* email. Platforms that use WhatsApp OTP as their sole auth factor
should:

1. Monitor for anomalous login patterns (new device ID after a WhatsApp OTP session)
2. Require re-authentication with a second factor (email OTP, biometric) when a
   new device is detected
3. Send an out-of-band security alert via email whenever a new device association
   is recorded

#### Multi-Layer Auth Stack (Recommended for Fintech)


┌─────────────────────────────────────────────────────────────┐
│  Layer 1 (Identity)  Phone number ownership via WA OTP      │
│  Layer 2 (Possession) Device binding / trusted device token  │
│  Layer 3 (Knowledge)  User-set 6-digit PIN or passphrase     │
│  Layer 4 (Inherence)  Biometric (fingerprint / face ID)      │
│  Layer 5 (Context)    Behavioural / risk scoring engine      │
└─────────────────────────────────────────────────────────────┘


High-value transactions (e.g., withdrawals >Rp 5 million) should require Layers 1+3
minimum, with Layer 4 strongly recommended. OJK's fintech regulations increasingly
expect this layered approach in licence applications and audits.

---

## 6. Integration Architecture Reference

### 6.1 End-to-End OTP Flow (Production Pattern)


[Client App] 
    │
    ├─(1) User enters phone number
    │
    ▼
[Backend API]
    │
    ├─(2) Validate phone format (+62xxxxxxxxxx)
    ├─(3) Check rate limit (Redis)
    ├─(4) Check device fingerprint + CAPTCHA score
    │
    ▼
[Fraud Engine]
    │
    ├─(5) SIM change age check (telco API)
    ├─(6) Risk score: device, IP, velocity
    │
    ▼ (if score < block threshold)
[OTP Service]
    │
    ├─(7) Generate 6-digit TOTP (30s window) — stored in Redis with TTL=300s
    ├─(8) SELECT channel: WhatsApp (primary) or SMS (fallback)
    │
    ▼
[WhatsApp Cloud API / BSP]
    │
    ├─(9) POST /messages — authentication template
    ├─(10) Receive delivery webhook (status: delivered / failed)
    │       └─ if failed after 15s → trigger SMS fallback
    │
    ▼
[Client App]
    │
    ├─(11) User enters OTP (or one-tap autofill)
    │
    ▼
[Backend API]
    ├─(12) Verify TOTP (constant-time compare)
    ├─(13) Invalidate token in Redis
    ├─(14) Issue session token (JWT / opaque token)
    └─(15) Log auth event (device, IP, channel, timestamp)

### 6.2 WhatsApp Authentication Template (Meta-Approved Format)

```json
{
  "name": "login_otp_id",
  "language": "id",
  "category": "AUTHENTICATION",
  "components": [
    {
      "type": "BODY",
      "text": "Kode verifikasi {{app_name}} kamu adalah *{{1}}*. Kode ini berlaku selama {{2}} menit.\n\n⚠️ Jangan bagikan kode ini kepada siapapun termasuk petugas {{app_name}}.",
      "example": {
        "body_text": [["482756", "5"]]
      }
    },
    {
      "type": "FOOTER",
      "text": "Jika kamu tidak meminta kode ini, abaikan pesan ini."
    },
    {
      "type": "BUTTONS",
      "buttons": [
        {
          "type": "OTP",
          "otp_type": "ONE_TAP",
          "text": "Salin Kode",
          "autofill_text": "Autofill",
          "package_name": "com.yourapp.id",
          "signature_hash": "<your_app_signing_hash>"
        }
      ]
    }
  ]
}

> **Note:** The `ONE_TAP` button type requires the app's SHA-256 signing certificate
> hash and the package name to be registered in the Meta template. This enables Android
> autofill without clipboard access, reducing spyware attack surface.

### 6.3 BSP Selection Criteria for Indonesia

| Criteria                   | Why It Matters for Indonesia                              |
|----------------------------|-----------------------------------------------------------|
| Local routing optimisation | Reduces latency on Telkomsel / XL Axiata networks         |
| OTP pumping protection     | Fraud-detection logic at the BSP layer adds a second line |
| SLA-backed delivery        | Indonesian enterprise requires contractual uptime guarantees |
| Authentication-International awareness | Correctly routes domestic vs international OTPs |
| Meta policy compliance     | No "grey routes" — grey route use risks WABA suspension   |
| Local IDR billing          | Avoids FX conversion overhead on Rp-denominated accounts  |
| Bahasa Indonesia support   | Template approval and support interactions in-language    |

---

## 7. Regulatory Context

Indonesian platforms deploying WhatsApp-based authentication must be aware of:

- **OJK POJK 6/2022 (Consumer Protection):** Requires clear disclosure to users about
  data collected during authentication and how it is used. WhatsApp OTP message content
  must not contain unnecessary personal data beyond the OTP code itself.
- **OJK e-Money Regulations (POJK 18/2018):** Tiered KYC thresholds require progressively
  stronger identity verification at higher transaction limits. WhatsApp Flows-based
  KYC (with ID document upload + face match) is an accepted digital channel for Tier 2
  upgrades.
- **Kominfo Data Privacy (UU PDP, Law No. 27 of 2022):** Indonesia's Personal Data
  Protection Law, effective 2024, requires explicit user consent before collecting biometric
  data via WhatsApp Flows (selfie / liveness capture). A consent screen must be the first
  screen in any identity verification Flow.
- **Meta WhatsApp Commerce Policy:** Authentication templates must be pre-approved and
  used only for their declared purpose. Sending marketing content inside an authentication
  template is a policy violation that can result in WABA suspension.

---

## 8. Key Takeaways and Decision Framework

```
Is WhatsApp penetration high in your target user segment?
  └─ Yes (urban Indonesia, age 18–55) → WhatsApp primary, SMS fallback
  └─ No (rural, elderly, feature phone) → SMS primary, WA optional

Is your platform registered in Indonesia?
  └─ Yes → Domestic authentication rate (Rp 250–357/msg)
  └─ No  → Authentication-International rate (Rp 1,940/msg) — model this carefully

Do you need KYC beyond phone number verification?
  └─ Yes → WhatsApp Flows with PhotoPicker (KTP upload + liveness)
  └─ No  → Standard authentication OTP template

Is fraud risk high (fintech, e-wallet)?
  └─ Yes → Layer: WA OTP + trusted device + PIN + SIM change detection + risk engine
  └─ No  → WA OTP + trusted device sufficient

Volume > 500K OTPs/month?
  └─ Yes → Negotiate BSP volume tier; consider direct Meta partner status
  └─ No  → Standard BSP pricing adequate


WhatsApp-based authentication is not a silver bullet — it is **one critical layer in a
defence-in-depth strategy**. The Indonesian threat landscape (SIM swap, fake BTS, APK
spyware, social engineering) requires that OTP delivery be paired with device binding,
risk scoring, and continuous fraud monitoring. When combined thoughtfully, WhatsApp's
reach, trust, E2EE security, and competitive pricing make it the **default primary
authentication channel** for any Indonesian digital platform targeting mainstream
smartphone users.

---

## Summary

**What I did:**
Ran seven targeted web searches across all five required research areas, then synthesised findings into a production-quality system-design knowledge base article.

**What I found:**
1. **Pricing (Indonesia):** Meta moved to per-message billing (July 2025). Domestic authentication = Rp 250–357/msg (~$0.019–0.022); Authentication-International (sent from outside Indonesia) = Rp 1,940/msg (~$0.12) — a ~5–8× trap for global companies.
2. **WhatsApp Flows:** Supports multi-screen in-chat KYC (ID upload via PhotoPicker, liveness/face match, form collection). One-Tap Autofill on Android eliminates manual OTP entry. Backend endpoint validation required for field regex.
3. **SMS vs WA cost (Indonesia):** Indonesian SMS OTP costs Rp 400–600 with ~70–85% delivery; WhatsApp OTP starts at Rp 357 with >98% delivery. True cost-per-delivered-OTP favours WhatsApp by ~43% despite seemingly similar headline prices.
4. **Indonesian apps:** Gojek offers 5 auth methods (WA OTP, SMS OTP, magic link, silent SIM auth, biometric/One Tap). DANA uses WhatsApp as the explicit primary verification path. OVO integrates WhatsApp for security alerts and OTP fraud warnings. All platforms print anti-sharing warnings inside every OTP message.
5. **Anti-fraud:** Key threats are SIM swap, OTP phishing, fake BTS SMS interception (mitigated by WA E2EE), malicious APK spyware, and OTP pumping. Defence patterns include: rate limiting, 6-digit TOTP with ≤5 min expiry, CAPTCHA before OTP send, SIM change age detection via telco API, device binding, and multi-layer auth stacks for fintech.

**Output:** Full markdown article (above) — ready to paste directly into the knowledge base. No files created (output delivered inline as requested).

**Issues:** One search returned a rate-limit error (429) on the first parallel batch; it was retried and succeeded in the second batch. All data requirements were ultimately fulfilled.