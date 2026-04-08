---
title: "Anti-Fraud Verification Patterns"
category: identity-and-verification
summary: "Fraud prevention in verification — velocity checks, device intelligence, IP reputation, synthetic identity detection, and multi-signal fraud scoring."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# Anti-Fraud Verification Patterns

> Fraud prevention in verification — velocity checks, device intelligence, IP reputation, synthetic identity detection, and multi-signal fraud scoring.

## 5.1 Velocity Checks

Velocity checks detect fraud by analyzing the *frequency and speed* of actions within time windows:

```
VELOCITY CHECK ARCHITECTURE:

Event Stream (Redis / Kafka)
         │
         ▼
┌─────────────────────────────────────────────────────┐
│                VELOCITY COUNTERS                     │
│                                                     │
│  Per IP address:                                    │
│    ip:{hash}:attempts:5m   → 5-minute window        │
│    ip:{hash}:attempts:1h   → 1-hour window          │
│    ip:{hash}:attempts:24h  → 24-hour window         │
│                                                     │
│  Per Device fingerprint:                            │
│    device:{hash}:accounts  → unique accounts seen   │
│    device:{hash}:ips       → unique IPs used        │
│                                                     │
│  Per Phone number:                                  │
│    phone:{hash}:otps:15m   → OTP requests/15 min   │
│    phone:{hash}:fails      → consecutive failures   │
│                                                     │
│  Per User ID:                                       │
│    user:{id}:transactions:1h → transaction count   │
│    user:{id}:amount:24h    → cumulative amount      │
└─────────────────────────────────────────────────────┘
         │
         ▼
DECISION RULES (examples):
  • ip_attempts_5m > 10       → CAPTCHA
  • ip_attempts_1h > 50       → Temporary block (30 min)
  • device_unique_accounts > 5 → Fraud ring flag → Manual review
  • phone_otps_15m > 3        → Block OTP, suggest email fallback
  • phone_fails_consec > 3    → Lock account, require reset
  • transaction_amount > $5K  → EDD trigger
```

**Redis implementation pattern:**
```python
import redis
import time

r = redis.Redis()

def velocity_check(key: str, limit: int, window_seconds: int) -> bool:
    """
    Returns True if action is allowed, False if rate-limited.
    Uses sliding window counter with Redis.
    """
    pipe = r.pipeline()
    now = time.time()
    window_start = now - window_seconds
    
    pipe.zremrangebyscore(key, 0, window_start)  # Remove old events
    pipe.zadd(key, {str(now): now})               # Add current event
    pipe.zcard(key)                               # Count events in window
    pipe.expire(key, window_seconds)              # Auto-expire key
    
    results = pipe.execute()
    current_count = results[2]
    
    return current_count <= limit
```

## 5.2 Device Intelligence

Device fingerprinting creates a stable identifier that persists across sessions, even when users clear cookies:

```
DEVICE FINGERPRINT COMPOSITION:

Browser/JS Signals:
  ├── Canvas fingerprint (unique rendering)
  ├── WebGL renderer hash
  ├── Audio context fingerprint
  ├── Screen resolution + color depth
  ├── Installed fonts (CSS probe)
  ├── Browser plugins/extensions list
  ├── TimeZone offset
  ├── Language settings
  ├── Do Not Track header
  └── User Agent string

Network Signals:
  ├── IP address (v4/v6)
  ├── ASN / ISP
  ├── Connection type (fiber/mobile/datacenter)
  └── TLS fingerprint (JA3/JA4 hash)

Hardware Signals (Mobile):
  ├── Device model + OS version
  ├── Battery API patterns
  ├── Gyroscope/accelerometer signatures
  ├── Secure Enclave presence
  └── Platform attestation (SafetyNet/App Attest)

FINAL FINGERPRINT:
  SHA-256(concat(canvas, webgl, audio, fonts, ...))
  = Stable ~95% accuracy across sessions
```

**Fraud signals from device intelligence:**
- **IP mismatch**: Device timezone = Jakarta, IP geolocation = Singapore → flag
- **Emulator detection**: Missing hardware sensors, perfect metrics → likely bot
- **Multiple accounts**: Same device fingerprint across 5+ user IDs → fraud ring
- **Rooted/jailbroken**: Elevated risk score for financial operations
- **Proxy/VPN**: IP is datacenter or known anonymizer

---

## 5.3 IP Reputation

```
IP REPUTATION CHECK PIPELINE:

Incoming Request IP
         │
         ├─── Lookup in local blocklist (Redis cache, sub-1ms)
         │         ├── Known Tor exit nodes
         │         ├── Known VPN endpoints
         │         └── Previously flagged fraud IPs
         │
         ├─── Query IP intelligence API (MaxMind / IPQualityScore):
         │         ├── is_proxy / is_vpn / is_hosting
         │         ├── fraud_score (0–100)
         │         ├── country_code
         │         ├── ASN organization
         │         └── abuse confidence %
         │
         └─── Correlation with device fingerprint:
                   ├── IP country ≠ device timezone → flag
                   ├── IP ASN = datacenter + new device → flag
                   └── IP rotates rapidly for same device → bot signal
```

## 5.4 Combined Fraud Signal Matrix

```
SIGNAL COMBINATION DECISION TABLE:

IP Clean + Device Known + Velocity Normal = ALLOW (no friction)
IP Clean + Device Unknown + Velocity Normal = ALLOW (soft track)
IP Proxy + Device Known + Velocity Normal  = CAPTCHA (low friction)
IP Proxy + Device Unknown + Velocity High  = BLOCK / OTP required
IP Tor   + Any Device    + Any Velocity    = BLOCK (always)
IP Clean + Device Fraud-flagged + Any      = MANUAL REVIEW
High Velocity + Any IP + Any Device        = TEMPORARY BLOCK + ALERT
```

---