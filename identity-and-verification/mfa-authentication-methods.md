---
title: "Multi-Factor Authentication Methods"
category: identity-and-verification
summary: "MFA/2FA implementation — TOTP, HOTP, WebAuthn/FIDO2, push notifications, hardware keys, and phishing-resistant authentication methods."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Multi-Factor Authentication Methods

> MFA/2FA implementation — TOTP, HOTP, WebAuthn/FIDO2, push notifications, hardware keys, and phishing-resistant authentication methods.

### How It Works Technically

**Authentication Factors**:
- **Something you know**: Password, PIN, security questions
- **Something you have**: Phone (OTP), hardware token, passkey
- **Something you are**: Biometrics (face, fingerprint, voice)
- **Somewhere you are**: Location-based (IP/GPS geofencing)
- **Something you do**: Behavioral biometrics

**TOTP (Time-based One-Time Password)** — RFC 6238:
```
K = Shared secret (typically 20 bytes, base32-encoded)
T = floor(Unix timestamp / 30)  // 30-second window
TOTP = HOTP(K, T)
HOTP = HMAC-SHA1(K, T) → 6-8 digit code
```
- Apps: Google Authenticator, Microsoft Authenticator, Authy, 1Password
- No network required after setup; highly phishing-resistant
- Vulnerable to real-time phishing (MITM captures and replays OTP before expiry)

**FIDO2 / WebAuthn / Passkeys** (RFC 8471, W3C WebAuthn Level 3):
```javascript
// Registration:
const publicKeyCredentialCreationOptions = {
  challenge: server_challenge,
  rp: { id: "example.com", name: "Example App" },
  user: { id: user_id, name: user_email, displayName: user_name },
  pubKeyCredParams: [{ type: "public-key", alg: -7 }],  // ES256
  authenticatorSelection: {
    residentKey: "preferred",
    userVerification: "preferred"
  }
};
const credential = await navigator.credentials.create({
  publicKey: publicKeyCredentialCreationOptions
});
// Send credential.id + attestation to server

// Authentication:
const assertion = await navigator.credentials.get({
  publicKey: { challenge: server_challenge, rpId: "example.com" }
});
// Verify assertion signature with stored public key
```

- **Private key**: Stored in device's Secure Enclave/TPM; never leaves device
- **Public key**: Stored on server
- **Phishing resistance**: Credential bound to exact origin (RP ID); works only on legitimate site
- **Passkeys**: Synced FIDO2 credentials across devices via Apple iCloud Keychain, Google Password Manager, 1Password
- **Device attestation**: Optional cryptographic proof of authenticator type (not all deployments use)

**FIDO2 Authentication Flow**:
```
1. Server sends challenge
2. Authenticator (device biometric/PIN) verifies user locally
3. Signs challenge + authenticatorData with private key
4. Server verifies signature with stored public key
5. Checks: rpIdHash, flags (UP, UV bits), counter, origin
```

**Hardware Security Keys** (YubiKey, Google Titan):
- Physical USB/NFC/Bluetooth device
- FIDO2/U2F/TOTP/PGP support
- Tamper-resistant hardware
- Most phishing-resistant MFA available
- Required for Google's "Advanced Protection Program"

**Push Notifications MFA**:
- Duo Security, Microsoft Authenticator, Okta Verify
- Server sends push to registered device; user taps Approve/Deny
- Vulnerable to "MFA fatigue" attacks (spamming pushes until user accidentally approves)
- Mitigations: Number matching (display number on both screens), geographic verification

**SMS OTP as MFA** (legacy):
- NIST SP 800-63B: "Restricted" authenticator — no longer recommended as strong MFA
- Acceptable for AAL1; insufficient for AAL2/3
- SIM swap attack vulnerability

### MFA Assurance Levels (NIST SP 800-63B)

| Level | Requirements | Examples |
|-------|-------------|----------|
| **AAL1** | Single factor | Password only |
| **AAL2** | Two factors; at least one must be a "secure" authenticator | Password + TOTP; Password + FIDO2 |
| **AAL3** | Hardware authenticator + verifier impersonation resistance | FIDO2 hardware key + PIN |

### Real-World MFA Implementations

- **Google Workspace**: FIDO2 passkeys, TOTP, SMS (discouraged), hardware keys
- **GitHub**: TOTP mandatory for npm publishers; passkeys supported
- **Binance**: TOTP (Google Authenticator) + device binding; hardware key for VIP
- **Banking**: App-based approval + TOTP; some use biometric step-up
- **PayPal**: TOTP + SMS fallback; passkey support added 2023

---