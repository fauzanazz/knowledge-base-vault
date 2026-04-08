---
title: "Micro-Frontends"
category: frontend-architecture
summary: "An architectural pattern that decomposes a frontend monolith into independently deployable, team-owned UI slices composed at runtime or build time."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Micro-Frontends

> An architectural pattern that decomposes a frontend monolith into independently deployable, team-owned UI slices composed at runtime or build time.

## What Are Micro-Frontends?

Micro-frontends extend microservice thinking to the UI layer. Each team owns an end-to-end vertical slice of the product — from database to UI — and ships their frontend fragment independently. The shell/host application composes these fragments into a coherent experience at runtime or during a build step.

**Companies using this pattern:** IKEA, Zalando, Spotify, OpenTable, DAZN, Klarna, SoundCloud.

## Composition Strategies

### 1. Build-Time Integration
NPM packages published by each team; host imports and bundles them together.
- ✅ Simple, single bundle, tree-shakeable
- ❌ Teams are coupled at deploy time; one publish blocks others

### 2. Runtime Integration via `<script>` Tags / ESM
Host page loads remote JS bundles dynamically at runtime.
```html
<script src="https://team-a.cdn.com/bundle.js"></script>
```
- ✅ Independent deployments
- ❌ Waterfall loading, no shared dependency management

### 3. Module Federation (Webpack 5 / Rspack)
The dominant modern approach. Remotes expose modules; host consumes them at runtime with shared dependency negotiation. See [[Module Federation]] for deep dive.

### 4. iframes
Oldest isolation mechanism. Used by Spotify for embedded players.
- ✅ Perfect CSS/JS isolation
- ❌ Accessibility challenges, no shared state, URL routing complexity

### 5. Web Components / Custom Elements
Framework-agnostic shell using `<team-a-header>` etc. SAP uses this heavily.

## single-spa Framework

[single-spa](https://single-spa.js.org/) is the most popular framework-agnostic orchestrator:
- Registers each micro-frontend as a "registered application" with `activeWhen` URL rules
- Manages lifecycle: `bootstrap`, `mount`, `unmount`
- Works with React, Vue, Angular simultaneously

```js
import { registerApplication, start } from 'single-spa';
registerApplication({
  name: '@org/navbar',
  app: () => import('@org/navbar'),
  activeWhen: ['/'],
});
start();
```

## Communication Between Micro-Frontends

| Method | Use Case | Coupling |
|--------|----------|---------|
| Custom Events | Loosely coupled events | Low |
| Shared EventBus | Cross-team pub/sub | Medium |
| URL / Query Params | Navigation state | Low |
| Shared Redux/Zustand | Heavy shared state | High |
| Backend for Frontend (BFF) | Server-driven state | None |

Prefer **custom events** or **URL state** to avoid tight coupling.

## Trade-offs

| Pros | Cons |
|------|------|
| Independent deployments per team | Higher operational complexity |
| Technology diversity (React + Vue + Angular) | Bundle duplication risk |
| Fault isolation (one MFE fails, others live) | Cross-MFE UX consistency challenges |
| Scales org horizontally | Performance overhead (multiple network requests) |
| Clear ownership boundaries | Testing integration is harder |

## Performance Implications

- **Bundle duplication**: Without Module Federation shared scope, React gets shipped twice. Use `shared: { react: { singleton: true } }`.
- **Loading waterfalls**: Lazy-load MFEs only when routes activate.
- **Core Web Vitals**: Each remote module load adds to TTFB/LCP. Use preloading hints.
- **Caching**: Hash-based versioned bundles allow aggressive CDN caching.

## When to Use

✅ **Use when:**
- 5+ independent teams working on the same frontend
- Different teams have different release cadences
- Sub-applications need different tech stacks (legacy migration)
- You're decomposing a frontend monolith incrementally

❌ **Avoid when:**
- Small team (< 3 engineers) — complexity not worth it
- Tight inter-component state sharing is required
- Performance is paramount and bundle size is constrained

---
*Related: [[Module Federation]], [[SPA vs MPA vs Hybrid]], [[Edge Rendering]]*
