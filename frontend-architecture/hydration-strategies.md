---
title: "Hydration Strategies"
category: frontend-architecture
summary: "The spectrum of techniques for attaching JavaScript interactivity to server-rendered HTML, ranging from full eager hydration to lazy, progressive, and partial approaches that minimize Time to Interactive."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Hydration Strategies

> The spectrum of techniques for attaching JavaScript interactivity to server-rendered HTML, ranging from full eager hydration to lazy, progressive, and partial approaches that minimize Time to Interactive.

## What is Hydration?

When a server-rendered page arrives in the browser, it's static HTML. **Hydration** is the process of attaching JavaScript event listeners and state to that HTML, making it interactive. The challenge: hydration is expensive — it requires parsing and executing JavaScript, reconstructing component trees, and reconciling with the DOM.

## 1. Full Hydration (Classic SPA/SSR)

The default behavior of React, Vue, and Angular SSR. The entire component tree hydrates on load.

```
Server: Renders full HTML
Client: Downloads full framework + app JS
        Reconstructs entire component tree
        Attaches all event listeners
        Page becomes interactive
```

**Cost**: Full JS parse + execute. On mobile, can be 3-10s TTI.  
**Used by**: Create React App SSR, Vue SSR, Angular Universal.

## 2. Partial Hydration

Only specific components hydrate — static components ship no JS:

```tsx
// Marko (IBM) — automatic partial hydration
<div>
  <StaticContent /> <!-- Never hydrated, 0 JS -->
  <InteractiveWidget /> <!-- Hydrated, sends JS -->
</div>
```

**Frameworks**: Marko (IBM), Astro (islands), Elder.js  
**Benefit**: Dramatically less JS. Static portions = zero cost.

## 3. Progressive Hydration

Hydration is deferred and prioritized. Components hydrate progressively — critical path first, below-the-fold later.

```tsx
// React's concurrent hydration (React 18)
// React prioritizes hydrating interacted components first
<Suspense>
  <AboveTheFold /> {/* Hydrates first */}
</Suspense>
<Suspense>
  <BelowTheFold /> {/* Hydrates later, doesn't block */}
</Suspense>
```

React 18 implements this via **concurrent hydration**: hydration yields to browser frames, can be interrupted by user interaction.

## 4. Lazy / On-Demand Hydration

Components only hydrate when needed (user interaction, viewport visibility):

```tsx
// Astro — client:visible
<HeavyWidget client:visible /> // Hydrates on IntersectionObserver trigger

// React — manual lazy hydration
const [hydrate, setHydrate] = useState(false);
return hydrate 
  ? <HeavyComponent /> 
  : <div onClick={() => setHydrate(true)} dangerouslySetInnerHTML={{__html: ssr}} />;
```

**Frameworks**: Astro (`client:visible`, `client:idle`), React Aria's lazy patterns, `react-lazy-hydration` library.

## 5. Selective Hydration (React 18)

React 18's streaming SSR enables selective hydration: React prioritizes hydrating components the user is *currently interacting with*:

```
1. Stream HTML arrives, shell renders
2. React starts hydrating AboveTheFold
3. User clicks BelowTheFold (not yet hydrated)
4. React pauses AboveTheFold, hydrates BelowTheFold first
5. Click handler fires
6. React resumes AboveTheFold
```

This requires `renderToPipeableStream` + `hydrateRoot` with `startTransition`.

## 6. Islands Hydration

Each interactive "island" hydrates independently with its own runtime (see [[Island Architecture]]). No shared framework instance between islands.

```astro
<Header />                    <!-- Static, 0 JS -->
<SearchBar client:load />     <!-- Hydrates immediately -->
<Comments client:visible />   <!-- Hydrates on scroll -->
<Footer />                    <!-- Static, 0 JS -->
```

## 7. Resumability (No Hydration)

Qwik's answer: serialize all state into HTML, no hydration step required. See [[Resumability]].

## Hydration Strategy Comparison

| Strategy | JS on Load | TTI | Complexity | Framework |
|----------|-----------|-----|-----------|-----------|
| Full | 100% | Slowest | Low | React, Vue, Angular |
| Progressive | 100% | Medium | Medium | React 18 |
| Selective | 100% | Medium | Medium | React 18 |
| Partial | Partial | Faster | Medium | Marko, Astro |
| Lazy | Deferred | Fast | Medium | Astro, custom |
| Islands | Per-island | Fast | Medium | Astro, Fresh |
| Resumability | ~1KB | Fastest | High | Qwik |

## Hydration Mismatch Errors

A common pitfall: server HTML doesn't match what client renders → React throws hydration errors.

**Causes:**
- `new Date()` — server/client time differs
- `Math.random()` — non-deterministic
- `window` / browser APIs used during SSR
- Locale-dependent formatting

**Solutions:**
```tsx
// Use suppressHydrationWarning for known mismatches
<time suppressHydrationWarning>{new Date().toISOString()}</time>

// Or defer to client only
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
if (!mounted) return <Skeleton />;
```

## Performance Benchmarks

Based on Web Almanac 2023 and community benchmarks:
- **Full hydration** (React SPA): median TTI 4.2s on mobile (3G)  
- **Islands (Astro)**: ~90% JS reduction on content sites  
- **React 18 progressive**: ~30% TBT improvement over React 17  
- **Qwik resumability**: Sub-1s TTI on content pages  

## When to Use Which Strategy

| Scenario | Recommended Strategy |
|----------|---------------------|
| Highly interactive app | Full / Progressive hydration |
| Content site with some interaction | Islands / Partial |
| E-commerce product pages | Islands or Lazy |
| Mobile-first, slow networks | Resumability or Islands |
| Real-time dashboard | Full with streaming |

---
*Related: [[Island Architecture]], [[Resumability]], [[Streaming SSR]], [[React Server Components]]*
