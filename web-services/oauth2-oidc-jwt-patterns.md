---
title: "OAuth2, OIDC, and JWT Patterns"
category: web-services
summary: "Comprehensive guide to OAuth2 flows, OpenID Connect, JWT internals, token lifecycle, and security pitfalls for modern auth architectures."
sources:
  - web-research-2026
updated: 2026-04-08
---
# OAuth2, OIDC, and JWT Patterns
> Comprehensive guide to OAuth2 flows, OpenID Connect, JWT internals, token lifecycle, and security pitfalls for modern auth architectures.

## OAuth2 Flows

OAuth2 is an authorization *framework* — it defines how a client obtains tokens to access resources on behalf of a user (or itself). Choosing the right flow depends on where code runs and who the principal is.

### Authorization Code Flow
The gold standard for server-side web apps. The browser is redirected to the authorization server, the user authenticates, and a short-lived **code** is returned. The code is exchanged server-side (never exposed in the browser) for tokens.

```
Browser → /authorize?response_type=code&client_id=...&redirect_uri=...&state=xyz
AS      → redirect to redirect_uri?code=ABC&state=xyz
Server  → POST /token { code=ABC, client_secret=... }
AS      → { access_token, refresh_token, id_token }
```

**Use when:** Server-side app with a secure back-end that can store a `client_secret`.

### Authorization Code + PKCE (Proof Key for Code Exchange)
Extends Authorization Code for **public clients** (SPAs, mobile apps) where a `client_secret` cannot be kept secret. A random `code_verifier` is hashed (`code_challenge = SHA256(verifier)`), sent with the `/authorize` request, and the raw verifier is sent at token exchange. An intercepted code is useless without the verifier.

```python
import secrets, hashlib, base64

verifier = secrets.token_urlsafe(64)
challenge = base64.urlsafe_b64encode(
    hashlib.sha256(verifier.encode()).digest()
).rstrip(b'=').decode()
# Send challenge with /authorize, verifier with /token
```

**Use when:** SPAs, mobile/native apps, or any client that cannot securely store credentials.

### Client Credentials Flow
Machine-to-machine (M2M). No user involved. The service authenticates directly with `client_id` + `client_secret` (or a signed JWT assertion) and gets an access token scoped to its own permissions.

```
Service → POST /token { grant_type=client_credentials, client_id, client_secret, scope=read:data }
AS      → { access_token, expires_in }
```

**Use when:** Background jobs, microservice-to-microservice calls, CI/CD pipelines.

### Device Code Flow (Device Authorization Grant)
For devices with limited input capability (smart TVs, CLI tools). Device displays a short code + URL; user authenticates on a separate device; device polls the token endpoint.

```
Device → POST /device/code → { device_code, user_code="ABCD-1234", verification_uri }
Device → shows user_code + URL to user, polls /token
User   → visits URL, enters ABCD-1234, logs in
Device → receives access_token on next poll
```

**Use when:** IoT devices, CLI tools (GitHub CLI, AWS CLI use this).

---

## OpenID Connect (OIDC)

OIDC is an **identity layer on top of OAuth2**. OAuth2 says nothing about *who* the user is — OIDC adds an **ID Token** (a JWT) that carries identity claims.

### Key Additions over OAuth2
| Concept | Description |
|---|---|
| `scope=openid` | Triggers OIDC; required for ID token issuance |
| **ID Token** | JWT with `sub`, `iss`, `aud`, `exp`, `iat`, and profile claims |
| **UserInfo Endpoint** | `GET /userinfo` with access token → extended claims |
| **Discovery** | `/.well-known/openid-configuration` — JSON doc with all endpoints, supported algorithms, JWKS URI |

### Discovery Document
```json
{
  "issuer": "https://auth.example.com",
  "authorization_endpoint": "https://auth.example.com/authorize",
  "token_endpoint": "https://auth.example.com/token",
  "jwks_uri": "https://auth.example.com/.well-known/jwks.json",
  "userinfo_endpoint": "https://auth.example.com/userinfo",
  "id_token_signing_alg_values_supported": ["RS256"]
}
```

**ID Token vs Access Token:** The ID token is for the **client** to verify identity. The access token is for the **resource server** to authorize requests. Never use the access token as proof of identity.

---

## JWT Internals

A JWT is `base64url(header).base64url(payload).signature`.

```json
// Header
{ "alg": "RS256", "typ": "JWT", "kid": "key-2024-01" }

// Payload
{
  "iss": "https://auth.example.com",
  "sub": "user_123",
  "aud": "api.example.com",
  "exp": 1712600000,
  "iat": 1712596400,
  "scope": "read:orders write:orders",
  "email": "alice@example.com"
}
```

### RS256 vs HS256

| | RS256 (asymmetric) | HS256 (symmetric) |
|---|---|---|
| **Signing** | Private key (held by AS only) | Shared secret |
| **Verification** | Public key (anyone can fetch from JWKS) | Requires the shared secret |
| **Key distribution** | Easy — public JWKS endpoint | Hard — secret must be shared securely |
| **Use case** | Multi-service, public verification | Single trusted service, internal only |
| **Rotation** | Rotate private key, publish new JWKS | Must re-distribute secret |

**Always prefer RS256** in distributed systems. HS256 creates a shared secret sprawl problem.

### Validation Checklist
1. Verify signature against JWKS (cache JWKS, respect `Cache-Control`, rotate on unknown `kid`)
2. Check `exp` (not expired) and `nbf` (not before, if present)
3. Validate `iss` matches expected issuer
4. Validate `aud` contains your service identifier — **never skip this**
5. Check `sub` is not empty

---

## Token Lifecycle

### Three Token Types
| Token | Lifetime | Purpose |
|---|---|---|
| **Access Token** | Short (5–60 min) | Presented to resource servers |
| **Refresh Token** | Long (days–weeks) | Exchanges for new access tokens |
| **ID Token** | Short (same as access) | Identity proof for the client |

### Refresh Token Rotation
Each time a refresh token is used, it is **invalidated** and a new one is issued. If an old refresh token is replayed (stolen token scenario), the AS detects the reuse and revokes the entire token family.

```
Client → POST /token { grant_type=refresh_token, refresh_token=RT_v1 }
AS     → { access_token=AT_v2, refresh_token=RT_v2 }  ← RT_v1 now invalid
Attacker replays RT_v1 → AS detects reuse → revokes RT_v2, forces re-login
```

**Implement rotation** for any app with meaningful session persistence.

---

## Security Pitfalls

| Pitfall | Risk | Mitigation |
|---|---|---|
| **Token in URL** | Logged in server access logs, Referer headers | Keep tokens in `Authorization` header or secure cookies only |
| **Missing `state` param** | CSRF — attacker initiates auth and binds their account | Always generate a random `state`, validate on callback |
| **Open redirect** | Attacker redirects to evil.com | Whitelist exact redirect URIs; reject anything not pre-registered |
| **Implicit flow** | Tokens in URL fragment, no refresh tokens | Deprecated — use PKCE instead |
| **Weak `client_secret`** | Brute-force | Use cryptographically random 32+ byte secrets |
| **Storing access tokens in localStorage** | XSS steals tokens | Use `HttpOnly` cookies or in-memory only |
| **Skipping `aud` validation** | Token issued for service A accepted by service B | Always validate `aud` on resource servers |
| **Long-lived access tokens** | Breach window is wide | Keep access token TTL ≤ 15 min; use refresh tokens |

---

## Practical: Which Flow to Use?

```
Is there a human logging in?
├── Yes
│   ├── Can you keep a client_secret server-side?
│   │   ├── Yes → Authorization Code Flow
│   │   └── No (SPA/mobile) → Authorization Code + PKCE
│   └── Limited input device (TV, CLI)?
│       └── Device Code Flow
└── No (M2M)
    └── Client Credentials Flow
```

### Token Storage Decision Tree
- **Server-side session app:** Store tokens in server session, session ID in `HttpOnly` cookie
- **SPA:** In-memory JS variable (not localStorage); use `HttpOnly` cookie for refresh token via BFF (Backend For Frontend) pattern
- **Mobile:** Secure enclave / Keychain / Keystore; never plain storage
