---
title: "Streaming SSR"
category: frontend-architecture
summary: "A server rendering technique that sends HTML to the browser in chunks as it's generated, allowing the browser to progressively render content before the full response is complete."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Streaming SSR

> A server rendering technique that sends HTML to the browser in chunks as it's generated, allowing the browser to progressively render content before the full response is complete.

## The Problem with Traditional SSR

Classical SSR is synchronous: the server waits for ALL data to resolve before sending any HTML.

```
Traditional SSR:
Request ──► [Wait for all DB queries: 800ms] ──► Send entire HTML ──► Browser renders
                        ↑
              User sees blank screen for 800ms
```

Even fast components (header, footer) must wait for the slowest query.

## How Streaming Works

HTTP/1.1 supports **chunked transfer encoding**; HTTP/2 uses streams natively. The server sends HTML in pieces as each piece becomes ready:

```
Streaming SSR:
Request ──► Send shell HTML immediately (0ms)
        ──► Send header chunk (10ms)
        ──► Send fast-loading sections (50ms)
        ──► Send slow data section when ready (800ms)
        ──► Close stream
```

The browser starts rendering the first chunk while waiting for subsequent chunks — dramatically improving **TTFB** and **FCP**.

## React 18 Streaming with Suspense

React 18's `renderToPipeableStream` (Node.js) and `renderToReadableStream` (Edge/Web Streams) enable Suspense-aware streaming:

```tsx
// Server (Node.js)
import { renderToPipeableStream } from 'react-dom/server';

app.get('/', (req, res) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/client.js'],
    onShellReady() {
      // Send the shell immediately — Suspense fallbacks rendered
      res.setHeader('Content-Type', 'text/html');
      pipe(res);
    },
  });
});
```

```tsx
// App component
function App() {
  return (
    <html>
      <body>
        <Header />         {/* Rendered immediately */}
        <Suspense fallback={<Spinner />}>
          <SlowFeed />     {/* Streamed in when data resolves */}
        </Suspense>
        <Suspense fallback={<Spinner />}>
          <Recommendations />  {/* Independent stream */}
        </Suspense>
        <Footer />         {/* Rendered immediately */}
      </body>
    </html>
  );
}
```

## Out-of-Order Streaming

React's streaming is **out-of-order**: chunks don't have to arrive in DOM order. Each Suspense boundary resolves independently. React injects a small `<script>` that moves content into the right position:

```html
<!-- Initially sent -->
<div id="feed-fallback"><div class="spinner"></div></div>

<!-- Later chunk, appended at bottom of body -->
<div hidden id="S:1">
  <article>Actual feed content...</article>
</div>
<script>
  // Moves content from hidden div to the right position
  $RC("feed-fallback", "S:1")
</script>
```

This enables the browser to render fast sections immediately without waiting for the slow data to arrive in order.

## Selective Hydration

Streaming + Suspense enables **selective hydration**: React starts hydrating the visible shell immediately and prioritizes hydrating components the user interacts with:

```
1. Shell HTML arrives → browser paints it
2. React JS loads → starts hydrating shell
3. User clicks on Sidebar (not yet hydrated)
4. React prioritizes Sidebar hydration over other components
5. Click handler fires after Sidebar hydrates
```

## Next.js App Router Streaming

Next.js App Router uses streaming by default:
```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <>
      <StaticHeader />
      <Suspense fallback={<MetricsSkeleton />}>
        <Metrics />  {/* async RSC — streams independently */}
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <DataTable /> {/* async RSC — streams independently */}
      </Suspense>
    </>
  );
}
```

`loading.tsx` files automatically wrap segments in Suspense.

## Edge Streaming

Streaming works particularly well at the **edge** where TTFB is already low. Cloudflare Workers, Vercel Edge, and Deno Deploy support the Web Streams API:

```ts
// Cloudflare Worker
export default {
  fetch: async (req) => {
    const { readable, writable } = new TransformStream();
    renderToReadableStream(<App />, { 
      bootstrapScripts: ['/client.js'] 
    }).then(stream => stream.pipeTo(writable));
    return new Response(readable, { 
      headers: { 'Content-Type': 'text/html' } 
    });
  }
};
```

## Performance Impact

| Metric | Traditional SSR | Streaming SSR |
|--------|----------------|---------------|
| TTFB | After all data | Immediately (shell) |
| FCP | After TTFB | Much earlier |
| LCP | Depends on data | Progressive |
| TBT | Hydration cost | Selective hydration reduces it |

Real-world: Facebook reported significant LCP improvements moving to streaming. Vercel dashboards show ~2-3x FCP improvement for data-heavy pages.

## Trade-offs

| Pros | Cons |
|------|------|
| Faster perceived performance | Headers must be sent before shell — no late redirects |
| Progressive rendering | Status code committed before full render |
| Suspense boundaries isolate slow parts | Error handling after shell is complex |
| Out-of-order delivery | Some CDNs/proxies buffer streams |
| Selective hydration prioritization | Requires streaming-aware infrastructure |

## When to Use

✅ **Use when:**
- Pages with mix of fast and slow data (dashboards, feeds)
- React 18+ with Suspense
- Next.js App Router (built-in)
- Optimizing LCP and FCP for data-heavy pages

❌ **Avoid when:**
- Pages with a single critical data dependency (no benefit)
- Infrastructure that buffers HTTP responses
- Needing to redirect based on data (stream is already open)

---
*Related: [[React Server Components]], [[Hydration Strategies]], [[Rendering Strategies]], [[Edge Rendering]]*
