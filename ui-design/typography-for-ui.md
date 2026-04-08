# Typography for UI

> **Audience:** UI/UX designers and front-end engineers  
> **Last updated:** April 2026

---

## 1. Typographic Scale

A **typographic scale** is a predetermined set of font sizes with a consistent ratio, removing ad-hoc size decisions and ensuring visual harmony.

### Modular Scale Ratios

| Name | Ratio | Step sizes from 16px base |
|---|---|---|
| Major Second | 1.125 | 16, 18, 20.3, 22.8, 25.6 |
| Major Third | 1.250 | 16, 20, 25, 31.3, 39 |
| Perfect Fourth | 1.333 | 16, 21.3, 28.4, 37.9, 50.5 |
| Golden Ratio | 1.618 | 16, 25.9, 41.9, 67.8, 110 |

**Major Third (1.25×)** is the sweet spot for dense UIs (dashboards, admin panels) — sizes remain close enough to avoid jarring jumps.  
**Perfect Fourth (1.333×)** suits marketing and editorial pages where strong visual contrast between levels is desirable.  
**Golden Ratio** is too aggressive for body-heavy UIs; jumps become unmanageable after 3–4 steps.

```css
/* Perfect Fourth scale as CSS custom properties */
:root {
  --text-xs:   0.563rem;  /* ~9px  */
  --text-sm:   0.750rem;  /* 12px  */
  --text-base: 1rem;      /* 16px  */
  --text-lg:   1.333rem;  /* ~21px */
  --text-xl:   1.777rem;  /* ~28px */
  --text-2xl:  2.369rem;  /* ~38px */
  --text-3xl:  3.157rem;  /* ~51px */
}
```

> **Trade-off:** A strict modular scale can produce awkward fractional values (e.g., 28.43px). Round to the nearest 0.5px or 1px in production without losing perceptible harmony.

---

## 2. Font Pairing Strategies

Good pairings create contrast while maintaining coherence. Key axes of contrast: classification, weight, width, and x-height.

| Strategy | Example Pair | Use Case |
|---|---|---|
| **Serif + Sans** | Playfair Display + Inter | Editorial, long-form |
| **Variable weight contrast** | Inter 800 (headings) + Inter 400 (body) | Single-family product UI |
| **Geometric + Humanist** | Futura + Gill Sans | Brand-forward landing pages |
| **Slab Serif + Grotesque** | Roboto Slab + Roboto | Technical documentation |

**Rules of thumb:**
- Limit to **2 typefaces** (3 max, only if a monospace is needed for code).
- Match the *mood*, not just aesthetics — a playful display font on a fintech dashboard undermines trust.
- Verify both fonts render cleanly at small sizes (test at 12px on non-Retina screens).

---

## 3. Type Hierarchy: Display → Caption

A four-tier system covers 99% of UI surfaces:

```
Display    64–96px   Hero headlines, splash screens
Heading    H1–H6     Section titles, card headers
Body       14–18px   Paragraphs, list items, labels
Caption    10–12px   Timestamps, metadata, legal text
```

Hierarchy is expressed through **three levers in order of effectiveness**: size > weight > color. Avoid using color alone to convey hierarchy (accessibility violation).

```css
/* Semantic usage */
.display   { font-size: var(--text-3xl); font-weight: 700; line-height: 1.1; }
.heading-1 { font-size: var(--text-2xl); font-weight: 600; line-height: 1.2; }
.heading-2 { font-size: var(--text-xl);  font-weight: 600; line-height: 1.3; }
.body      { font-size: var(--text-base); font-weight: 400; line-height: 1.6; }
.caption   { font-size: var(--text-sm);  font-weight: 400; line-height: 1.5; color: var(--text-muted); }
```

---

## 4. Line Height and Measure

### Line Height (Leading)

| Text type | Recommended leading |
|---|---|
| Display / Hero | 1.0–1.2 (tight) |
| Headings | 1.2–1.4 |
| Body | 1.5–1.7 |
| Captions | 1.4–1.5 |

Tight leading on large display text is intentional — visual gaps between lines grow proportionally with font size. Loose leading on body text reduces eye fatigue during long reads.

### Measure (Line Length)

**Optimal measure:** 45–75 characters per line for body text (often cited as 66 chars as the ideal).

```css
/* Enforce measure on prose containers */
.prose {
  max-width: 65ch; /* ~65 characters wide */
}
```

Going beyond 85ch causes readers to lose their place when scanning back. Under 40ch causes excessive hyphenation and choppy rhythm. On mobile, 35–50ch is acceptable.

---

## 5. Variable Fonts

Variable fonts embed a full design space (weight, width, slant, optical size) in a single file, replacing multiple static files.

```css
/* Variable font with weight and optical size axes */
@font-face {
  font-family: 'Inter';
  src: url('Inter.var.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-display: swap;
}

.headline {
  font-variation-settings: 'wght' 750, 'opsz' 32;
}
.body {
  font-variation-settings: 'wght' 400, 'opsz' 16;
}
```

**Benefits:** Single HTTP request; smooth weight animations; optical sizing per context.  
**Trade-offs:** Variable font files are typically 150–400 KB (larger than a single weight subset) — always use `unicode-range` subsetting. Safari ≤ 13 has inconsistent `font-variation-settings` support.

---

## 6. System Font Stacks

System fonts load instantly (zero network cost) and match OS UI conventions. Ideal for application UIs where brand expression is secondary to performance.

```css
/* Modern system font stack */
body {
  font-family:
    system-ui,            /* CSS standard */
    -apple-system,        /* Safari / macOS / iOS */
    BlinkMacSystemFont,   /* Chrome on macOS */
    'Segoe UI',           /* Windows */
    Roboto,               /* Android / Chrome OS */
    Oxygen,               /* KDE */
    Ubuntu,               /* Ubuntu */
    Cantarell,            /* GNOME */
    sans-serif;
}

/* Monospace stack for code */
code {
  font-family: ui-monospace, 'Cascadia Code', 'Fira Code', 'Consolas', monospace;
}
```

> **Trade-off:** You lose cross-platform visual consistency. San Francisco (macOS) and Segoe UI (Windows) have different metrics — test both, especially if you rely on `ch` or `ex` units for layout.

---

## 7. Web Font Loading: FOIT, FOUT, and `font-display`

| Behavior | Description | User impact |
|---|---|---|
| **FOIT** (Flash of Invisible Text) | Browser hides text until font loads | Layout shift-free but content invisible |
| **FOUT** (Flash of Unstyled Text) | Fallback font shows, then swaps | Text readable immediately, but reflow |

### `font-display` Descriptor

```css
@font-face {
  font-family: 'MyFont';
  src: url('my-font.woff2') format('woff2');
  font-display: swap;     /* FOUT: show fallback, swap when ready */
  /* font-display: optional; — best for non-critical decorative fonts */
  /* font-display: block;   — short FOIT (avoid for body text) */
  /* font-display: fallback; — 100ms block, 3s swap window */
}
```

**Recommendation:**
- `swap` for body and heading fonts — content is readable immediately.
- `optional` for display/decorative fonts — if not cached, just use fallback forever.
- Match fallback font metrics using `size-adjust`, `ascent-override`, `descent-override` (CSS `@font-face` descriptors) to minimize reflow on swap.

```css
/* Reduce FOUT reflow for Inter → system-ui fallback */
@font-face {
  font-family: 'Inter-fallback';
  src: local('Arial');
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
  size-adjust: 107%;
}
```

---

## 8. Responsive Typography with `clamp()`

Fluid typography scales smoothly between viewport breakpoints without media queries.

```css
/* clamp(min, preferred, max) */
h1 {
  /* 2rem at 320px viewport, scales to 4rem at 1280px */
  font-size: clamp(2rem, 1.5rem + 2.5vw, 4rem);
}

body {
  font-size: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
}
```

**Formula:** `preferred = min + (max - min) * (100vw - minVW) / (maxVW - minVW)`

Tools like [Utopia.fyi](https://utopia.fyi) generate full fluid scales. The fluid approach replaces brittle breakpoint-based font size overrides and ensures text never overflows or becomes microscopic.

> **Trade-off:** `clamp()` with `vw` units breaks when the user increases their OS font size preference — always anchor the min to a relative unit (`rem`, not `px`) so accessibility scaling is preserved.

---

## 9. CJK (Chinese, Japanese, Korean) Considerations

CJK text requires special handling due to ideographic characters and distinct typographic conventions:

- **Font stacks must specify CJK fonts explicitly.** Generic `sans-serif` may fall back to CJK system fonts that differ radically by OS.
- **Line breaking:** CJK text breaks between any two characters. Use `word-break: break-all` sparingly; prefer `overflow-wrap: anywhere`.
- **Line height:** CJK glyphs are taller relative to Latin at the same font-size. Use `line-height: 1.8–2.0` for CJK body text (vs 1.6 for Latin).
- **Kerning/tracking:** Disable `letter-spacing` for CJK or keep it near zero — even 0.05em degrades CJK readability.
- **Minimum size:** 14px for CJK body text (vs 12px for Latin); complex kanji become unreadable below 12px.

```css
:lang(zh), :lang(ja), :lang(ko) {
  font-family: 'Noto Sans CJK SC', 'Source Han Sans', sans-serif;
  line-height: 1.9;
  letter-spacing: 0;
  word-break: normal;
  overflow-wrap: anywhere;
}
```

---

## 10. Accessibility

### Minimum Sizes (WCAG 2.1 / 3.0)

| Context | Minimum | Recommended |
|---|---|---|
| Body text | 12px (WCAG advisory) | 16px |
| UI labels / buttons | 11px | 14px |
| Caption / legal | 10px | 12px |
| CJK body | 14px | 16px |

Always use **`rem` not `px`** for font sizes so the user's browser font-size preference (`Ctrl`/`Cmd` `+`) is respected. `1rem = 16px` in default settings.

### Dyslexia-Friendly Typography

- **Font choice:** Prefer humanist sans-serifs (Inter, Atkinson Hyperlegible, Lexie Readable) over geometric or condensed fonts. OpenDyslexic is contentious — research is mixed.
- **Letter-spacing:** Slightly increased tracking (`letter-spacing: 0.05em`) on body text aids letter differentiation.
- **Word spacing:** `word-spacing: 0.1em` reduces crowding.
- **Avoid justified text** (`text-align: justify`) — uneven word spacing disrupts reading rhythm.
- **Line length:** Keep to 60–70ch; shorter lines reduce tracking errors.
- **Contrast:** Minimum 4.5:1 contrast ratio (WCAG AA); 7:1 for AAA. Off-white backgrounds (`#F8F8F8`) over pure white reduce glare.

```css
/* Accessibility-forward body text */
body {
  font-size: 1rem;           /* respects browser preference */
  line-height: 1.65;
  letter-spacing: 0.02em;
  word-spacing: 0.05em;
  max-width: 68ch;
  color: #1A1A2E;            /* near-black, not pure #000 */
  background: #FAFAFA;
}
```

---

## 11. Practical CSS Patterns Cheatsheet

```css
/* ── 1. Fluid scale with Utopia-style clamp ── */
:root {
  --step--1: clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem);
  --step-0:  clamp(1rem,    0.9rem + 0.5vw,  1.25rem);
  --step-1:  clamp(1.25rem, 1.1rem + 0.75vw, 1.75rem);
  --step-2:  clamp(1.5rem,  1.3rem + 1vw,    2.25rem);
  --step-3:  clamp(2rem,    1.6rem + 2vw,    3.5rem);
}

/* ── 2. Truncation utilities ── */
.truncate        { overflow: hidden; white-space: nowrap; text-overflow: ellipsis; }
.line-clamp-3    { display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; overflow: hidden; }

/* ── 3. Balanced headings (Chrome 114+) ── */
h1, h2, h3      { text-wrap: balance; }
p               { text-wrap: pretty; }   /* avoids orphans */

/* ── 4. Optical margin alignment for pull quotes ── */
blockquote      { hanging-punctuation: first last; }

/* ── 5. Numeric tabular figures for data tables ── */
.data-table td  { font-variant-numeric: tabular-nums; }

/* ── 6. Small caps for labels ── */
.label          { font-variant-caps: all-small-caps; letter-spacing: 0.08em; }

/* ── 7. Preload critical web font ── */
/* In <head>: <link rel="preload" as="font" href="/fonts/Inter.var.woff2" crossorigin> */
```

---

## Quick Decision Guide

```
Need zero-latency, system-native feel?    → System font stack
Need brand expression + performance?      → Variable font + font-display:swap + preload
Dense data UI (dashboard, table)?         → Major Third scale, tabular-nums, 14px base
Long-form editorial / blog?               → Perfect Fourth scale, 65ch measure, 1.65 leading
Multilingual / CJK product?              → Separate font stack per :lang(), 1.9 leading
Accessibility-first?                      → rem units, 16px body min, 4.5:1 contrast, no justify
```

---

## Further Reading

- [Utopia — Fluid Type Scale Calculator](https://utopia.fyi)
- [Font Face Observer](https://fontfaceobserver.com/) — programmatic font loading
- [Variable Fonts — v-fonts.com](https://v-fonts.com)
- [Atkinson Hyperlegible](https://brailleinstitute.org/freefont) — accessibility-first typeface
- WCAG 2.1 Success Criterion 1.4.4 (Resize Text), 1.4.12 (Text Spacing)
