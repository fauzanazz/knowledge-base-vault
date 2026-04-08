---
title: "JAMstack Architecture"
category: frontend-architecture
summary: "An architecture paradigm decoupling the frontend from the backend by pre-building static assets served via CDN, with dynamic functionality handled by JavaScript and APIs — including edge functions and headless CMS."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# JAMstack Architecture

> An architecture paradigm decoupling the frontend from the backend by pre-building static assets served via CDN, with dynamic functionality handled by JavaScript and APIs — including edge functions and headless CMS.

## What is JAMstack?

**JAM** = **J**avaScript, **A**PIs, **M**arkup. Coined by Netlify CEO Matt Biilmann in 2015.

The core principles:
1. **Pre-build everything possible** at deploy time (not request time)
2. **Serve from CDN** — no origin servers for static content
3. **Dynamic features via APIs** — microservices, serverless, edge functions
4. **JavaScript for enhancement** — not required for core content

```
Traditional Stack:            JAMstack:
Browser                       Browser
  │                             │
  ▼                             ▼
Web Server (dynamic)          CDN (static HTML/JS/CSS)
  │                             │ (on demand / API call)
  ▼                             ▼
Database                      API / Serverless Function
                                │
                                ▼
                              Database / Headless CMS / SaaS
```

## Static Site Generation (SSG)

At build time, the framework fetches all data and pre-renders every page to HTML:

```tsx
// Next.js SSG
export async function getStaticProps() {
  const posts = await fetchBlogPosts();  // Runs at build time
  return { props: { posts } };
}

export async function getStaticPaths() {
  return { paths: ['/posts/hello', '/posts/world'], fallback: false };
}
```

**Build output**: A directory of `.html`, `.js`, `.css` files deployable to any CDN.

**Companies using SSG**: Vercel (their docs), GitHub Docs, Stripe Docs, Smashing Magazine, Cloudflare Docs.

## Incremental Static Regeneration (ISR)

Next.js innovation: regenerate static pages on-demand or on a timer, without full rebuilds:

```tsx
// Next.js ISR — regenerate this page every 60 seconds
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data }, revalidate: 60 };
}
```

The first request after `revalidate` seconds triggers background regeneration. Served stale while regenerating (stale-while-revalidate pattern).

**Platforms**: Vercel (native), Netlify (DPR), Cloudflare Pages.

## Headless CMS

Content editors use a CMS; developers consume content via API. Completely decoupled:

| CMS | Type | Best For |
|-----|------|----------|
| Contentful | API-first | Enterprise, multi-channel |
| Sanity | Real-time, GROQ queries | Flexible schemas |
| Strapi | Self-hosted, open-source | Full control |
| Prismic | Document-based | Marketing sites |
| Hygraph | GraphQL-native | Complex queries |
| Notion API | Docs-as-CMS | Developer teams |

```ts
// Contentful + Next.js at build time
import { createClient } from 'contentful';
const client = createClient({ space: process.env.SPACE_ID, ... });

export async function getStaticProps() {
  const entries = await client.getEntries({ content_type: 'blogPost' });
  return { props: { posts: entries.items } };
}
```

## Edge Functions

Serverless functions running at CDN edge locations globally (< 50ms to any user):

```ts
// Vercel Edge Function
export const config = { runtime: 'edge' };

export default function handler(req: Request) {
  const country = req.headers.get('x-vercel-ip-country') || 'US';
  const personalized = personalizeContent(country);
  return new Response(personalized, { headers: { 'Content-Type': 'text/html' } });
}
```

**Use cases**: A/B testing, authentication redirects, personalization, geolocation, bot detection.

**Platforms**: Vercel Edge Functions, Netlify Edge, Cloudflare Workers, Fastly Compute.

## Build Performance at Scale

JAMstack's Achilles' heel: build times grow with page count.

| Site Size | Build Time | Solution |
|-----------|-----------|---------|
| 100 pages | < 1 min | SSG |
| 10,000 pages | 5-10 min | ISR or on-demand SSR |
| 1M+ pages | Hours | ISR, distributed builds, DSG |

**Gatsby Deferred Static Generation (DSG)**: Delay building low-traffic pages until first request.

## On-Demand ISR (Next.js 12.2+)

Revalidate specific pages via API:
```ts
// pages/api/revalidate.ts
export default async function handler(req, res) {
  await res.revalidate('/products/123');  // Rebuild specific page
  res.json({ revalidated: true });
}
```

Webhooks from headless CMS trigger this on content publish.

## JAMstack vs Traditional Stack

| Factor | JAMstack | Traditional |
|--------|---------|-------------|
| Scalability | Infinite (CDN) | Requires horizontal scaling |
| Security | Reduced attack surface | Exposed server |
| Performance | Excellent (CDN edge) | Depends on origin |
| Hosting cost | Very low | Higher |
| Build time | Can be long | N/A |
| Dynamic personalization | Via edge functions | Easy |
| Real-time content | Limited | Easy |

## Trade-offs

| Pros | Cons |
|------|------|
| Exceptional CDN performance | Long build times for large sites |
| High security (no DB exposed) | Real-time content requires extra patterns |
| Cheap hosting | Personalization complexity |
| Developer experience | ISR invalidation can be tricky |
| Git-based deployments | Not suitable for fully dynamic apps |

## When to Use

✅ **Use when:**
- Marketing sites, blogs, documentation
- High traffic, read-heavy content
- Content changes infrequently (daily, not per-second)
- SEO and performance are priorities

❌ **Avoid when:**
- Highly personalized content per user (use SSR)
- Real-time data (stock tickers, live feeds)
- Complex session/auth per page (use SSR or edge)
- User-generated content at scale

---
*Related: [[Rendering Strategies]], [[Edge Rendering]], [[SPA vs MPA vs Hybrid]], [[React Server Components]]*
