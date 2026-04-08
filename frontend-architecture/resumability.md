---
title: "Resumability"
category: frontend-architecture
summary: "An alternative to hydration where application state is serialized into HTML so the browser can resume execution instantly without replaying component logic or re-downloading framework bootstrap code."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Resumability

> An alternative to hydration where application state is serialized into HTML so the browser can resume execution instantly without replaying component logic or re-downloading framework bootstrap code.

## The Problem with Hydration

Traditional SSR + hydration has a fundamental inefficiency: **work is duplicated**. The server renders HTML, but then the client must re-execute all component logic to reconstruct the virtual DOM and attach event listeners — even if the UI hasn't changed.

```
Traditional Hydration Timeline:
1. Server renders HTML → sends to client         (fast)
2. Browser downloads framework JS                (slow, blocking)
3. Framework boots, parses component tree        (CPU expensive)
4. Re-executes all render functions              (redundant work)
5. Reconciles with existing DOM                  (wasteful)
6. Attaches event listeners                      (finally interactive!)
```

On low-end devices, this "Time to Interactive" gap can be **3-10 seconds**, even though the page *looks* done.

## Resumability: The Concept

Resumability (coined and implemented by **Misko Hevery** in **Qwik**) serializes all execution state into HTML. The browser doesn't re-run the application — it *resumes* from where the server left off.

```
Resumability Timeline:
1. Server renders HTML + serializes all state    (comprehensive)
2. Browser receives HTML → page is interactive   (immediately!)
3. Framework listener (~1KB) attaches globally   (tiny bootstrap)
4. User clicks button                            
5. ONLY that handler's code is lazy-loaded       (on-demand)
6. Handler executes with deserialized state      (resumes!)
```

**No eager JavaScript execution on load.** No framework boot. No virtual DOM reconciliation.

## Qwik's Implementation

Qwik is the only production framework built on resumability. Key mechanisms:

### 1. Serialized State in HTML
```html
<!-- Qwik serializes component state into the DOM -->
<button on:click="./chunk-abc123.js#handleClick_component_div_0" 
        q:id="1">
  Clicked: 3
</button>
<script type="qwik/json">{"objs":["3"],"subs":[]}</script>
```

### 2. `$` Symbol — Lazy Boundaries
The `$` suffix marks a "lazy boundary" — code that will be split into a separate chunk:
```tsx
import { component$, useSignal } from '@builder.io/qwik';

export const Counter = component$(() => {
  const count = useSignal(0);
  
  // onClick$ is a lazy boundary — this handler is in a separate chunk
  // It's only downloaded when the user actually clicks
  return <button onClick$={() => count.value++}>Count: {count.value}</button>;
});
```

### 3. Global Event Listener
Qwik installs a single `<script>` that listens for ALL events globally. On an event, it resolves the relevant handler, fetches the chunk if not cached, deserializes state, and executes.

### 4. Speculative Module Fetching
Qwik City uses service workers to prefetch likely-needed chunks in the background after load, so they're cached before the user interacts.

## Resumability vs Hydration Comparison

| Aspect | Hydration (React/Vue) | Resumability (Qwik) |
|--------|----------------------|---------------------|
| JS on load | Full framework + components | ~1KB listener |
| Time to Interactive | After JS parse + execute | Instant |
| Event handling | Attached after hydration | Serialized in HTML |
| State reconstruction | Re-execute render | Deserialize from HTML |
| Lazy loading | Route-level | Component-level (automatic) |
| Developer experience | Mature ecosystem | New paradigm, learning curve |

## Qwik City (Meta-Framework)

Qwik's full-stack framework (analogous to Next.js) with:
- File-based routing
- Server actions / loaders
- Edge-ready (Cloudflare Workers, Vercel Edge)
- Built-in PWA support

**Builder.io** (Qwik's creator) runs qwik.builder.io in production on Qwik, reporting near-instant interactivity metrics.

## Serialization Constraints

Resumability requires serializing component state into HTML, which has constraints:
- Functions cannot be trivially serialized (Qwik uses `$` boundaries to map them to chunk URLs)
- Closures must be analyzable at build time
- Some reactive patterns require framework-specific APIs (not plain React hooks)

## Performance Results

- **Qwik on Builder.io**: ~0ms TTI reported (page is interactive before JS loads)
- **JS bundle on load**: Often < 5KB vs 100-300KB for React apps
- **Core Web Vitals INP**: Dramatically lower due to no hydration blocking

## Trade-offs

| Pros | Cons |
|------|------|
| Near-instant interactivity | Requires Qwik specifically |
| No hydration cost | New mental model ($ boundaries) |
| Automatic code splitting | Less mature ecosystem |
| Mobile/low-end device friendly | Serialization constraints on state |
| No framework boot JS | Debugging lazy chunks is complex |

## When to Use

✅ **Use when:**
- Time to Interactive is critical (e-commerce, mobile-heavy traffic)
- App has lots of components, few initially interacted with
- Targeting low-end devices / slow networks
- Willing to adopt Qwik's paradigm

❌ **Avoid when:**
- Team is deeply invested in React/Vue ecosystem
- App is highly interactive from first frame (games, editors)
- Migration from existing codebase (Qwik is not drop-in)

---
*Related: [[Hydration Strategies]], [[Streaming SSR]], [[React Server Components]], [[Island Architecture]]*
