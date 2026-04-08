---
title: "TLS Security"
category: security
summary: "TLS (Transport Layer Security) provides encryption, authentication, and integrity for application-layer protocols through asymmetric and symmetric encryption, digital certificates, and secure hash functions."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.258Z
---

# TLS Security

> TLS (Transport Layer Security) provides encryption, authentication, and integrity for application-layer protocols through asymmetric and symmetric encryption, digital certificates, and secure hash functions.

# TLS Security

TLS (Transport Layer Security) secures network communication by providing three critical security properties: encryption, authentication, and integrity for application-layer protocols like HTTP.

## Encryption

TLS uses a hybrid encryption approach:
1. **Asymmetric encryption** during handshake to negotiate a shared secret
2. **Symmetric encryption** for ongoing communication using the shared secret
3. **Key rotation** periodically renegotiates secrets to mitigate decryption risks

This approach combines the security of asymmetric encryption with the performance of symmetric encryption.

## Authentication

Authentication prevents man-in-the-middle attacks through:
- **Digital signatures**: Servers sign data with private keys that clients verify using public keys
- **Certificates**: Documents containing server details, public keys, and certificate authority signatures
- **Certificate chains**: Hierarchical trust model ending with well-known root certificate authorities

Browsers maintain lists of trusted root CAs to validate certificate chains. Servers typically send complete certificate chains to avoid additional network calls.

## Integrity

Message integrity prevents tampering through Hash-based Message Authentication Codes (HMACs):
- Each message includes a hash of its payload
- Recipients verify integrity by comparing computed hashes with transmitted hashes
- This also detects data corruption beyond TCP's checksum capabilities

## TLS Handshake

Connection establishment involves negotiating:
- Cipher suites (encryption, signature, and HMAC algorithms)
- Shared secrets through key-exchange algorithms
- Certificate verification for server (and optionally client) authentication

TLS connection overhead emphasizes the importance of server proximity and connection reuse for performance optimization.

---
*Related: [[Network Communication]], [[Digital Certificates]], [[Encryption]], [[Authentication]], [[Network Security]]*
