---
title: "CAPTCHA & Bot Detection"
category: identity-and-verification
summary: "Human verification and bot prevention — reCAPTCHA, hCaptcha, Turnstile, behavioral analysis, proof-of-work challenges, and invisible verification."
sources:
  - research/user-verification-methods-2026
updated: 2026-04-08T18:00:00.000Z
---

# CAPTCHA & Bot Detection

> Human verification and bot prevention — reCAPTCHA, hCaptcha, Turnstile, behavioral analysis, proof-of-work challenges, and invisible verification.

### How It Works Technically

**reCAPTCHA v2** ("I am not a robot"):
- Behavioral analysis: Mouse movements, typing patterns, scroll behavior collected before checkbox click
- Low-risk users: Checkbox only
- High-risk users: Image challenge (traffic signs, crosswalks, fire hydrants)
- Backend verification: POST token to `www.google.com/recaptcha/api/siteverify`
- Weakness: ML image classifiers can now solve challenges; farms of human solvers

**reCAPTCHA v3** (invisible, risk-score only):
- Continuous JavaScript behavioral monitoring
- Returns score 0.0–1.0 (0 = likely bot, 1 = likely human)
- No user interaction unless score low
- Developer sets threshold; actions taken based on score
- Privacy concern: Requires loading Google's tracking script

**hCaptcha**:
- Similar UI to reCAPTCHA v2 but privacy-focused (no Google tracking)
- Revenue sharing model: Publishers earn from AI training data labeling
- Enterprise: Behavioral signals + ML, GDPR-compliant option
- Used by: Cloudflare (formerly), Discord, Epic Games

**Cloudflare Turnstile**:
- Privacy-preserving alternative
- Proves human presence without visual challenges in most cases
- Uses browser signals, cryptographic attestation
- No tracking pixels; GDPR-friendly
- Free for most usage

**Arkose Labs (MatchKey)**:
- Gamified challenges (match key images) — resistant to both bots and human farms
- Fraud farm detection: Challenges designed to be uneconomical for human click-farms
- Behavioral biometrics + device signals + 225+ risk attributes
- Not CAPTCHA-only: Full bot management platform
- Used by: GitHub, Snapchat, PayPal, Riot Games

**DataDome / PerimeterX / Imperva**:
- Server-side bot detection at reverse proxy level
- Real-time ML analysis of every HTTP request
- Signals: User-Agent analysis, TLS fingerprinting, HTTP header order, IP reputation, behavioral velocity
- No user friction for legitimate users

**Bot Detection Signal Stack**:
```
Layer 1: IP/Network signals
  - IP reputation (abuse databases)
  - VPN/proxy/Tor detection  
  - ASN analysis (datacenter vs. residential)
  - Geolocation consistency

Layer 2: Device/Browser signals
  - User-agent parsing + consistency check
  - TLS fingerprint (JA3/JA4 hash)
  - HTTP/2 fingerprint
  - Browser API behavior (headless detection)
  - Canvas/WebGL fingerprint

Layer 3: Behavioral signals
  - Mouse movement trajectory (humans = curved; bots = straight)
  - Typing cadence (bursts vs. natural rhythm)
  - Time-on-page before submission
  - Scroll patterns
  - Click precision (humans misclick slightly)
  - Form field interaction order

Layer 4: Session/History signals
  - Account age
  - Transaction history
  - Cross-session device consistency
  - Rate/velocity patterns
```

### Common Attack Vectors

| Attack | Mitigation |
|--------|------------|
| **ML image classification** | Obfuscate image generation, use novel challenge types |
| **Human click farms** | Economic deterrence (time-consuming challenges), behavioral biometrics |
| **Bot libraries** (Puppeteer, Playwright, Selenium) | Headless browser detection (missing APIs, timing) |
| **Residential proxy rotation** | Velocity-based detection, behavioral analysis |
| **CAPTCHA solving services** | Challenge rotation, gamified challenges |

### Real-World Examples

- **Google**: reCAPTCHA v3 on all products
- **PayPal**: Arkose Labs (switched from reCAPTCHA)
- **GitHub**: Arkose Labs for account creation
- **Cloudflare**: Turnstile for protected pages
- **Tokopedia/Shopee**: hCaptcha or custom bot detection

---