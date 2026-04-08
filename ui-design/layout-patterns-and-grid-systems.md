---
title: "Layout Patterns & Grid Systems"
category: ui-design
summary: "A practical guide to CSS layout decisions — Grid vs Flexbox decision matrix, classic patterns (holy grail, sidebar, dashboard, card grid), spacing systems, visual rhythm, responsive grids, container queries, subgrid, aspect-ratio, and gap vs margin."
sources:
  - web-research
updated: 2026-04-08T18:25:00.000Z
---

# Layout Patterns & Grid Systems

> A practical guide to CSS layout decisions — Grid vs Flexbox decision matrix, classic patterns (holy grail, sidebar, dashboard, card grid), spacing systems, visual rhythm, responsive grids, container queries, subgrid, aspect-ratio, and gap vs margin.

---

## CSS Grid vs Flexbox — Decision Matrix

The single most-asked layout question: *which one do I use?* The short answer: **Grid owns 2D space, Flexbox owns 1D flow**. In practice you'll use both — often in the same component.

| Criterion | CSS Grid | Flexbox |
|-----------|----------|---------|
| **Axis** | Two-dimensional (rows **and** columns simultaneously) | One-dimensional (row **or** column) |
| **Content vs. container** | Container defines the track structure | Items size themselves; container distributes space |
| **Item placement** | Explicit cell/area placement | Implicit sequential flow |
| **Alignment control** | Full 2D: `justify-items`, `align-items`, `place-items` | Cross-axis only (`align-items`); main-axis is distribution |
| **Overlap / layering** | Native — items can share cells | Requires `position: absolute` hacks |
| **Responsive images/cards** | `auto-fill`/`auto-fit` + `minmax` = truly fluid columns | Needs `flex-wrap` with fixed `flex-basis` |
| **Navigation bars / toolbars** | Overkill | Natural fit |
| **Page-level layouts** | Ideal | Fragile for 2D |
| **Browser support** | Excellent (96%+) | Universal |

**Rule of thumb:**
- `nav`, `toolbar`, `button group`, `inline tag list` → **Flexbox**
- `page shell`, `card grid`, `form layout`, `dashboard` → **Grid**
- `card internals` (icon + text + actions) → **Flexbox inside a Grid cell**

---

## Spacing Systems — The 4px / 8px Base Unit

Consistent spacing is the foundation of visual rhythm. The industry-standard approach uses **8px as the base unit** (with 4px for tight micro-spacing), because most screens have pixel densities divisible by 8 and the scale maps cleanly to common device widths.

```css
/* Design token scale (multiples of 4px) */
:root {
  --space-1:  4px;   /* micro: icon padding, badge insets */
  --space-2:  8px;   /* tight: inline element gaps        */
  --space-3: 12px;   /* small: compact list items         */
  --space-4: 16px;   /* base: default component padding   */
  --space-5: 24px;   /* medium: section gaps              */
  --space-6: 32px;   /* large: card padding               */
  --space-7: 48px;   /* xl: section separation            */
  --space-8: 64px;   /* 2xl: hero / page margins          */
}
```

### Visual Rhythm

**Visual rhythm** is the perception of consistent cadence as the eye moves through a layout. Three practices enforce it:

1. **Vertical baseline grid** — type sits on a 4px or 8px grid. Line heights are multiples of the base unit.
2. **Proximity grouping** — related elements have `--space-2` gaps; unrelated sections have `--space-6`+. Whitespace communicates relationship.
3. **Consistent component padding** — every card, panel, and dialog uses the same internal padding token, never raw pixel values.

```css
/* Good: tokens preserve rhythm when scale changes */
.card { padding: var(--space-6); gap: var(--space-4); }

/* Bad: magic numbers break when base unit shifts */
.card { padding: 30px; gap: 18px; }
```

---

## Common Layout Patterns

### Holy Grail Layout

A header, footer, main content, and two sidebars — the layout that broke table-based design in the 2000s. CSS Grid makes it trivial:

```css
.holy-grail {
  display: grid;
  grid-template:
    "header  header  header"  auto
    "nav     main    aside"   1fr
    "footer  footer  footer"  auto
    / 200px  1fr     160px;
  min-height: 100vh;
}

header  { grid-area: header; }
nav     { grid-area: nav;    }
main    { grid-area: main;   }
aside   { grid-area: aside;  }
footer  { grid-area: footer; }
```

On mobile, redefine template areas — no JavaScript needed.

### Sidebar Layout

A persistent sidebar with a scrollable content area:

```css
.app-shell {
  display: grid;
  grid-template-columns: 260px 1fr;
  height: 100vh;
}

.sidebar { overflow-y: auto; }
.content { overflow-y: auto; }

/* Collapse sidebar on mobile */
@media (max-width: 768px) {
  .app-shell { grid-template-columns: 1fr; }
  .sidebar   { display: none; } /* or transform: translateX(-100%) */
}
```

### Dashboard Layout

Dashboards mix metric cards, charts, and tables — prime Grid territory:

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--space-5);
}

.metric-card  { grid-column: span 3; }   /* 4 cards per row */
.chart-main   { grid-column: span 8; }
.chart-side   { grid-column: span 4; }
.data-table   { grid-column: span 12; }  /* full width */

@media (max-width: 1024px) {
  .metric-card  { grid-column: span 6; }  /* 2 per row */
  .chart-main,
  .chart-side   { grid-column: span 12; } /* stack */
}
```

### Card Grid (Responsive, No Media Queries)

The `auto-fill` vs `auto-fit` pattern is one of CSS Grid's superpowers:

```css
/* auto-fill: preserves empty columns (tracks exist) */
.card-grid-fill {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: var(--space-5);
}

/* auto-fit: collapses empty columns (items stretch to fill) */
.card-grid-fit {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-5);
}
```

**When to use which:** `auto-fit` is correct for most card grids (cards grow to fill space). `auto-fill` is useful when you want a grid to maintain consistent column widths regardless of item count (e.g., a calendar).

---

## Responsive Grids — 12-Column System

The 12-column grid is divisible by 2, 3, 4, and 6 — maximally flexible for varied breakpoint layouts:

```css
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--space-5);
  /* optional max-width container */
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: var(--space-5);
}

/* Utility span classes */
.col-1  { grid-column: span 1;  }
.col-2  { grid-column: span 2;  }
.col-3  { grid-column: span 3;  }  /* quarter */
.col-4  { grid-column: span 4;  }  /* third */
.col-6  { grid-column: span 6;  }  /* half */
.col-8  { grid-column: span 8;  }  /* two-thirds */
.col-12 { grid-column: span 12; }  /* full */
```

---

## Container Queries

Media queries respond to the **viewport**; container queries respond to the **parent element's size** — crucial for truly reusable components.

```css
/* 1. Mark the container */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* 2. Query against the container, not the viewport */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 120px 1fr;  /* image + content side-by-side */
  }
}

@container card (max-width: 399px) {
  .card {
    flex-direction: column;            /* stacked on narrow containers */
  }
}
```

**Trade-off:** Container queries require a containment context, which can affect paint performance. Avoid nesting container queries more than two levels deep.

---

## Subgrid

Subgrid lets a child element **participate in its parent's grid tracks** — eliminating the misalignment problem in card grids where labels, images, and footers are different heights.

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  grid-template-rows: auto;          /* row tracks to share */
  gap: var(--space-5);
}

.card {
  display: grid;
  grid-row: span 3;                  /* claim 3 row tracks */
  grid-template-rows: subgrid;       /* inherit parent row heights */
}

/* Now .card__image, .card__body, .card__footer align across all cards */
.card__image  { /* row 1 */ }
.card__body   { /* row 2 */ }
.card__footer { /* row 3 — all footers at same baseline */ }
```

Browser support: Chrome 117+, Firefox 71+, Safari 16+. Use `@supports (grid-template-rows: subgrid)` for progressive enhancement.

---

## `aspect-ratio`

Maintain proportional sizing without padding hacks:

```css
/* Old hack */
.ratio-16-9 { padding-top: 56.25%; position: relative; }

/* Modern */
.video-embed { aspect-ratio: 16 / 9; width: 100%; }
.avatar      { aspect-ratio: 1;      width: 48px; }
.og-image    { aspect-ratio: 1200 / 630; width: 100%; }

/* Combine with object-fit for images */
.card__thumbnail {
  aspect-ratio: 4 / 3;
  width: 100%;
  object-fit: cover;    /* crops without distorting */
}
```

---

## `gap` vs `margin`

| | `gap` | `margin` |
|-|-------|---------|
| **Scope** | Between items in a flex/grid container | Any element, any context |
| **Bleeding** | Never bleeds outside the container | Margin collapse, last-item bleed |
| **Symmetry** | Automatically symmetric | Asymmetric unless carefully managed |
| **Negative spacing** | Not supported | Supported (negative margins) |
| **Best for** | Spacing between grid/flex children | External spacing, offset tricks |

**Recommendation:** Use `gap` for all internal component spacing. Use `margin` only for pushing a component away from its sibling context (e.g., `margin-top: auto` to push a footer to the bottom of a flex column).

---

## Layout Composition Patterns

### Stack

A vertical flex container — the workhorse of component internals:

```css
.stack {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
}
```

### Cluster

Inline elements that wrap naturally (tags, badges, buttons):

```css
.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  align-items: center;
}
```

### Sidebar Composition (Intrinsic)

A sidebar that wraps below content when it doesn't fit, without a media query:

```css
.with-sidebar {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-5);
}

.sidebar-child  { flex-basis: 240px; flex-grow: 1; }
.content-child  { flex-basis: 0;     flex-grow: 999; min-width: 60%; }
```

When `content-child` can't maintain 60% of the container, the sidebar wraps below — pure intrinsic layout, no breakpoints.

### Center

A horizontally and vertically centered content column with max-width:

```css
.center {
  box-sizing: content-box;
  max-width: 65ch;           /* ~readable line length */
  margin-inline: auto;
  padding-inline: var(--space-5);
}
```

---

## Trade-offs Summary

| Approach | Wins | Loses |
|----------|------|-------|
| **CSS Grid (explicit)** | Precise 2D placement, overlapping elements | Can be brittle with dynamic content counts |
| **CSS Grid (intrinsic)** | Truly responsive, no breakpoints | Less predictable at extreme sizes |
| **Flexbox** | Content-driven sizing, one-dimensional | Complex 2D alignment needs workarounds |
| **Container queries** | Portable component responsiveness | Adds containment overhead, newer API |
| **Subgrid** | Cross-card alignment without JS | Browser support requires progressive enhancement |
| **12-column utility grid** | Designer–developer shared language | Semantic class names can leak into markup |

---

## Key Takeaways

1. **Default to `gap` over `margin`** inside components — it's predictable and symmetric.
2. **Use the 8px base unit** and map all spacing to tokens — magic numbers kill rhythm.
3. **`auto-fit` + `minmax`** eliminates most card-grid breakpoints and is the modern baseline.
4. **Subgrid** solves the card-footer-alignment problem elegantly — use it where supported.
5. **Container queries** are production-ready; prefer them over viewport media queries for component-level responsiveness.
6. **Grid areas with named template** (`grid-template-areas`) are the clearest way to document page-level layouts for both designers and developers.
