---
title: "Mobile-First & Responsive Design"
category: ui-design
summary: "A comprehensive guide to mobile-first philosophy, breakpoint strategies, touch targets, responsive images, container queries, navigation patterns, thumb zones, viewport units, progressive enhancement, and responsive testing."
sources:
  - web-research
updated: 2026-04-08T18:25:00.000Z
---

# Mobile-First & Responsive Design

> A comprehensive guide to mobile-first philosophy, breakpoint strategies, touch targets, responsive images, container queries, navigation patterns, thumb zones, viewport units, progressive enhancement, and responsive testing.

## The Mobile-First Philosophy

**Mobile-first** (Ethan Marcotte popularized responsive design in 2010; Luke Wroblewski codified mobile-first in 2011) is a *design and development constraint*: start with the smallest viewport and most limited context, then progressively enhance toward larger screens.

The underlying principle is **progressive disclosure** — a small screen forces you to decide what truly matters. If a feature cannot survive on a 375px screen, it probably wasn't essential to begin with.

### Desktop-First vs. Mobile-First

| Dimension | Desktop-First | Mobile-First |
|-----------|--------------|--------------|
| CSS direction | Starts complex, uses `max-width` queries to strip | Starts minimal, uses `min-width` queries to add |
| Cognitive load | Easy to design; hard to cut features | Forces prioritization upfront |
| Performance | Mobile downloads desktop CSS then overrides | Mobile downloads only base CSS |
| Media query style | `@media (max-width: 768px) { … }` | `@media (min-width: 768px) { … }` |
| Default experience | Desktop users; mobile degrades | Mobile users; desktop enhances |

```css
/* ❌ Desktop-first — mobile must undo styles */
.nav { display: flex; gap: 24px; }
@media (max-width: 768px) {
  .nav { display: none; }   /* undoing work */
}

/* ✅ Mobile-first — desktop adds styles */
.nav { display: none; }     /* hidden by default on small screens */
@media (min-width: 768px) {
  .nav { display: flex; gap: 24px; }
}
```

---

## Breakpoint Strategies

### Content-Driven vs. Device-Driven

| Strategy | How | Pros | Cons |
|----------|-----|------|------|
| **Device-driven** | Match common device widths (320, 768, 1024, 1280) | Predictable; matches design tool artboards | Breaks when new devices appear; arbitrary |
| **Content-driven** | Add a breakpoint when the content *breaks* | Future-proof; semantic; design-led | Requires more design iteration |

**Prefer content-driven breakpoints.** Resize the browser until the layout looks wrong, then add a breakpoint — not before.

```css
/* Major breakpoints (rough guidelines, not rules) */
/* xs: 0–479 — phones portrait         */
/* sm: 480–767 — phones landscape       */
/* md: 768–1023 — tablets               */
/* lg: 1024–1279 — small laptops        */
/* xl: 1280+ — desktops                 */

/* Tailwind-style utility: customize in tailwind.config.js */
screens: {
  sm: '480px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px',
}
```

---

## Touch Targets

Google Material and Apple HIG both mandate a **minimum 48×48 dp/pt touch target**, even if the visual element is smaller. Tap targets that are too small cause mis-taps and frustration.

```css
/* Icon button — visually 24px, tap target 48px */
.icon-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;   /* ensures tap area, not just the icon */
  padding: 12px;  /* alternative: use padding to enlarge */
}
```

**Rules of thumb:**
- Minimum **48×48 px** tap target; 44px on iOS per HIG.
- At least **8px gap** between adjacent interactive elements to prevent accidental taps.
- Never rely on `hover` for revealing interactive affordances on touch devices; use `:focus-visible` + visible labels.

---

## Responsive Images

### `srcset` and `sizes` (Resolution Switching)

Used when the *same image* is served at different resolutions — the browser picks the best fit.

```html
<img
  src="hero-800.jpg"
  srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 800px"
  alt="Mountain landscape"
  loading="lazy"
/>
```

### `<picture>` and Art Direction

Used when the *composition* of the image should change at different breakpoints — crop differently, use a portrait vs. landscape shot.

```html
<picture>
  <!-- Tall crop for phones -->
  <source media="(max-width: 767px)" srcset="hero-portrait.webp" type="image/webp" />
  <!-- Wide crop for desktops -->
  <source media="(min-width: 768px)" srcset="hero-landscape.webp" type="image/webp" />
  <!-- Fallback -->
  <img src="hero-landscape.jpg" alt="City skyline" />
</picture>
```

| Technique | Use When | Key Attribute |
|-----------|----------|---------------|
| `srcset` + `sizes` | Same image, different resolutions/widths | `sizes` tells browser the rendered width |
| `<picture>` art direction | Different crops, orientations, or subjects | `<source media="…">` |
| `<picture>` format switching | Serve WebP/AVIF with JPEG fallback | `<source type="image/avif">` |

---

## Fluid vs. Adaptive Layouts

| Model | Mechanism | Behaviour |
|-------|-----------|-----------|
| **Fluid (Responsive)** | Percentage widths, `fr` units, `clamp()` | Layout stretches/shrinks continuously |
| **Adaptive** | Fixed-width containers that snap at breakpoints | Layout jumps between defined widths |
| **Hybrid** | Fluid columns inside adaptive breakpoint ranges | Most common in production |

```css
/* Fluid typography with clamp() — scales between 1rem and 1.5rem */
h1 { font-size: clamp(1rem, 2.5vw + 0.5rem, 1.5rem); }

/* Fluid grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}
```

---

## Container Queries vs. Media Queries

**Media queries** respond to the *viewport*. **Container queries** respond to the *parent container's* size — enabling truly reusable components.

```css
/* Register a containment context */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* Component adapts to its container, not the viewport */
@container card (min-width: 400px) {
  .card { flex-direction: row; }
}
@container card (max-width: 399px) {
  .card { flex-direction: column; }
}
```

| Feature | Media Query | Container Query |
|---------|-------------|-----------------|
| Responds to | Viewport width/height/feature | Parent container size |
| Reusability | Component tied to viewport assumptions | Component is context-agnostic |
| Browser support | Universal | Chrome 105+, Firefox 110+, Safari 16+ |
| Use for | Page-level layout, global breakpoints | Component-level layout, design system atoms |

**Rule:** Use media queries for macro layout (page structure); use container queries for micro layout (component internals).

---

## Responsive Tables

Tables are notoriously hard to make responsive. Three patterns cover most cases:

| Pattern | Technique | Best For |
|---------|-----------|---------|
| **Horizontal scroll** | `overflow-x: auto` on wrapper | Data grids where all columns matter |
| **Column priority / hide** | Hide less-important columns at small widths | Reports with many secondary columns |
| **Card transform** | Each row becomes a card with label–value pairs | Small tables (< 5 rows) with meaningful labels |

```css
/* Pattern 1: Horizontal scroll */
.table-wrapper { overflow-x: auto; -webkit-overflow-scrolling: touch; }

/* Pattern 3: Card transform */
@media (max-width: 600px) {
  table, thead, tbody, th, td, tr { display: block; }
  thead tr { position: absolute; top: -9999px; left: -9999px; } /* visually hide header */
  td::before {
    content: attr(data-label);  /* set data-label="Column Name" on each <td> */
    font-weight: bold;
    display: inline-block;
    width: 8rem;
  }
}
```

---

## Navigation Patterns

| Pattern | Description | Best For | Trade-offs |
|---------|-------------|----------|------------|
| **Hamburger menu** | Icon opens a drawer/overlay | Many nav items; desktop carries full nav | Hidden navigation reduces discoverability |
| **Bottom navigation bar** | 3–5 icons fixed to screen bottom | Primary app destinations | Limited to ~5 items; competes with browser chrome |
| **Tab bar** | Tabs across top or segmented | Section switching within a screen | Takes vertical space; doesn't work for many sections |
| **Priority+ / "More" overflow** | Show top items, overflow into a dropdown | Desktop–tablet transition | Requires dynamic measurement |

**Thumb Zone (Steven Hoober's research):** On a phone held one-handed, the natural thumb arc covers the bottom-center; the top corners are hardest to reach.

```
┌─────────────────┐
│  ❌ Hard reach  │  ← avoid primary actions here
│    ⚠️ OK        │
│  ✅ Easy reach  │  ← place primary CTA here
│  ✅ Easy reach  │
└─────────────────┘
         Bottom navigation lives here — prime real estate
```

Design implications:
- Place primary CTAs (Compose, Add, Confirm) in the bottom third.
- Avoid top-right placement for frequently used actions on mobile.
- Floating Action Buttons (FABs) at bottom-center capitalize on thumb-zone reach.

---

## Viewport Units: `dvh`, `svh`, `lvh`

Classic `100vh` on mobile often means "100% of the viewport including the browser toolbar," causing layout overflow. Three new units solve this:

| Unit | Stands For | Behavior |
|------|------------|----------|
| `svh` | Small Viewport Height | 100% of the *smallest* viewport (toolbar fully shown) |
| `lvh` | Large Viewport Height | 100% of the *largest* viewport (toolbar hidden) |
| `dvh` | Dynamic Viewport Height | Updates dynamically as toolbars appear/disappear |
| `100vh` | Legacy | Inconsistent on mobile; freezes at initial value |

```css
/* Full-height hero that avoids toolbar overlap */
.hero {
  min-height: 100svh; /* safe: toolbar-aware, no jumpy reflow */
}

/* Full-screen interactive app (reflows with toolbar) */
.app-shell {
  height: 100dvh;
}
```

**Recommendation:** Use `svh` for static full-height sections to avoid reflow. Use `dvh` for app shells that need to truly fill the screen at all times. Avoid raw `100vh` on mobile.

---

## Progressive Enhancement

Progressive enhancement layers capability on a solid baseline:

```
1. Semantic HTML  →  works everywhere, accessible, indexable
2. CSS Layout     →  visual hierarchy, responsive structure
3. CSS Enhance    →  animations, custom properties, container queries
4. JavaScript     →  interactivity, data fetching, offline support
```

Build every feature so it works without JS; use JS to *enhance*, not to *enable*. Gate CSS features with `@supports` so unsupported browsers get the baseline:

```css
/* Base: all browsers */
.layout { display: flex; flex-wrap: wrap; }

/* Enhancement: browsers with container query support */
@supports (container-type: inline-size) {
  .card-wrapper { container-type: inline-size; }
}
```

---

## Responsive Testing

| Method | Tool | What It Catches |
|--------|------|-----------------|
| **Browser DevTools responsive mode** | Chrome / Firefox DevTools | Quick visual check at preset device sizes |
| **Real device testing** | iOS Safari, Android Chrome | Touch accuracy, toolbar behavior, font scaling |
| **Cloud device farms** | BrowserStack, Sauce Labs | OS/browser/device matrix |
| **Visual regression** | Percy, Chromatic, Playwright | Layout regressions across breakpoints |
| **Accessibility + responsive** | axe DevTools, Lighthouse | Tap targets, zoom-to-400%, reflow at 320px |

**WCAG 1.4.10 Reflow** requires content to present without horizontal scrolling at 320 CSS pixels wide (1280px viewport at 400% zoom) — this is the responsive accessibility baseline.

**Pre-ship checklist:**
- [ ] No horizontal scroll at 320px
- [ ] All touch targets ≥ 48×48 px
- [ ] Images use `srcset`/`<picture>` or are appropriately sized
- [ ] Navigation accessible at all breakpoints
- [ ] `svh`/`dvh` used instead of `100vh` for full-height elements
- [ ] Tested on real iOS Safari and Android Chrome

---

## Key Takeaways

1. **Write CSS mobile-first** with `min-width` queries — performance and prioritization both benefit.
2. **Break at content, not device** — add breakpoints where the layout breaks, not at arbitrary device widths.
3. **48px minimum touch target** — pad small elements; don't just make the visible hit zone tiny.
4. **Prefer `svh`/`dvh` over `100vh`** on mobile to sidestep browser toolbar issues.
5. **Container queries for components, media queries for pages** — keeps components reusable and context-agnostic.
6. **Progressive enhancement beats graceful degradation** — a solid HTML/CSS baseline ensures resilience; JS is an enhancement layer, not a requirement.
