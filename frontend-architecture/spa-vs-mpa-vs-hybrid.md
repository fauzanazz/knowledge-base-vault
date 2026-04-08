---
title: "SPA vs MPA vs Hybrid"
category: frontend-architecture
summary: "A comparison of Single Page Applications, Multi-Page Applications, and modern hybrid approaches, covering trade-offs in performance, SEO, developer experience, and user experience."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# SPA vs MPA vs Hybrid

> A comparison of Single Page Applications, Multi-Page Applications, and modern hybrid approaches, covering trade-offs in performance, SEO, developer experience, and user experience.

## Single Page Applications (SPA)

SPAs load once and dynamically update the DOM via JavaScript. Navigation is handled client-side — no full page reloads.

**Architecture:**
```
Browser ──► Load index.html + app bundle ──► JS router intercepts clicks
         ──► Fetch data via API           ──► Update DOM in place
```

**Examples**: Gmail, Figma, Linear, Notion, GitHub (partially), Twitter Web.

**Core technologies**: React + React Router, Vue + Vue Router, Angular Router.

### SPA Pros
- Fluid navigation (no page reloads)
- Rich client-side state management
- Works offline (with service workers)
- App-shell caching for repeat visits
- Transitions and animations between views

### SPA Cons
- Large initial JS bundle → slow FCP on first load
- Poor SEO without SSR (crawlers struggle with JS)
- Browser back/forward button complexity
- Memory leaks if not managed carefully
- Flash of empty content before JS executes

**Performance profile:**
```
First Load: Slow (full bundle download + execute)
Subsequent Navigations: Fast (no server roundtrip)
SEO: Poor (without SSR)
```

## Multi-Page Applications (MPA)

Traditional server-rendered architecture. Each navigation is a full page load — server returns complete HTML.

**Examples**: Wikipedia, Amazon product pages, government sites, most WordPress sites.

**Core technologies**: PHP/Laravel, Ruby on Rails, Django, traditional Next.js Pages Router with SSR.

### MPA Pros
- Excellent SEO (server-rendered HTML)
- Fast first paint (HTML arrives fully rendered)
- Natural browser behavior (back/forward, Ctrl+F)
- Works without JavaScript
- Independent page caching

### MPA Cons
- Full page reloads between routes (jarring UX)
- No shared client state between pages
- Can't do complex client-side transitions
- Re-downloading shared components on each navigation

**Performance profile:**
```
First Load: Fast (pre-rendered HTML)
Subsequent Navigations: Medium (full page reload, but cacheable)
SEO: Excellent
```

## Hybrid Approaches

Modern frameworks blend SPA and MPA — server-render for first load, client-navigate for subsequent routes.

### Server-Side Rendering + Client Hydration (SSR+SPA)

```
First visit: Server renders full HTML → Fast FCP
Subsequent: Client router takes over → SPA navigation
```

**Frameworks**: Next.js (Pages Router), Nuxt.js, SvelteKit, Remix.

```tsx
// Next.js: SSR first load, SPA navigation
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}
```

### Partial Prerendering (Next.js 14+)

Next.js 14 introduced PPR: pages have a static shell (instantly served from CDN) with dynamic holes that stream in:

```tsx
export const experimental_ppr = true;

export default function Page() {
  return (
    <>
      <StaticShell />    {/* Pre-rendered, CDN-cached */}
      <Suspense>
        <DynamicContent /> {/* Streams from server */}
      </Suspense>
    </>
  );
}
```

### Islands Architecture (MPA + Selective Interactivity)

Astro, Fresh: MPA with opt-in interactive islands. Best of both worlds for content sites.

### View Transitions API

The browser-native API for SPA-like transitions in MPAs:

```js
document.startViewTransition(() => {
  // Update DOM / navigate
  updateContent(newContent);
});
```

Supported in Chrome 111+, Firefox (behind flag). Astro supports this natively.

## Framework Decision Matrix

| Framework | Architecture | Best For |
|-----------|-------------|----------|
| Create React App | SPA | Internal tools, dashboards |
| Next.js App Router | Hybrid (RSC + streaming) | Full-stack apps, e-commerce |
| Remix | Hybrid (SSR + client) | Form-heavy, data-driven apps |
| Astro | MPA + Islands | Content sites, docs, marketing |
| SvelteKit | Hybrid | Versatile, performance-focused |
| Nuxt 3 | Hybrid | Vue ecosystem |
| Angular SSR | Hybrid | Enterprise apps |

## Performance Comparison (Real-World)

| Metric | SPA | MPA | Hybrid SSR |
|--------|-----|-----|-----------|
| TTFB | Fast | Varies | Fast |
| FCP | Slow (cold) | Fast | Fast |
| LCP | Slow | Fast | Fast |
| Navigation speed | Very fast | Slow | Fast |
| SEO | Poor | Excellent | Excellent |
| Offline support | Excellent | Poor | Medium |

## When to Use Each

### Choose SPA when:
- Application is highly interactive (editor, dashboard, game)
- Most traffic is authenticated/returning users
- Rich client-side state across all views
- Offline functionality required

### Choose MPA when:
- Content-first (blog, docs, marketing)
- SEO is paramount
- Progressive enhancement required
- Simple pages with minimal interactivity

### Choose Hybrid (SSR/RSC) when:
- Mix of public (SEO) and authenticated (interactive) pages
- E-commerce (SEO + cart/checkout interactivity)
- Need both fast first-load and smooth navigation
- This is the default for most modern production apps

---
*Related: [[Rendering Strategies]], [[JAMstack Architecture]], [[Island Architecture]], [[React Server Components]]*
