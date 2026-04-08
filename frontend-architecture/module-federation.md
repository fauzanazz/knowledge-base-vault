---
title: "Module Federation"
category: frontend-architecture
summary: "A Webpack 5 / Rspack feature enabling multiple independently deployed JavaScript applications to share code and components at runtime, forming the technical backbone of modern micro-frontend architectures."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Module Federation

> A Webpack 5 / Rspack feature enabling multiple independently deployed JavaScript applications to share code and components at runtime, forming the technical backbone of modern micro-frontend architectures.

## Overview

Module Federation (MF), introduced in **Webpack 5** by Zack Jackson in 2020, allows separate JavaScript applications ("remotes") to expose modules that other applications ("hosts") consume at **runtime** — without pre-bundling together. Each application is independently deployed and versioned.

This solves the hardest micro-frontend problem: **shared dependencies without duplication**.

## Core Concepts

### Host
The application that consumes remote modules. Orchestrates the experience.

### Remote
An application that exposes modules for others to consume. Deployed independently.

### Shared Dependencies
Libraries (React, lodash) shared between host and remotes to avoid duplication.

### Exposed Modules
Components, utilities, or entire routing trees a remote makes available.

## Basic Configuration

### Remote App (exposes a component)
```js
// webpack.config.js — Remote
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'cart',
      filename: 'remoteEntry.js',
      exposes: {
        './CartWidget': './src/components/CartWidget',
        './checkout': './src/pages/Checkout',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

### Host App (consumes remote)
```js
// webpack.config.js — Host
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        cart: 'cart@https://cart.cdn.com/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

```tsx
// Host component — lazy imports from remote at runtime
const CartWidget = React.lazy(() => import('cart/CartWidget'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <CartWidget />
    </Suspense>
  );
}
```

## Dynamic Remotes

Remotes can be resolved dynamically at runtime — useful for multi-tenant or feature-flagged deployments:

```js
// Dynamic remote loading
async function loadRemote(scope, module) {
  await __webpack_init_sharing__('default');
  const container = window[scope];
  await container.init(__webpack_share_scopes__.default);
  const factory = await container.get(module);
  return factory();
}

// Load different versions based on feature flags
const url = featureFlags.useNewCart 
  ? 'https://cart-v2.cdn.com/remoteEntry.js' 
  : 'https://cart-v1.cdn.com/remoteEntry.js';
```

## Shared Dependency Version Negotiation

MF's most powerful feature: version negotiation between host and remotes.

```js
shared: {
  react: {
    singleton: true,           // Only one React instance
    requiredVersion: '^18.0.0', // Semantic version constraint
    eager: false,              // Lazy load shared dep
    strictVersion: false,      // Warn but don't fail on mismatch
  }
}
```

**Resolution algorithm:**
1. Both host and remote declare `react: { singleton: true }`
2. Host has React 18.2, Remote has React 18.1
3. MF negotiates: highest compatible version wins (18.2 used by both)
4. If remote requires React 17 (incompatible): warning, separate instance loaded

## Rspack Module Federation

**Rspack** (Rust-based Webpack alternative by ByteDance) implements Module Federation with dramatically faster builds:

```js
// rspack.config.js
const { ModuleFederationPlugin } = require('@module-federation/enhanced/rspack');

module.exports = {
  plugins: [new ModuleFederationPlugin({ /* same API */ })],
};
```

**@module-federation/enhanced** (maintained by Zack Jackson) provides:
- TypeScript types for remote modules
- Runtime plugin system
- Enhanced error handling
- Works with Webpack 5, Rspack, Vite (via plugin)

## Vite Module Federation

```js
// vite.config.js
import federation from '@originjs/vite-plugin-federation';

export default {
  plugins: [
    federation({
      name: 'remote-app',
      filename: 'remoteEntry.js',
      exposes: { './Button': './src/Button.vue' },
      shared: ['vue'],
    }),
  ],
};
```

Note: Vite MF has limitations compared to Webpack 5 — async import chunks work differently.

## Type Safety with TypeScript

```ts
// Generate types for remote modules
// @module-federation/enhanced auto-generates these

declare module 'cart/CartWidget' {
  const CartWidget: React.ComponentType<{ userId: string }>;
  export default CartWidget;
}
```

**Module Federation 2.0** (2024) includes automatic type generation and sharing.

## Production Architecture Pattern

```
                    ┌─────────────────┐
                    │  Shell / Host    │
                    │  (Next.js SSR)   │
                    └────────┬────────┘
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                   ▼
   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
   │ Nav Remote  │   │ Cart Remote │   │PDP Remote   │
   │ (React 18)  │   │ (React 18)  │   │(Vue 3)      │
   └─────────────┘   └─────────────┘   └─────────────┘
```

Companies using MF in production: **DAZN** (streaming platform), **Lululemon** (e-commerce), **Volkswagen**, **ByteDance** internal tools.

## Trade-offs

| Pros | Cons |
|------|------|
| Runtime sharing — no rebuild needed | Network request for each remote on load |
| Independent deployments | Version negotiation complexity |
| Singleton shared deps (no React duplication) | Debugging across remote boundaries |
| Works across frameworks | Circular dependencies can be tricky |
| No organizational coupling | Runtime errors if remote is down |

## When to Use

✅ **Use when:**
- Multiple teams needing independent deployments
- Migrating a monolith incrementally to micro-frontends
- Sharing component libraries at runtime across apps
- A/B testing different versions of a feature

❌ **Avoid when:**
- Single team application
- Performance critical with strict bundle budgets (adds async overhead)
- Server-rendered pages (MF is primarily client-side)

---
*Related: [[Micro-Frontends]], [[SPA vs MPA vs Hybrid]], [[Client State Management]]*
