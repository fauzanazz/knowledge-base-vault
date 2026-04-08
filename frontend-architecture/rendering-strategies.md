---
title: "Rendering Strategies"
category: frontend-architecture
summary: "A comprehensive guide to the four primary web rendering strategies — CSR, SSR, SSG, and ISR — covering their performance characteristics, SEO implications, infrastructure requirements, and when to choose each."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Rendering Strategies

> A comprehensive guide to the four primary web rendering strategies — CSR, SSR, SSG, and ISR — covering their performance characteristics, SEO implications, infrastructure requirements, and when to choose each.

## The Four Core Strategies

### 1. Client-Side Rendering (CSR)

The browser downloads a minimal HTML shell + JavaScript bundle, then renders everything client-side.

```
Request → Receive index.html (empty div) → Download JS → Execute → Render
```

```html
<!-- CSR HTML is nearly empty -->
<!DOCTYPE html>
<html>
<body>
  <div id="root"></div>
  <script src="/bundle.js"></script>  <!-- All the work is here -->
</body>
</html>
```

**Metrics:**
- **TTFB**: Fast (tiny HTML)
- **FCP**: Slow (after JS parse)
- **TTI**: Slow (JS-heavy)
- **SEO**: Poor (without prerendering)
- **Server infra**: CDN only (static)

**Best for**: Dashboards, authenticated apps, internal tools  
**Examples**: Create React App, vanilla Vite, Gmail client

---

### 2. Server-Side Rendering (SSR)

Every request triggers server-side rendering — data fetched and HTML generated per-request.

```
Request → Server fetches data → Renders HTML → Sends full HTML → Hydrate
```

```tsx
// Next.js Pages Router SSR
export async function getServerSideProps(context) {
  const { req, params } = context;
  const user = await getUser(req.cookies.token);
  const data = await fetchPersonalizedData(user.id);
  return { props: { data } };
}
```

**Metrics:**
- **TTFB**: Slow (must render first)
- **FCP**: Fast (after TTFB)
- **SEO**: Excellent
- **Personalization**: Perfect (per-request)
- **Infra**: Requires always-on server

**Best for**: Personalized pages, authenticated dashboards, real-time data  
**Examples**: Next.js SSR, Nuxt SSR, SvelteKit SSR, Remix

---

### 3. Static Site Generation (SSG)

HTML generated at **build time** and served from CDN. No server required.

```
Build → Fetch all data → Generate HTML files → Deploy to CDN
Request → CDN serves cached HTML → Client hydrates
```

```tsx
// Next.js SSG
export async function getStaticProps() {
  const posts = await getAllBlogPosts(); // Runs at BUILD TIME
  return { props: { posts } };
}

export async function getStaticPaths() {
  const posts = await getAllBlogPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: false, // 404 for unknown paths
  };
}
```

**Metrics:**
- **TTFB**: Fastest (CDN edge)
- **FCP**: Fastest
- **SEO**: Excellent
- **Dynamic content**: None (stale at build time)
- **Infra**: CDN only (cheapest)

**Best for**: Marketing sites, blogs, documentation  
**Examples**: Gatsby, 11ty, Hugo, Astro, Next.js with SSG

---

### 4. Incremental Static Regeneration (ISR)

Next.js innovation: hybrid of SSG + SSR. Pages are statically generated but can be **regenerated** in the background.

```tsx
// Next.js ISR
export async function getStaticProps() {
  const product = await getProduct(params.id);
  return {
    props: { product },
    revalidate: 60,  // Regenerate at most once per 60 seconds
  };
}
```

**How it works:**
1. First request: CDN serves stale HTML (if exists) or waits for render
2. In background: Next.js re-generates the page
3. Next request: Serves fresh HTML

```
                     stale-while-revalidate
User Request ──► CDN (cached HTML) ──► Background: fetch + regenerate
                     ▲
              Next user gets fresh HTML
```

**On-Demand ISR** (Next.js 12.2+):
```ts
// Triggered by CMS webhook on content update
await res.revalidate('/products/123');
```

**Metrics:**
- **TTFB**: Fast (CDN)
- **FCP**: Fast
- **SEO**: Excellent
- **Freshness**: Configurable (seconds to hours)
- **Infra**: Serverless + CDN

---

## Comparison Matrix

| Strategy | TTFB | FCP | SEO | Personalization | Freshness | Cost |
|----------|------|-----|-----|-----------------|----------|------|
| CSR | Fast | Slow | ❌ | ✅ | Real-time | Low |
| SSR | Slow | Fast | ✅ | ✅ | Real-time | High |
| SSG | Fastest | Fastest | ✅ | ❌ | Build-time | Lowest |
| ISR | Fast | Fast | ✅ | ❌ (per-page) | Configurable | Low |
| ISR + PPR | Fast | Fast | ✅ | Edge-only | Configurable | Low |

## Partial Prerendering (PPR) — Next.js 14+

Next.js 14 introduces PPR as a new hybrid: static shell + dynamic holes:

```tsx
export const experimental_ppr = true;

export default function ProductPage({ params }) {
  return (
    <div>
      {/* Static shell — cached at CDN */}
      <ProductDetails id={params.id} />
      
      {/* Dynamic hole — streams from server */}
      <Suspense fallback={<RecommendationSkeleton />}>
        <PersonalizedRecommendations userId={getUserId()} />
      </Suspense>
    </div>
  );
}
```

## Choosing the Right Strategy

### Decision Tree

```
Is content personalized per user?
  ├── Yes → SSR (or CSR for authenticated apps)
  └── No →
       Does content change frequently? (< 1 hour)
         ├── Yes, real-time → SSR
         ├── Yes, periodic → ISR
         └── Rarely → SSG
```

### By Application Type

| App Type | Recommended Strategy |
|----------|---------------------|
| Marketing / landing pages | SSG |
| Blog / documentation | SSG or ISR |
| E-commerce product pages | ISR (price changes periodically) |
| E-commerce cart/checkout | SSR or CSR |
| News site | ISR (short revalidation) |
| Dashboard | CSR or SSR |
| Personalized feed | SSR or CSR |
| Admin panel | CSR |

## Infrastructure Implications

```
SSG:  Build pipeline → CDN (S3 + CloudFront, Vercel, Netlify)
ISR:  Serverless function + CDN cache + revalidation queue
SSR:  Always-on server / serverless with cold start consideration
CSR:  CDN (just static assets)
```

## Per-Route Strategy (Next.js App Router)

Modern frameworks allow **per-route** strategy:
```tsx
// app/blog/[slug]/page.tsx → SSG
export const revalidate = false; // or 0 for SSR

// app/dashboard/page.tsx → SSR
export const dynamic = 'force-dynamic';

// app/products/[id]/page.tsx → ISR (1 hour)
export const revalidate = 3600;
```

---
*Related: [[JAMstack Architecture]], [[React Server Components]], [[Streaming SSR]], [[Edge Rendering]], [[SPA vs MPA vs Hybrid]]*
