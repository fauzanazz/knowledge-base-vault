---
title: "Verification Data Storage & Privacy"
category: identity-and-verification
summary: "Data handling for verification — what to store vs hash vs delete, biometric data regulations, retention schedules, and privacy-by-design patterns."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Verification Data Storage & Privacy

> Data handling for verification — what to store vs hash vs delete, biometric data regulations, retention schedules, and privacy-by-design patterns.

## 6.1 What to Store, What to Hash, What to Delete

```
┌────────────────────────────────────────────────────────────┐
│             DATA CLASSIFICATION FRAMEWORK                  │
└────────────────────────────────────────────────────────────┘

STORE PERMANENTLY (encrypted, access-controlled):
  ├── Verification decision record (pass/fail/timestamp)
  ├── Risk score at time of verification
  ├── Verification method used (doc type, check type)
  ├── Compliance audit log (who verified what, when)
  └── Minimal user reference (user_id only, no PII)

STORE TEMPORARILY (auto-delete after retention period):
  ├── Document images (typically 30–90 days post-verification)
  ├── Selfie/liveness video (30 days or as legally required)
  ├── OCR extracted data (name, DOB, ID number)
  └── Verification session data

STORE AS HASH ONLY (never store plaintext):
  ├── National ID number → SHA-256("ID_SALT" + id_number)
  ├── Tax ID / NPWP → HMAC-SHA256(key, npwp_number)
  ├── OTP codes → bcrypt(otp_code, 10 rounds)
  └── Phone number → HMAC-SHA256(key, e164_format)
  
  (Allow: "did user X use this ID?" check without exposing the ID)

NEVER STORE:
  ├── Raw biometric templates (store encrypted FaceMap refs only)
  ├── Full document scans longer than legally required
  ├── OTP codes in plaintext
  └── Raw credit card / financial account numbers

DELETE IMMEDIATELY:
  ├── Session tokens after TTL
  ├── OTP codes after verification or expiry
  └── Temporary image buffers after processing
```

## 6.2 Data Retention Schedule (KYC/AML Compliant)

| Data Type | Retention Period | Basis |
|---|---|---|
| Verification decision + audit log | 5–10 years | AML regulatory obligation |
| Document images (passed KYC) | 5 years after relationship ends | FATF / local AML law |
| Document images (failed KYC) | 6 months | Fraud investigation purposes |
| Biometric templates (FaceMap) | Duration of account + 90 days | Minimum needed |
| OTP codes (hashed) | 24 hours | Security + fraud investigation |
| Session logs | 12 months | Security investigation |
| Transaction records | 5–7 years | AML/tax law |
| Deleted account PII | 30 days (then purge) | Grace period for disputes |

**Indonesian context (PPDP Law):** Indonesia's Personal Data Protection Law (UU PDP, effective 2024) requires explicit consent, purpose limitation, and gives users the right to erasure. Data must be stored in servers physically located within Indonesia for "strategic" data categories.

## 6.3 Storage Architecture

```
┌────────────────────────────────────────────────────────────┐
│              VERIFICATION DATA STORAGE TIERS               │
└────────────────────────────────────────────────────────────┘

HOT TIER (active verification sessions):
  Redis / DynamoDB
  • Session state
  • OTP hash + expiry
  • Velocity counters
  TTL: 10 min – 24 hours

WARM TIER (recent verifications):
  PostgreSQL / Aurora (encrypted at rest, AES-256)
  • Verification records
  • Risk scores
  • Extracted metadata (hashed)
  • Audit logs
  Retention: Weeks to months depending on policy

COLD TIER (long-term compliance):
  S3 / GCS with SSE-KMS (AES-256 + customer-managed keys)
  • Document image archives
  • Audit log exports
  • Compliance reports
  Retention: 5–10 years per AML requirements
  Access: Only compliance/legal team with MFA + audit trail

PURGE PIPELINE:
  Automated job runs nightly:
  1. Query records past retention expiry
  2. Anonymize PII fields (replace with UUID tokens)
  3. Secure-delete document images (overwrite + delete)
  4. Remove from search indices (Elasticsearch)
  5. Purge from backup snapshots (after backup rotation completes)
  6. Log deletion event to immutable audit log
```

## 6.4 Privacy-by-Design Principles

1. **Data minimization**: Extract only needed fields from documents; don't store the whole image longer than necessary
2. **Purpose limitation**: Verification data cannot be used for marketing or analytics
3. **Pseudonymization**: Replace PII with tokens in analytics pipelines
4. **Encryption everywhere**: AES-256 at rest, TLS 1.3 in transit, customer-managed KMS keys
5. **Access controls**: RBAC — only compliance officers can see raw data; developers see only anonymized records
6. **Right to erasure**: Technical capability to delete all PII for a user on request (subject to AML hold requirements)
7. **Data locality**: Store user data in-region (Indonesia requires Indonesian data centers for regulated entities)

---