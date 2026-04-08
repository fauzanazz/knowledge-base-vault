---
title: "Edge Computing"
category: fullstack-patterns
summary: "Running compute logic at CDN edge nodes geographically close to users, reducing latency by eliminating origin round-trips for dynamic logic, personalization, and API responses."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Edge Computing

> Running compute logic at CDN edge nodes geographically close to users, reducing latency by eliminating origin round-trips for dynamic logic, personalization, and API responses.

## What It Is

Edge computing moves server-side logic from centralized origins to distributed PoPs (Points of Presence) operated by CDN providers. Instead of a user in Tokyo hitting your US-East origin (150ms RTT), they hit a Tokyo edge node (5ms) and computation happens there.

**Edge runtime constraints** (compared to Node.js):
- No filesystem access
- Limited CPU time per request (1–50ms)
- No Node.js built-ins (no `fs`, `child_process`, `net`)
- V8 isolates, not full VMs — cold starts measured in microseconds
- Memory limits: 128MB–512MB depending on platform

## Cloudflare Workers

The most widely adopted edge compute platform (300+ PoPs):

```typescript
// Cloudflare Worker — runs globally, <1ms cold start
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    
    // A/B testing at edge — no origin needed
    const userId = request.headers.get('cf-ray') ?? crypto.randomUUID();
    const variant = parseInt(userId.slice(-1), 16) < 8 ? 'control' : 'treatment';
    
    // Modify request to origin
    const originUrl = new URL(request.url);
    originUrl.hostname = 'api.origin.example.com';
    
    const response = await fetch(originUrl, {
      headers: { ...Object.fromEntries(request.headers), 'x-variant': variant }
    });
    
    // Mutate response at edge
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('x-edge-variant', variant);
    return newResponse;
  }
};
```

**Cloudflare KV**: eventually consistent key-value store accessible from Workers. Good for feature flags, user sessions, config.

**Durable Objects**: single-threaded stateful objects with guaranteed consistency — the solution to coordination at the edge (e.g., rate limiting, WebSocket rooms).

```typescript
// Durable Object for per-user rate limiting
export class RateLimiter {
  private requests = 0;
  
  async fetch(request: Request) {
    this.requests++;
    if (this.requests > 100) {
      return new Response('Rate limited', { status: 429 });
    }
    return new Response('OK');
  }
}
```

## Deno Deploy

Deno Deploy runs Deno (TypeScript-native) code at the edge. Key differentiators:
- TypeScript first-class (no transpilation step)
- Web-standard APIs only (Fetch, WebStreams, WebCrypto)
- Free tier generous; integrates with Deno KV (globally distributed SQLite-like)

```typescript
// Deno Deploy handler
Deno.serve(async (req) => {
  const kv = await Deno.openKv();
  const count = await kv.get<number>(['visits']);
  await kv.set(['visits'], (count.value ?? 0) + 1);
  return new Response(`Visits: ${count.value}`);
});
```

## Vercel Edge Functions / Middleware

Next.js middleware runs on Vercel's edge network (powered by Cloudflare):

```typescript
// middleware.ts — runs on every request, before routing
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Geolocation-based routing
  const country = request.geo?.country ?? 'US';
  if (country === 'EU') {
    return NextResponse.rewrite(new URL('/eu' + request.nextUrl.pathname, request.url));
  }
  
  // Auth check at edge — redirect before page loads
  const token = request.cookies.get('session')?.value;
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

## Common Edge Use Cases

### 1. Authentication / Authorization
Check JWT validity at edge → reject unauthenticated requests before hitting origin. **Saves origin cost and reduces latency.**

### 2. Personalization
Serve different content based on geo, device, user segment without origin:
```typescript
const country = request.cf?.country;
const html = await getLocalizedHtml(country); // from KV/cache
```

### 3. Request/Response Transformation
Add security headers, transform API responses, inject analytics snippets:
```typescript
response.headers.set('Strict-Transport-Security', 'max-age=31536000');
response.headers.set('X-Content-Type-Options', 'nosniff');
```

### 4. Rate Limiting
Per-IP or per-user rate limits using Durable Objects or Cloudflare's built-in rules.

### 5. Cache Warming / Stale-While-Revalidate
```typescript
const cached = await cache.match(request);
if (cached) {
  // Return stale, revalidate in background
  ctx.waitUntil(revalidate(request));
  return cached;
}
```

## Latency Impact

| Scenario | Without Edge | With Edge |
|----------|-------------|-----------|
| Static asset | 80ms (CDN) | 5ms (edge cache) |
| Auth check | 200ms (origin) | 8ms (edge JWT verify) |
| API (no DB) | 150ms (origin) | 15ms (edge) |
| API (requires DB) | 150ms | 150ms+ (DB call from edge) |

**Key insight**: edge compute helps most for logic that doesn't require database calls. For DB-dependent responses, you're still bound by DB latency — use regional databases (PlanetScale, Neon, Turso) co-located with edge PoPs.

## Trade-offs

| Pro | Con |
|-----|-----|
| Sub-10ms for eligible logic | No filesystem, limited Node.js APIs |
| Scales to zero cost | Debugging is harder |
| Global by default | Cold starts still possible (isolate recycling) |
| Reduces origin load | Stateful logic requires Durable Objects |
| V8 isolates = security boundary | Database latency still applies |

## When to Use

✅ Auth/session checks on every request  
✅ Geo-based routing or personalization  
✅ Request/response header manipulation  
✅ Rate limiting and bot protection  
✅ A/B testing flag evaluation  
❌ CPU-intensive operations (video transcoding, ML inference)  
❌ Heavy database-dependent APIs (edge + remote DB = worse latency)  
❌ Workloads needing Node.js ecosystem (use Lambda or containers)

---
*Related: [[WebSocket Architecture]], [[Feature Flags]], [[Observability Stack]]*
