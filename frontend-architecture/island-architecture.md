---
title: "Island Architecture"
category: frontend-architecture
summary: "A rendering pattern that delivers mostly static HTML with isolated interactive 'islands' that hydrate independently, minimizing JavaScript sent to the browser."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Island Architecture

> A rendering pattern that delivers mostly static HTML with isolated interactive 'islands' that hydrate independently, minimizing JavaScript sent to the browser.

## Origin

Coined by **Jason Miller** (creator of Preact) in 2020 and popularized by **Astro**. The core insight: most content pages are 90%+ static. Shipping a full SPA runtime to render a blog post is wasteful — only the interactive parts (carousel, like button, search) need JavaScript.

## How It Works

The page is rendered as static HTML on the server. Interactive components are marked as "islands" — they are server-rendered into the HTML *and* hydrated client-side in isolation, independently of each other.

```
┌─────────────────────────────────┐
│  Static Header (no JS)          │
├──────────────┬──────────────────┤
│ Static Body  │ 🏝 Cart Widget   │
│  (no JS)     │  (hydrated)      │
├──────────────┴──────────────────┤
│ 🏝 Search Bar (hydrated)        │
├─────────────────────────────────┤
│  Static Footer (no JS)          │
└─────────────────────────────────┘
```

Each island is an isolated component tree. No shared runtime coordination required.

## Astro Implementation

Astro is the canonical island architecture framework. It ships **zero JS by default**:

```astro
---
// Component is server-rendered
import HeavyCarousel from './HeavyCarousel.jsx';
import StaticText from './StaticText.astro';
---

<StaticText />
<!-- Hydration directives control when/how islands hydrate -->
<HeavyCarousel client:visible />
<SearchBar client:idle />
<Cart client:load />
```

### Astro Hydration Directives

| Directive | When it hydrates |
|-----------|-----------------|
| `client:load` | Immediately on page load |
| `client:idle` | When browser is idle (`requestIdleCallback`) |
| `client:visible` | When component enters viewport (IntersectionObserver) |
| `client:media` | When a CSS media query matches |
| `client:only` | Client-side only, no SSR |

## Framework Support

- **Astro**: First-class island support, framework-agnostic (React, Vue, Svelte, Solid in same page)
- **Fresh** (Deno): Islands-based framework with Preact
- **Marko**: IBM's framework with automatic island detection
- **Enhance**: Web-component-centric islands
- **11ty + WebC**: Islands via Web Components

## Partial Hydration vs Islands

These terms overlap but are distinct:
- **Partial hydration**: Don't hydrate the whole page, only some components (superset concept)
- **Islands**: Isolated, independently hydrated components with no shared JS runtime between them (specific implementation)

Islands enforce isolation; partial hydration may still share a framework runtime.

## Performance Benefits

Real-world data from Astro's own benchmarks and community:
- **90% JS reduction** on content-heavy sites vs Next.js
- Markedly improved **Interaction to Next Paint (INP)** due to smaller main-thread parse cost
- Better **LCP** from server-rendered above-the-fold content with no JS blocking
- **FID/TBT** improvements from reduced JS execution

## Trade-offs

| Pros | Cons |
|------|------|
| Minimal JS shipped to client | Islands can't easily share client state |
| Independent hydration prioritization | Not ideal for highly interactive apps |
| Framework-agnostic composition | Cross-island communication requires events/stores |
| Progressive enhancement built-in | Tooling younger than React/Vue ecosystems |
| Excellent for content sites | Not suitable for dashboards or SPAs |

## Shared State Between Islands

Islands are isolated, so cross-island state needs a solution:
- **Nano Stores** (Astro's recommended): framework-agnostic atom store
- **Custom Events**: `window.dispatchEvent(new CustomEvent(...))`
- **URL/Query Params**: for navigation-coupled state
- **localStorage / IndexedDB**: for persistent state

```js
// nanostores shared across React + Vue islands
import { atom } from 'nanostores';
export const cartCount = atom(0);
```

## When to Use

✅ **Use when:**
- Content-heavy sites: blogs, marketing, docs, e-commerce product pages
- Strong Core Web Vitals requirements
- Mix of static and interactive content
- SEO is critical

❌ **Avoid when:**
- Highly interactive apps (dashboards, editors, real-time tools)
- Heavy cross-component state sharing
- Team deeply invested in SPA routing patterns

---
*Related: [[Hydration Strategies]], [[Rendering Strategies]], [[Progressive Enhancement]], [[Streaming SSR]]*
