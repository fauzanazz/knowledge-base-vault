---
title: "Service-to-Service Auth & mTLS"
category: web-services
summary: "Secure internal service communication using mTLS, SPIFFE workload identity, service meshes, and zero-trust principles."
sources:
  - web-research-2026
updated: 2026-04-08
---
# Service-to-Service Auth & mTLS
> Secure internal service communication using mTLS, SPIFFE workload identity, service meshes, and zero-trust principles.

## The Problem: "Trusted Network" Is a Myth

Traditional architectures assumed the internal network was safe. If a request came from inside the datacenter, it was trusted. This assumption breaks when:
- An attacker achieves lateral movement after an initial compromise.
- A misconfigured service is exposed internally.
- Multi-tenant clusters share a network namespace.
- Cloud network ACLs fail or are misconfigured.

**Zero-trust principle:** Every service must authenticate and authorize every request, regardless of network origin. "Never trust, always verify."

---

## mTLS: How It Works

Standard TLS authenticates the *server* to the *client* (the client verifies the server's certificate). **Mutual TLS (mTLS)** adds the reverse: the *server* also demands a certificate from the *client*. Both sides present X.509 certificates; both verify the other's chain of trust against a shared CA.

### TLS Handshake (mTLS variant)
```
Client                          Server
  │── ClientHello ─────────────→ │
  │← ServerHello ────────────── │
  │← Server Certificate ─────── │  (server proves identity)
  │← CertificateRequest ─────── │  ← mTLS addition
  │── Client Certificate ──────→ │  ← client proves identity
  │── CertificateVerify ───────→ │
  │── Finished ────────────────→ │
  │← Finished ─────────────────  │
  │══════ Encrypted channel ════ │
```

**What each cert proves:**
- Server cert: "I am the payments-service, signed by our internal CA."
- Client cert: "I am the orders-service, signed by our internal CA."

Authentication is cryptographic — no passwords, no API keys in headers, no network ACLs to maintain.

---

## Certificate Management

### PKI Components
```
Root CA (offline, air-gapped)
  └─ Intermediate CA (online, issues service certs)
       ├─ payments-service.internal  (leaf cert, 24h TTL)
       ├─ orders-service.internal    (leaf cert, 24h TTL)
       └─ user-service.internal      (leaf cert, 24h TTL)
```

**Why short-lived leaf certs?**
- CRL (Certificate Revocation List) and OCSP are operationally painful at scale.
- A 24-hour cert that auto-rotates effectively makes revocation moot — a compromised cert expires soon anyway.
- The intermediate CA's private key stays protected; if it's compromised, rotate and reissue all leaf certs.

### Rotation Without Downtime
1. Issue new cert before old one expires (overlap window: old_expiry - 1h).
2. Both old and new certs are valid during the window.
3. Service reloads cert from disk/memory without restarting.
4. After cutover, old cert expires naturally.

Most TLS libraries support hot-reload via `ssl.SSLContext` reload or a `SIGHUP` handler.

---

## Service Mesh: Auto-mTLS (Istio & Linkerd)

Implementing mTLS manually in every service is error-prone. Service meshes inject a sidecar proxy (Envoy for Istio, linkerd-proxy for Linkerd) that handles TLS transparently.

### Istio
```yaml
# PeerAuthentication: enforce mTLS for a namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT   # Reject any plaintext connection
```

```yaml
# AuthorizationPolicy: service-level RBAC
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-orders-to-payments
  namespace: production
spec:
  selector:
    matchLabels:
      app: payments-service
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/orders-service"]
    to:
    - operation:
        methods: ["POST"]
        paths: ["/v1/charges"]
```

App code sees plaintext on localhost; the sidecar upgrades to mTLS on the wire. Services don't change.

### Linkerd
Linkerd's auto-mTLS is on by default with zero configuration. Certificates are issued by the control plane's certificate authority (cert-manager + trust-anchor configurable). Lighter weight than Istio; fewer features.

```bash
# Verify mTLS is active for a deployment
linkerd viz edges deployment/orders-service -n production
```

---

## SPIFFE/SPIRE: Workload Identity

**SPIFFE** (Secure Production Identity Framework For Everyone) standardizes *workload identity* — a cryptographically verifiable identity for any workload, regardless of where it runs (Kubernetes, VMs, bare metal, Lambda).

**SVID (SPIFFE Verifiable Identity Document):** The identity artifact. Two forms:
- **X.509-SVID:** A certificate with a SPIFFE URI in the SAN field: `spiffe://example.org/ns/production/sa/orders-service`
- **JWT-SVID:** A JWT with the SPIFFE URI as the `sub` claim.

**SPIRE** is the reference SPIRE implementation:
```
SPIRE Server (per trust domain)
  └─ SPIRE Agent (per node, talks to server)
       └─ Workload API (Unix socket, served to each workload)
            └─ Application fetches SVID via API
```

```go
// Go: fetch X.509-SVID from SPIRE agent
client, _ := workloadapi.New(ctx, workloadapi.WithAddr("unix:///run/spire/sockets/agent.sock"))
svid, _ := client.FetchX509SVID(ctx)
// svid.Certificates contains the X.509 cert ready for mTLS
```

**Why SPIFFE over ad-hoc PKI?**
- Works across cloud providers, on-prem, and hybrid.
- Identity is tied to the *workload* (process + attestation), not to a hostname or IP.
- Federated trust domains allow cross-cluster and cross-organization authentication.
- Istio and Linkerd both support SPIFFE SVIDs natively.

---

## JWT-Based Service Auth (Alternative)

When mTLS is impractical (e.g., calling a third-party API, or a language runtime with poor TLS library support), use short-lived JWTs.

```
Service A                      Service B
  │                                │
  │── POST /resource ─────────────→│
  │   Authorization: Bearer <JWT>   │
  │                                │
  │                        Validate JWT:
  │                        - Verify signature (shared secret or public key)
  │                        - Check `iss` (issuer = "service-a")
  │                        - Check `aud` (audience = "service-b")
  │                        - Check `exp` (expiry < 5 min)
```

**JWT claims for service-to-service:**
```json
{
  "iss": "spiffe://example.org/ns/prod/sa/orders-service",
  "sub": "orders-service",
  "aud": "payments-service",
  "iat": 1712600460,
  "exp": 1712600760,
  "jti": "unique-nonce-to-prevent-replay"
}
```

Sign with RS256 or ES256. Rotate signing keys using JWKS endpoint (`/.well-known/jwks.json`). Short expiry (5–15 min) limits blast radius if a token is intercepted.

---

## Zero-Trust Networking Principles

1. **Authenticate every request** — never assume a source IP is trusted.
2. **Least privilege** — service A can only call the specific endpoints of service B it needs.
3. **Encrypt in transit** — all internal traffic encrypted (mTLS or TLS + JWT).
4. **Short-lived credentials** — prefer 24h certs over long-lived API keys.
5. **Audit everything** — log who called what, when. Service mesh provides this automatically.
6. **Verify workload integrity** — SPIRE attestation can check Kubernetes service account tokens, TPM measurements, etc.

**Network perimeter is still useful** (VPCs, network policies) as *defense in depth*, but it cannot be the *only* security layer.

---

## Practical: cert-manager + Kubernetes

`cert-manager` automates issuance and rotation of TLS certificates in Kubernetes.

```yaml
# 1. Create a ClusterIssuer backed by an internal CA
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: internal-ca
spec:
  ca:
    secretName: internal-ca-key-pair  # Secret containing CA cert + key

# 2. Request a certificate for a service
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: payments-service-tls
  namespace: production
spec:
  secretName: payments-service-tls
  issuerRef:
    name: internal-ca
    kind: ClusterIssuer
  duration: 24h
  renewBefore: 1h
  dnsNames:
    - payments-service.production.svc.cluster.local
  uris:
    - spiffe://cluster.local/ns/production/sa/payments-service
```

cert-manager automatically rotates the certificate 1 hour before expiry. Mount the secret as a volume; your app reloads on file change.

```yaml
# Mount in deployment
volumes:
  - name: tls
    secret:
      secretName: payments-service-tls
volumeMounts:
  - name: tls
    mountPath: /etc/tls
    readOnly: true
```

---

## Trade-offs: mTLS vs API Keys vs JWT

| Dimension | mTLS | API Keys | JWT (short-lived) |
|-----------|------|----------|-------------------|
| Mutual auth | ✅ Both sides | ❌ Server only | ❌ Server only |
| Key rotation | Auto (cert-manager/SPIRE) | Manual, error-prone | Auto (JWKS rotation) |
| Revocation | Short TTL (hours) | Requires key invalidation | Short TTL (minutes) |
| Operational complexity | High (PKI, mesh) | Low | Medium |
| Works without service mesh | Yes (but manual) | Yes | Yes |
| Cross-org / third-party | Possible (SPIFFE federation) | Common | Common (OIDC) |
| Performance overhead | ~1ms TLS handshake (cached) | Negligible | JWT validation ~0.1ms |
| Debugging | Harder (cert issues cryptic) | Easy | Medium |

**Decision guide:**
- **Kubernetes-native, internal services:** Service mesh auto-mTLS (Istio/Linkerd). Zero code change, automatic.
- **Cross-cluster or cross-cloud:** SPIFFE/SPIRE + X.509-SVIDs.
- **Calling external APIs or third parties:** JWT (OIDC) or API keys with secret manager.
- **Legacy services you can't change:** API keys or network-level controls as stopgap, then migrate.
- **Never:** Long-lived API keys stored in environment variables without rotation or audit.

---

## Common Pitfalls

- **Certificate pinning in internal services** — breaks on rotation. Use CA trust instead.
- **Not checking `aud` claim in JWTs** — allows token replay across services.
- **CRL/OCSP without short TTLs** — hard to operate at scale; prefer short-lived certs.
- **Permissive `PERMISSIVE` mTLS mode left in production** — Istio's default; change to `STRICT` before go-live.
- **Using the same CA for internal and external certs** — compartmentalize trust domains.
