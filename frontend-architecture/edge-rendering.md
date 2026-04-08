---
title: "Edge Rendering"
category: frontend-architecture
summary: "Executing rendering logic at CDN edge nodes geographically close to users, reducing latency by eliminating roundtrips to origin servers for server-side rendering, personalization, and middleware logic."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Edge Rendering

> Executing rendering logic at CDN edge nodes geographically close to users, reducing latency by eliminating roundtrips to origin servers for server-side rendering, personalization, and middleware logic.

## What is Edge Rendering?

Traditional SSR runs on a centralized origin server (e.g., US-East). A user in Singapore experiences: Browser → CDN (cache miss) → Origin (US) → Back. Round-trip latency: ~250ms.

Edge rendering deploys server logic to **distributed edge nodes** worldwide (300+ PoPs). A Singapore user hits a Singapore edge node — round-trip latency drops to ~5ms.

```
Traditional SSR:                    Edge SSR:
User (Singapore)                    User (Singapore)
  │ 250ms RTT                         │ 5ms RTT
  ▼                                   ▼
Origin (US-East)               Edge Node (Singapore)
  │ Render HTML                   │ Render HTML
  ▼                               ▼
Send response                  Send response
```

## Edge Runtimes

Edge functions use a **restricted Web Standards API** — not full Node.js. This is a key constraint.

| Platform | Runtime | PoPs | Node.js Support |
|----------|---------|------|----------------|
| Cloudflare Workers | V8 isolates | 300+ | Subset (node_compat) |
| Vercel Edge Functions | V8 (Cloudflare) | 300+ | No |
| Netlify Edge | Deno | 300+ | No |
| AWS Lambda@Edge | Node.js 18 | ~25 | Yes (limited) |
| Fastly Compute | Wasm (Rust/JS) | 50+ | No |
| Deno Deploy | Deno | 35+ | No |

Available APIs: `fetch`, `Request`/`Response`, `URL`, `crypto`, `TextEncoder`, `Cache API`, `ReadableStream`.

NOT available: `fs`, `net`, `child_process`, most Node.js built-ins.

## Edge SSR with Next.js

```tsx
// app/page.tsx — Edge runtime
export const runtime = 'edge';

export default async function Page({ params }: { params: { slug: string } }) {
  // Data fetching at edge — hits nearby KV store or origin
  const data = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await data.json();
  
  return <article><h1>{post.title}</h1><p>{post.body}</p></article>;
}
```

```tsx
// middleware.ts — Runs at edge on every request
import { NextResponse } from 'next/server';

export function middleware(request: Request) {
  const country = request.headers.get('x-vercel-ip-country') ?? 'US';
  
  // Redirect to localized version at edge — no origin roundtrip
  if (country === 'DE') {
    return NextResponse.redirect(new URL('/de' + request.nextUrl.pathname, request.url));
  }
}
export const config = { matcher: ['/((?!_next|api).*)'] };
```

## Cloudflare Workers

The most widely deployed edge platform (processes 50M+ requests/day at the edge):

```ts
// Cloudflare Worker — full SSR at edge
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    
    // Route handling
    if (url.pathname.startsWith('/api/')) {
      return handleAPI(request, env);
    }
    
    // Render React at edge
    const html = await renderToReadableStream(
      <App url={url} />, 
      { bootstrapScripts: ['/client.js'] }
    );
    
    return new Response(html, { 
      headers: { 'Content-Type': 'text/html' } 
    });
  }
};
```

**Cloudflare Workers KV**: Eventually consistent key-value store accessible at edge.  
**Cloudflare Durable Objects**: Strongly consistent storage at edge (for sessions, real-time).  
**Cloudflare R2**: S3-compatible storage accessed from edge without egress fees.

## Edge Middleware Use Cases

| Use Case | Implementation |
|----------|---------------|
| A/B Testing | Split traffic at edge, set cookie |
| Geo-based routing | `x-vercel-ip-country` header → redirect |
| Authentication | Validate JWT at edge — no origin hit |
| Bot detection | Rate limit, challenge at edge |
| Personalization | Serve variant based on user segment cookie |
| Feature flags | Read KV store, rewrite URL |
| Edge caching | Cache API to store/serve rendered HTML |

## Streaming from Edge

Edge + Streaming SSR = extremely low TTFB:

```ts
// Vercel Edge Function with streaming
import { renderToReadableStream } from 'react-dom/server';

export const config = { runtime: 'edge' };

export default async function handler() {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/client.js'],
  });
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    },
  });
}
```

First byte from Singapore: ~10-20ms vs ~300ms from US-East origin.

## Edge KV and Databases

Full SSR at edge requires data sources *also* at the edge:

| Solution | Type | Latency | Use Case |
|----------|------|---------|---------|
| Cloudflare KV | Eventually consistent KV | <5ms | Config, sessions |
| Cloudflare Durable Objects | Strongly consistent | ~10ms | Real-time, coordination |
| PlanetScale (Vitess) | MySQL, distributed | ~20ms | Relational data |
| Neon | Postgres serverless | ~20ms | Full DB at edge |
| Upstash Redis | Redis HTTP API | ~10ms | Caching, rate limit |
| Turso (libSQL) | SQLite replicated | ~5ms | Read-heavy data |

## Performance Results

Real-world data:
- **Vercel**: Edge middleware reduces redirect latency from ~150ms to ~5ms
- **Cloudflare**: Workers handle 50M+ req/day; p99 latency < 50ms globally
- **Next.js with edge runtime**: TTFB improvements of 100-200ms vs origin SSR for global users
- **Auth.js at edge**: JWT validation at edge eliminates origin auth roundtrip

## Trade-offs

| Pros | Cons |
|------|------|
| Dramatically lower global latency | No full Node.js — limited npm ecosystem |
| Geo-based personalization at near-zero cost | Cold start on some platforms (Lambda@Edge) |
| Scalability: auto-scales to millions of RPS | Debugging edge functions is harder |
| Streaming support at edge | Data must be co-located (edge KV, replicated DB) |
| Reduces origin load | Bundle size limits (1-5MB on Cloudflare) |

## When to Use

✅ **Use when:**
- Global audience with diverse geographic distribution
- Authentication, A/B testing, geo-routing logic (always edge)
- Content personalization without full SSR
- Streaming SSR for data-light pages

❌ **Avoid when:**
- Depending on Node.js-only libraries (use Node.js Lambda/ECS instead)
- Requiring strong transactional consistency (use origin + DB)
- CPU-intensive rendering (edge has CPU time limits)

---
*Related: [[Streaming SSR]], [[JAMstack Architecture]], [[Rendering Strategies]], [[Micro-Frontends]]*
