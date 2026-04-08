# Dashboard & Data Visualization UX

> **Category:** UI Design · **Last Updated:** 2026-04-08
> **Tags:** dashboards, data-viz, charts, KPI, accessibility, Tufte, UX, analytics

---

## Overview

Dashboards translate raw data into decisions. When designed well, they surface signal above noise, guide users toward insight, and compress hours of analysis into seconds of comprehension. Designed poorly, they become "data graveyards" — dense, decorative, and ignored.

This article covers the full UX stack for dashboards: from high-level type classification through chart selection, layout, color, accessibility, and real-time patterns.

---

## Dashboard Types

Every dashboard design decision starts with *what job it does*. Mismatching a user's goal with the wrong dashboard type is the most common root failure.

| Type | Time Horizon | Primary User | Refresh Rate | Example |
|---|---|---|---|---|
| **Operational** | Now / today | Frontline staff, on-call | Seconds–minutes | Support queue status, server health, live orders |
| **Analytical** | Past weeks/months | Analysts, PMs | Hours–daily | Funnel analysis, cohort retention, A/B results |
| **Strategic** | Quarters / years | Executives, leadership | Daily–weekly | Revenue trend, market share, OKR progress |

**Design implications by type:**
- **Operational:** Prioritize anomaly detection. Use alerts, status indicators, and thresholds. Avoid clutter — when something breaks, users must see it in <2 seconds.
- **Analytical:** Allow exploration. Expose filters, drill-downs, and cross-chart interactions. Density is acceptable because users are in investigation mode.
- **Strategic:** Minimize detail, maximize narrative. Use large numbers, trend arrows, and sparklines. Executives skim; every pixel must earn its place.

---

## Information Density vs. Cognitive Load

**Cognitive load** is the mental effort required to process a display. More data ≠ more value. There is a U-shaped optimum: too sparse and users can't draw conclusions; too dense and they give up.

**Practical density rules:**
- **Above the fold:** Limit to 5–7 KPI cards or charts. Beyond 7 items, working memory saturates (Miller's Law).
- **Progressive disclosure:** Show summaries at the top; expose detail on demand via drill-down, tabs, or expand controls.
- **White space is data:** Grouping via spatial proximity is a pre-attentive signal — don't sacrifice it for one more chart.
- **Avoid dual-axis charts** unless unavoidable. They require the brain to hold two scales simultaneously, doubling cognitive load.

---

## Chart Type Selection

Choosing the wrong chart type miscommunicates data even when the underlying numbers are correct.

| Chart | Use When | Avoid When |
|---|---|---|
| **Bar (vertical/horizontal)** | Comparing discrete categories | More than ~15 categories; continuous data |
| **Line** | Showing change over continuous time | Fewer than 3 data points; categorical x-axis |
| **Pie / Donut** | Part-to-whole with ≤5 slices | More than 5 categories; precise comparison needed |
| **Scatter** | Correlation between two variables | Categorical axes; sparse data needing trend lines |
| **Heatmap** | Two-dimensional patterns, density, calendar activity | Precise value reading required |
| **Treemap** | Hierarchical proportions (nested categories) | Deeply nested hierarchies with many small leaves |
| **Area** | Cumulative totals or stacked composition over time | Overlapping areas that obscure each other |
| **Histogram** | Distribution of a single continuous variable | Ordinal or nominal categories |

**Decision heuristic:**
1. *What relationship am I showing?* (comparison / composition / distribution / relationship / trend)
2. *How many variables?* (1, 2, multi)
3. *Is the x-axis time-based?* → Line or area; otherwise bar.
4. *Do parts sum to 100%?* → Pie (small n) or stacked bar (larger n).

---

## Data-Ink Ratio (Tufte)

Edward Tufte's **data-ink ratio** = (ink used to encode data) ÷ (total ink on the graphic). The goal: maximize it by removing non-data ink without losing information.

**Apply it:**
- Remove gridlines or reduce to light grey — let data carry the weight.
- Eliminate chart borders ("chartjunk").
- Drop unnecessary legends when labels are applied directly to lines/bars.
- Remove tick marks if axis labels are self-explanatory.
- Don't use 3D charts — the third dimension adds ink but zero data.

```
❌ High chartjunk:          ✅ High data-ink ratio:
- Bold axis borders         - No border
- Dark gridlines            - Light grey gridlines only
- 3D bars                   - Flat bars
- Legend box with border    - Direct labels on series
- Background fill           - White or transparent bg
```

Tufte's principle doesn't demand austerity — it demands *purpose*. Every visual element should either encode data or aid comprehension of data.

---

## Color Usage in Charts

### Palette Types

| Palette | Use Case | Example Tool |
|---|---|---|
| **Categorical** | Distinguishing unrelated groups (e.g., product lines) | D3 `schemeTableau10`, Material palette |
| **Sequential** | Encoding magnitude of a single variable (low → high) | ColorBrewer `Blues`, Viridis |
| **Diverging** | Values with a meaningful midpoint (e.g., profit/loss, sentiment) | ColorBrewer `RdBu`, `PiYG` |

**Rules:**
- Use **≤8 categorical colors** per chart. Beyond that, perception degrades; switch to patterns or labels.
- **Never encode data with hue alone** — vary lightness and saturation too, so the chart works in greyscale and for colorblind users.
- For sequential scales, **avoid rainbow** (jet colormap) — it introduces false perceptual boundaries and is not colorblind-safe. Prefer perceptually uniform scales: Viridis, Cividis, Plasma.
- Use **semantic color conventions** where established: red = danger/loss, green = success/gain, yellow = warning. Don't invert them.
- Mute non-selected series on hover with 30–40% opacity rather than hiding them — preserves spatial context.

### Colorblind Safety
~8% of men have red-green color vision deficiency. Test palettes with tools like Coblis or the Figma A11y plugin. Use **blue-orange** or **blue-red** instead of red-green for pass/fail distinctions.

---

## Responsive Charts

Charts on mobile require deliberate adaptation — they are not auto-scaled desktop charts.

```
Desktop (1200px+)   →  Full chart with legend, annotations
Tablet (768-1199px) →  Simplified chart, legend below
Mobile (<768px)     →  Single KPI stat or sparkline; swipe to expand
```

**Responsive patterns:**
- **Aspect ratio locks:** Maintain a consistent height/width ratio (e.g., 16:9) so charts don't become unreadably tall on narrow screens.
- **Axis label rotation:** Rotate x-axis labels 45° or replace with abbreviated labels below ~400px.
- **Reduce data density:** Show weekly aggregates on mobile instead of daily points.
- **Touch targets:** Make interactive chart elements (bars, dots) ≥44×44px minimum.
- **Scrollable tables:** Never force horizontal scroll on the whole page; wrap tables in a scrollable container.

---

## Number Formatting and Units

Unformatted numbers destroy trust and require mental arithmetic from users.

```
❌  1283947.23          ✅  1.28M
❌  0.03421             ✅  3.4%
❌  86400 seconds       ✅  24 hours
❌  $1,000,000.00       ✅  $1.0M
❌  0.992341234         ✅  99.2%
```

**Formatting rules:**
- Round to **2–3 significant figures** for KPI cards; expose full precision in data tables.
- Abbreviate: K (thousands), M (millions), B (billions). Always suffix units.
- Show **delta indicators** (▲ 12% vs last period) alongside absolute values for context.
- Currency: match locale. Use symbol prefix for USD/EUR/GBP; suffix for many Asian currencies.
- Timestamps: use relative time ("3 hours ago") for real-time dashboards; absolute ISO-8601 for analytical ones.

---

## KPI Card Design

KPI cards are the highest-value pixel real estate on any dashboard.

```
┌─────────────────────────┐
│  Monthly Revenue         │
│                          │
│  $2.4M    ▲ 14.2%        │
│           vs last month  │
│  ▁▂▃▄▅▆▇█  (sparkline)  │
└─────────────────────────┘
```

**Anatomy:**
1. **Label** — concise, unambiguous metric name (avoid jargon).
2. **Primary value** — large, high-contrast, abbreviated.
3. **Delta** — change vs prior period with direction arrow and color (green/red).
4. **Comparison context** — "vs last month", "vs target", "YoY".
5. **Sparkline** (optional) — micro-trend without axes; shows direction, not exact values.

**Anti-patterns:** Omitting the comparison period ("revenue went up!" — from what baseline?); using emoji or icons that don't encode information; making the card clickable without a visible affordance.

---

## Drill-Down Patterns

Drill-down bridges strategic summaries and analytical detail.

| Pattern | Mechanism | Best For |
|---|---|---|
| **Click-to-filter** | Clicking a bar filters other charts | Cross-chart exploration |
| **Expand-in-place** | Chart expands to show sub-categories inline | Hierarchical data |
| **Modal / drawer detail** | Click opens a focused detail view | Record-level data |
| **Page navigation** | Click routes to a dedicated sub-dashboard | Deep hierarchies |
| **Tooltip enrichment** | Hover reveals supplementary metrics | Non-disruptive quick detail |

Always provide a **breadcrumb or back affordance** so users know how deep they've drilled and can return. Don't trap users in a drilled state on refresh — preserve state in the URL query string.

---

## Real-Time Dashboards

Real-time dashboards (operational) have distinct UX requirements from batch-refresh dashboards.

**Technical considerations surfacing as UX:**
- **WebSockets vs. polling:** WebSocket push reduces latency and eliminates wasted requests; use SSE for unidirectional streams.
- **Update frequency:** Animate updates at ≤1Hz visible rate — faster refresh creates anxiety and motion sickness, not insight.
- **Stale data indicators:** Show a "last updated" timestamp. If the connection drops, surface a visible warning (`⚠ Data paused — reconnecting`).
- **Streaming charts:** Use a **rolling window** (last N minutes) not ever-growing axes — the oldest data scrolls off.
- **Buffering spikes:** Debounce UI re-renders to 250–500ms to avoid jank during data bursts.

---

## Filtering and Date Range Patterns

Filters are how users direct their analysis. Poor filter UX bottlenecks every chart.

**Date range patterns:**
```
Presets:  [Today] [Last 7d] [Last 30d] [This Quarter] [Custom ▼]
Custom:   [Apr 1, 2026] → [Apr 8, 2026]  [Apply]
```

- Always show **presets** alongside a custom picker — 80% of selections use presets.
- Display the **active date range visibly** at all times (not just in the picker).
- On change, indicate loading state per-chart — don't freeze the whole page.
- **Relative vs. absolute ranges:** Default to relative ("last 30 days") so the dashboard stays current; let power users pin absolute ranges.

**Filter UX rules:**
- Show **active filter count** as a badge when the filter panel is collapsed.
- Provide a **"Clear all filters"** affordance.
- Apply filters **immediately** (or within 300ms) when possible — avoid requiring an explicit "Apply" button for simple toggles.
- Persist filter state in the URL for shareability and bookmark support.

---

## Table vs. Chart Decision

| Situation | Use Table | Use Chart |
|---|---|---|
| Precise values matter | ✅ | ❌ |
| Comparison of exact numbers | ✅ | ❌ |
| Trend or pattern over time | ❌ | ✅ |
| Part-to-whole relationship | ❌ | ✅ |
| Many rows, few columns | ✅ | ❌ |
| Anomaly detection at a glance | ❌ | ✅ |
| Mixed numeric + text fields | ✅ | ❌ |

**Hybrid:** Use **inline sparklines** in table cells to get the precision of a table with the trend signal of a chart.

---

## Sparklines

Sparklines are miniature charts embedded in context — typically 60–120px wide, no axes, no labels.

- **Best for:** Trend direction in KPI cards, table rows showing historical values, status panels.
- **Not for:** Exact value reading — they communicate shape, not magnitude.
- **Implementation tip:** Normalize sparklines to their own min/max, not a shared axis, so each line fills its container and shows its own trend clearly.

```html
<!-- SVG inline sparkline example -->
<svg viewBox="0 0 60 20" width="60" height="20">
  <polyline points="0,18 10,14 20,10 30,12 40,6 50,4 60,2"
            fill="none" stroke="#3b82f6" stroke-width="1.5"/>
</svg>
```

---

## Annotation and Context

Raw numbers become meaningful with context. Annotations explain the *why* behind the *what*.

**Annotation types:**
- **Reference lines:** "Target: 10,000 users/day" drawn as a horizontal dashed line.
- **Event markers:** Vertical lines at known events ("Launched feature X", "Marketing campaign").
- **Threshold bands:** Shaded zones for acceptable ranges (e.g., green band = SLA compliance zone).
- **Callout labels:** Direct text annotations on anomalies.

Keep annotations subtle — light grey, small type — so they provide context without competing with data.

---

## Dashboard Layout Patterns

| Layout | Structure | Best For |
|---|---|---|
| **KPI-first (scannable)** | Top row of KPI cards → charts below | Executive dashboards, at-a-glance status |
| **Grid** | Uniform card grid with drag-to-resize | Customizable analytics platforms (Grafana, Tableau) |
| **Magazine** | Asymmetric, editorial hierarchy | Public-facing data stories, reports |
| **Sidebar filter + canvas** | Fixed left filter panel + scrollable chart area | Analytical tools with heavy filtering |

**Grid guidelines:**
- Use a **12-column grid** (same as CSS frameworks) so charts can span 3, 4, 6, or 12 columns predictably.
- Group related charts spatially — proximity implies relationship.
- Lead with the most important metric; users read F-pattern (top-left to right, then down).

---

## Dark Mode for Data Viz

Dark mode requires rethinking chart colors, not just inverting backgrounds.

**Adjustments needed:**
- **Lower saturation** for chart series colors — saturated hues on dark backgrounds cause halation (colors appear to bleed/glow).
- **Increase lightness** slightly for lines and borders — what's visible on white disappears on dark grey.
- **Gridlines:** Switch from `#e5e7eb` (light mode) to `rgba(255,255,255,0.1)` (dark mode).
- **Avoid pure black** (#000) backgrounds — use `#111827` or `#1e293b` to reduce contrast harshness and make colors pop without glare.
- Test each palette separately in dark mode — don't assume a light-mode palette auto-adapts.

```css
/* Chart color tokens with dark mode */
:root {
  --chart-grid:    #e5e7eb;
  --chart-series-1: #3b82f6;
}
[data-theme="dark"] {
  --chart-grid:    rgba(255,255,255,0.08);
  --chart-series-1: #60a5fa; /* lighter blue */
}
```

---

## Accessibility in Charts

Charts are notoriously inaccessible by default. Screen readers can't read a `<canvas>` element.

**Techniques:**
- **Alt text on chart images:** Summarize the key insight, not the visual description.
  ```html
  <img src="revenue-chart.png"
       alt="Monthly revenue grew 40% from $1.7M in Jan to $2.4M in Jun 2026">
  ```
- **Data table fallback:** Provide a hidden (but focusable) `<table>` with the raw data behind every chart. Users can toggle it visible.
- **ARIA roles:** Use `role="img"` on SVG charts with a `<title>` and `<desc>` element inside.
  ```html
  <svg role="img" aria-labelledby="chart-title chart-desc">
    <title id="chart-title">Q2 Revenue by Region</title>
    <desc id="chart-desc">Bar chart showing APAC led with $1.2M...</desc>
  </svg>
  ```
- **Patterns + color:** Don't rely on color alone. Add fill patterns (hatching, dots) as a secondary encoding channel for colorblind users.
- **Keyboard navigation:** Interactive chart elements (drill-downs, tooltips) must be reachable and activatable via keyboard.
- **Focus indicators:** Visible focus ring on chart series, data points, and legend items.
- **Contrast:** Chart labels and axis text must meet WCAG AA (4.5:1 for normal text, 3:1 for large text / UI elements).

---

## Quick Reference: Chart Selection Cheat Sheet

```
What are you showing?
│
├── Change over time?          → LINE (continuous) / BAR (discrete periods)
├── Comparison of categories?  → BAR (vertical preferred; horizontal for long labels)
├── Part of a whole?           → PIE ≤5 parts / STACKED BAR >5 parts
├── Correlation / relationship?→ SCATTER (2 vars) / BUBBLE (3 vars)
├── Distribution / spread?     → HISTOGRAM / BOX PLOT
├── Geographic data?           → CHOROPLETH MAP / SYMBOL MAP
├── Two-variable density?      → HEATMAP
└── Hierarchical proportion?   → TREEMAP / SUNBURST
```

---

## Trade-Off Summary

| Decision | Option A | Option B | Choose When |
|---|---|---|---|
| Bar vs. Line | Bar | Line | Discrete categories vs. continuous time |
| Pie vs. Bar | Pie | Bar | ≤5 slices, rough comparison vs. precise comparison |
| Chart vs. Table | Chart | Table | Pattern/trend vs. exact values needed |
| Real-time vs. Batch | Real-time | Batch refresh | Operational SLA monitoring vs. analytical review |
| Dark mode colors | Lower saturation | Keep light-mode palette | Always recalibrate for dark backgrounds |
| Single axis vs. dual axis | Single | Dual | Prefer single; dual only if variables share units |

---

*See also: [Color Theory & Palette Design](./color-theory-and-palette-design.md) · [Accessibility (WCAG)](./accessibility-wcag.md) · [Layout Patterns & Grid Systems](./layout-patterns-and-grid-systems.md)*
