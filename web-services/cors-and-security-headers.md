---
title: "CORS and Security Headers"
category: web-services
summary: "Complete guide to CORS mechanics, preflight requests, security headers (CSP, HSTS, SRI), and hardened nginx/Caddy/Express configurations."
sources:
  - web-research-2026
updated: 2026-04-08
---
# CORS and Security Headers
> Complete guide to CORS mechanics, preflight requests, security headers (CSP, HSTS, SRI), and hardened nginx/Caddy/Express configurations.

Security headers and CORS are among the most misconfigured aspects of web services. This article covers the mechanics, correct configurations, and the pitfalls that either break legitimate usage or silently expose APIs.

---

## CORS: How It Actually Works

**Same-Origin Policy (SOP):** Browsers block cross-origin reads by default. CORS is the mechanism to explicitly relax this for trusted origins.

An **origin** is: `scheme + host + port`. `https://app.example.com` and `https://api.example.com` are **different origins**.

CORS is **enforced by the browser**, not the server. The server just sends headers; a non-browser client (curl, Postman, server-to-server) ignores CORS entirely. CORS is not a security mechanism against server-side attacks — it protects users' browsers from malicious pages making cross-origin requests on their behalf.

### Simple Requests (No Preflight)
Conditions: method is GET/HEAD/POST, headers are only the "safe" list (Accept, Content-Type limited to `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`), no custom headers.

```
Browser → GET https://api.example.com/data
          Origin: https://app.example.com

Server  → 200 OK
          Access-Control-Allow-Origin: https://app.example.com
```

If the server doesn't return `Access-Control-Allow-Origin` matching the request's `Origin`, the browser blocks the response (the request still reaches the server — the *response* is blocked).

### Preflighted Requests
Triggered when: custom headers (`Authorization`, `X-API-Key`, `Content-Type: application/json`), methods other than GET/HEAD/POST, or `ReadableStream` body.

```
Browser → OPTIONS https://api.example.com/data
          Origin: https://app.example.com
          Access-Control-Request-Method: POST
          Access-Control-Request-Headers: Content-Type, Authorization

Server  → 204 No Content
          Access-Control-Allow-Origin: https://app.example.com
          Access-Control-Allow-Methods: GET, POST, PUT, DELETE
          Access-Control-Allow-Headers: Content-Type, Authorization
          Access-Control-Max-Age: 86400   ← cache preflight for 1 day

Browser → POST https://api.example.com/data   (actual request)
          Origin: https://app.example.com
          Authorization: Bearer ...
```

**`Access-Control-Max-Age`** is important for performance. Without it, every `fetch()` call with custom headers fires a preflight OPTIONS request. Chrome caps at 7200s, Firefox at 86400s.

### Credentials Mode
By default, cross-origin requests don't send cookies or HTTP auth. To include them:

```javascript
fetch('https://api.example.com/data', {
  credentials: 'include'  // sends cookies
})
```

Server must respond with:
```
Access-Control-Allow-Origin: https://app.example.com   ← cannot be wildcard *
Access-Control-Allow-Credentials: true
```

**Critical rule:** When `credentials: include`, the server **cannot** use `Access-Control-Allow-Origin: *`. Must echo the specific requesting origin (after validation).

### Wildcard Restrictions Summary

| Situation | `*` Allowed? |
|-----------|-------------|
| Simple request, no credentials | ✅ Yes |
| `credentials: include` | ❌ No — must be specific origin |
| `Access-Control-Allow-Headers` with credentials | ❌ No |
| `Access-Control-Expose-Headers` | ✅ Yes (in modern browsers) |

---

## Correct CORS Implementation

### Validate the Origin
Never blindly echo back `Access-Control-Allow-Origin: <whatever-Origin-header-says>`. Validate against an allowlist:

```python
ALLOWED_ORIGINS = {
    "https://app.example.com",
    "https://admin.example.com",
    "http://localhost:3000",  # dev only — gate behind env flag
}

def cors_headers(request, response):
    origin = request.headers.get("Origin")
    if origin in ALLOWED_ORIGINS:
        response.headers["Access-Control-Allow-Origin"] = origin
        response.headers["Vary"] = "Origin"  # ← critical for caching
    # If origin not in allowlist: don't set the header. Browser will block.
```

**`Vary: Origin`** is mandatory when serving different CORS headers per origin. Without it, a CDN may cache a response with `ACAO: https://app.example.com` and serve it to `https://evil.com`, where the browser will (correctly) block it — but the CDN wasted the cache.

---

## Security Headers

### Content-Security-Policy (CSP)
Tells the browser which sources are allowed to load resources. The single most powerful XSS mitigation.

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
```

**Directives to know:**
- `default-src 'self'` — fallback for unspecified resource types.
- `script-src 'nonce-{random}'` — better than `'unsafe-inline'`; use nonces for inline scripts.
- `frame-ancestors 'none'` — supersedes `X-Frame-Options: DENY`.
- `upgrade-insecure-requests` — browser upgrades `http://` references to `https://`.
- `report-uri /csp-report` or `report-to` — collect violations before enforcing.

**Deployment strategy:** Start with `Content-Security-Policy-Report-Only` in monitoring mode. Fix violations. Then switch to enforcement.

### HTTP Strict Transport Security (HSTS)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
- `max-age`: how long browsers remember to use HTTPS (1 year standard).
- `includeSubDomains`: applies to all subdomains — ensure they're all HTTPS first.
- `preload`: submit to browser preload lists (hardcoded HTTPS-only in Chrome/Firefox). **Irreversible in the short term** — don't add `preload` until fully committed.

### X-Frame-Options (Legacy — use CSP `frame-ancestors`)
```
X-Frame-Options: DENY         # never framed
X-Frame-Options: SAMEORIGIN   # only same origin can frame
```
Superseded by CSP `frame-ancestors`, but keep for older browsers.

### X-Content-Type-Options
```
X-Content-Type-Options: nosniff
```
Prevents browsers from MIME-sniffing responses. Without this, browsers may execute a response served as `text/plain` as JavaScript if it looks like JS. Always set this.

### Referrer-Policy
```
Referrer-Policy: strict-origin-when-cross-origin
```
Controls how much referrer information is sent with requests:
- `no-referrer` — never send referrer.
- `strict-origin-when-cross-origin` — full URL within same origin, only origin for cross-origin (recommended default).
- `no-referrer-when-downgrade` — old default; leaks full URL on HTTPS→HTTP.

### Permissions-Policy (formerly Feature-Policy)
Controls access to browser APIs:
```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(self)
```
`()` = disabled for all origins. `(self)` = only allowed from same origin.

---

## Subresource Integrity (SRI)

Ensures CDN-served files haven't been tampered with:

```html
<script
  src="https://cdn.example.com/jquery-3.7.1.min.js"
  integrity="sha384-1H217gwSVyLSIfaLxHbE7dRb3v4mYCKbpQvzx0cegeju1MVsGrX5xXxAvs/HgeFs"
  crossorigin="anonymous">
</script>
```

Browser fetches the resource, hashes it, compares to `integrity` attribute. Blocks load if mismatch.

**Generate the hash:**
```bash
curl -s https://cdn.example.com/jquery-3.7.1.min.js | openssl dgst -sha384 -binary | openssl base64 -A
```

**Trade-off:** Breaks automatically when CDN updates the file (good! forces you to verify). Add `crossorigin="anonymous"` for cross-origin resources even without credentials — required for SRI checking.

---

## Security Headers Scoring

[securityheaders.com](https://securityheaders.com) grades headers A+ to F. Target A+.

| Header | A+ Required? | Notes |
|--------|-------------|-------|
| CSP | ✅ | Must have meaningful policy |
| HSTS | ✅ | With `includeSubDomains` and `preload` |
| X-Frame-Options | ✅ | Or CSP `frame-ancestors` |
| X-Content-Type-Options | ✅ | `nosniff` |
| Referrer-Policy | ✅ | Any value |
| Permissions-Policy | ✅ | Any value |

---

## Practical Configurations

### nginx
```nginx
# Security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
add_header Content-Security-Policy "default-src 'self'; frame-ancestors 'none'; upgrade-insecure-requests;" always;

# CORS (dynamic origin validation)
map $http_origin $cors_origin {
    default "";
    "https://app.example.com"   $http_origin;
    "https://admin.example.com" $http_origin;
}

server {
    location /api/ {
        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin $cors_origin always;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
            add_header Access-Control-Allow-Headers "Authorization, Content-Type" always;
            add_header Access-Control-Max-Age 86400 always;
            add_header Vary Origin always;
            return 204;
        }
        add_header Access-Control-Allow-Origin $cors_origin always;
        add_header Vary Origin always;
        proxy_pass http://backend;
    }
}
```

### Caddy
```caddy
example.com {
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        Referrer-Policy "strict-origin-when-cross-origin"
        Permissions-Policy "camera=(), microphone=()"
        Content-Security-Policy "default-src 'self'; frame-ancestors 'none';"
        -Server  # remove Server header
    }

    @cors_preflight {
        method OPTIONS
        header Origin *
    }
    handle @cors_preflight {
        header {
            Access-Control-Allow-Origin "{header.origin}"
            Access-Control-Allow-Methods "GET, POST, PUT, DELETE"
            Access-Control-Allow-Headers "Authorization, Content-Type"
            Access-Control-Max-Age "86400"
            Vary Origin
        }
        respond "" 204
    }
}
```

### Express (Node.js)
```javascript
const helmet = require('helmet');
const cors = require('cors');

const ALLOWED_ORIGINS = ['https://app.example.com', 'http://localhost:3000'];

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
}));

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (server-to-server, curl)
    if (!origin || ALLOWED_ORIGINS.includes(origin)) {
      callback(null, origin || false);
    } else {
      callback(new Error(`CORS: ${origin} not allowed`));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400,
}));
```

---

## Common CORS Pitfalls

| Pitfall | Impact | Fix |
|---------|--------|-----|
| `Access-Control-Allow-Origin: *` with credentials | Browser blocks all credentialed requests | Echo specific origin, add `ACAO-Credentials: true` |
| Reflecting any `Origin` without validation | Any domain can make credentialed requests | Validate against allowlist |
| Missing `Vary: Origin` | CDN caches wrong origin, causes stale CORS failures | Always add `Vary: Origin` |
| Forgetting `ACAO` on error responses (4xx, 5xx) | Browser blocks error response, client sees opaque network error | Add CORS headers to error responses too (`always` in nginx) |
| OPTIONS requests reaching app auth middleware | 401/403 on preflight before browser sends real request | Bypass auth for `OPTIONS` method |
| Regex origin matching (`.*\.example\.com`) | `evil-example.com` matches | Use exact-match allowlist, not regex |
| CORS config in load balancer + app | Duplicate headers (`ACAO` appears twice) — browser rejects | Pick one layer; strip duplicates |
| `null` origin | `file://` pages and sandboxed iframes send `null` — allowlisting `null` opens attack surface | Never allowlist `null` |

---

## Checklist

```
CORS
☐ Exact-match origin allowlist (not regex, not wildcard with credentials)
☐ Vary: Origin on all CORS responses
☐ CORS headers on error responses
☐ OPTIONS bypass auth middleware
☐ Access-Control-Max-Age set (reduce preflight overhead)

Security Headers
☐ HSTS with long max-age (start without preload, add after confirmed)
☐ CSP in report-only first, then enforce
☐ X-Content-Type-Options: nosniff
☐ Referrer-Policy set
☐ Permissions-Policy set
☐ Server/X-Powered-By headers removed
☐ SRI on third-party scripts/styles
☐ Score on securityheaders.com: A or A+
```
