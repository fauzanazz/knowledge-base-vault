---
title: "React Server Components"
category: frontend-architecture
summary: "A React architecture that renders components exclusively on the server, eliminating client-side JS for data-fetching components while enabling seamless server/client composition."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# React Server Components

> A React architecture that renders components exclusively on the server, eliminating client-side JS for data-fetching components while enabling seamless server/client composition.

## Overview

React Server Components (RSC), shipped in React 18 and adopted by **Next.js 13+ (App Router)**, fundamentally change the React mental model. Server Components run *only* on the server — they can access databases, file systems, and secrets directly, and their code is **never sent to the browser**.

Introduced by Dan Abramov & Lauren Tan at Meta in 2020. Now production-proven at Vercel, Shopify, and thousands of Next.js deployments.

## The Server/Client Boundary

```
Server                          Client
──────────────────────          ──────────────────────
Layout (RSC)                    
  └─ Sidebar (RSC)              
  └─ Main (RSC)                 
       └─ ProductList (RSC)     
            └─ AddToCart ─────► AddToCart (CC)
                                  └─ useCart hook
                                  └─ onClick handler
```

**Server Components (RSC):**
- Default in Next.js App Router
- Can `async/await` directly — no `useEffect` + fetch
- Can import server-only code (DB, secrets)
- Cannot use `useState`, `useEffect`, browser APIs, event handlers

**Client Components (CC):**
- Opted in via `'use client'` directive
- Full React reactivity, hooks, event handlers
- Serialized and hydrated on the client
- Can receive RSC-rendered children as props (the "donut pattern")

```tsx
// Server Component — runs only on server
async function ProductList() {
  const products = await db.query('SELECT * FROM products'); // direct DB!
  return (
    <ul>
      {products.map(p => (
        <li key={p.id}>
          {p.name}
          <AddToCart productId={p.id} /> {/* Client Component */}
        </li>
      ))}
    </ul>
  );
}
```

```tsx
'use client'; // Client Component boundary
function AddToCart({ productId }: { productId: string }) {
  const [added, setAdded] = useState(false);
  return <button onClick={() => setAdded(true)}>Add to Cart</button>;
}
```

## RSC Wire Format

RSC doesn't return HTML — it returns a special **RSC payload** (a serialized tree). This enables:
- **Streaming**: Chunks arrive progressively, Suspense boundaries unlock as data resolves
- **Client-side refetching**: Router can refetch server components without full page reload
- **No hydration mismatch**: Client knows exactly what was rendered

```
J0:{"type":"div","props":{"children":[...]}}
J1:["$","h1",null,{"children":"Title"}]
```

## Streaming with Suspense

```tsx
// app/page.tsx
export default function Page() {
  return (
    <main>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        <SlowDataComponent /> {/* Streams in when ready */}
      </Suspense>
      <Suspense fallback={<Skeleton />}>
        <AnotherSlowComponent /> {/* Independent stream */}
      </Suspense>
    </main>
  );
}
```

Each Suspense boundary can resolve independently, avoiding waterfall delays.

## Next.js App Router Integration

| Feature | Pages Router | App Router (RSC) |
|---------|-------------|-----------------|
| Data fetching | `getServerSideProps` | `async` component body |
| Layouts | `_app.js` | Nested `layout.tsx` (RSC) |
| Caching | Manual | Automatic with `fetch` cache |
| Streaming | Limited | First-class Suspense |
| Bundle size | Larger | Dramatically smaller |

## Caching Model (Next.js)

```tsx
// Cached indefinitely (SSG-like)
fetch('/api/data', { cache: 'force-cache' });

// Never cached (SSR-like)
fetch('/api/data', { cache: 'no-store' });

// ISR-like: revalidate every 60s
fetch('/api/data', { next: { revalidate: 60 } });
```

## Performance Implications

- **Zero JS for data-fetching code**: A component that fetches 50KB of data and renders a list ships 0 bytes of JS
- **Reduced waterfalls**: Sequential `await` in RSC doesn't cause client roundtrips
- **Smaller hydration payload**: Only Client Components hydrate
- **Shopify reported** 50% reduction in JS bundle on Hydrogen v2 migration to RSC

## Trade-offs

| Pros | Cons |
|------|------|
| Eliminates client data-fetching JS | Mental model complexity (server vs client) |
| Direct DB/file system access | Cannot use most npm packages (browser-only) |
| Streaming out of the box | Debugging across boundary is harder |
| Smaller bundles | Framework lock-in (Next.js App Router) |
| Collocated data fetching | RSC format not yet portable across frameworks |

## When to Use

✅ **Use when:**
- Building with Next.js App Router
- Heavy data-fetching components that don't need interactivity
- Wanting to move secrets/DB access to server without API routes
- Performance-sensitive production apps

❌ **Avoid when:**
- Highly interactive, real-time components (use Client Components)
- Framework other than Next.js (limited RSC support outside Next.js)
- Team unfamiliar with server/client mental model split

---
*Related: [[Streaming SSR]], [[Hydration Strategies]], [[Rendering Strategies]], [[Island Architecture]]*
