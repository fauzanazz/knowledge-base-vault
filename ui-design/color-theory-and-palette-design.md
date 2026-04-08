# Color Theory & Palette Design for UI

> **Category:** UI Design · **Last Updated:** 2026-04-08
> **Tags:** color, design-systems, accessibility, CSS, theming, dark-mode

---

## Overview

Color is one of the most powerful levers in UI design — it communicates hierarchy, encodes meaning, guides attention, and defines brand. Done well, a coherent color system makes an interface feel polished and predictable. Done poorly, it creates visual noise, accessibility failures, and inconsistency across components.

This article covers the full stack: from the mathematics of color models, through palette generation strategies, semantic color roles, accessibility requirements, dark mode design, and practical CSS implementation.

---

## Color Models

Understanding *how* colors are represented is foundational before designing with them.

### RGB
The native model for screens. Each channel (Red, Green, Blue) ranges 0–255 or 0–1. RGB is **device-oriented** — values map directly to pixel output — but it is deeply **non-intuitive** for designers. Moving a slider from `rgb(200, 50, 50)` to `rgb(200, 100, 50)` doesn't feel like a predictable perceptual shift.

### HSL (Hue / Saturation / Lightness)
A cylindrical re-mapping of RGB designed to be more human-readable. Hue is a 0–360° angle on the color wheel, Saturation is 0–100%, Lightness is 0–100%. HSL is far easier to reason about than raw RGB and is supported natively in CSS (`hsl(220, 70%, 55%)`).

**Trade-off:** HSL is *mathematically* uniform but *perceptually* non-uniform. Two colors with the same `L` value can appear dramatically different in perceived brightness — a yellow at `L: 50%` looks much brighter than a blue at `L: 50%`. This makes it unreliable for building accessible contrast systems.

### LCH (Lightness / Chroma / Hue)
LCH is based on the CIELAB color space, which was engineered to be **perceptually uniform**: equal numerical steps correspond to equal perceived differences. LCH separates:
- **L** — perceived lightness (0 = black, 100 = white)
- **C** — chroma (colorfulness/saturation)
- **H** — hue angle

LCH lets you rotate hue while keeping perceived brightness constant — a critical property for generating accessible tonal scales. CSS now supports `lch()` natively (CSS Color Level 4).

### OKLCH
OKLCH is an improved version of LCH developed by Björn Ottosson (also the creator of the Oklab space). It fixes some hue-shifting artifacts present in LCH, particularly in the blues/purples range. **OKLCH is now the recommended model for design systems** that require perceptual uniformity.

```css
/* CSS OKLCH syntax */
color: oklch(65% 0.18 250);
/*          L    C    H   */
```

**Why it matters:** When you build a 10-step tonal scale in HSL, lighter and darker shades shift perceived hue. In OKLCH, hue stays consistent across lightness steps — giving you cleaner, more cohesive palettes with fewer manual tweaks.

---

## Perceptual Uniformity

Perceptual uniformity means: *a unit step in the color model equals roughly a unit step in human perception.*

Non-uniform models (RGB, HSL) produce palettes where some adjacent swatches look nearly identical while others look dramatically different, even with equal numerical spacing. Uniform models (OKLCH, LCH, CIELAB) make programmatic palette generation reliable.

**Practical implication:** Design tokens and tonal palettes (e.g., `blue-100` through `blue-900`) are far more predictable when generated in OKLCH. Tools like Leonardo.io and Radix Colors leverage this principle.

---

## Palette Generation Strategies

### The 60-30-10 Rule
A classic interior design principle that translates well to UI:
- **60%** — dominant neutral/background color
- **30%** — secondary color (surfaces, cards, sidebars)
- **10%** — accent color (CTAs, highlights, interactive states)

This ratio creates visual balance and prevents color fatigue. The accent color has impact precisely *because* it's used sparingly.

### Complementary, Analogous, and Triadic Schemes

| Scheme | Definition | Best For |
|---|---|---|
| **Complementary** | Opposite hues (e.g., blue + orange) | High contrast, CTA pop |
| **Analogous** | Adjacent hues (e.g., blue, cyan, teal) | Cohesive, calm UIs |
| **Triadic** | Three equidistant hues | Vibrant, playful interfaces |
| **Split-complementary** | One hue + two hues flanking its complement | Complementary contrast with less tension |

For most product UIs, **analogous** palettes feel polished and professional. Complementary pairings are effective for action-state contrast (a blue primary with an orange CTA). Triadic schemes are rare in enterprise/productivity tools but common in consumer/creative apps.

### Tonal Scales
Beyond the macro scheme, each color needs a **tonal scale** — a range of lightness steps from near-white to near-black. Common patterns are 9-step (`100`–`900`) or 11-step (`50`, `100`–`950`).

**Generate tonal scales by:**
1. Fixing the target hue and chroma in OKLCH
2. Sweeping lightness from ~95 (light) to ~15 (dark) in equal steps
3. Manually verify that steps feel visually equidistant

---

## Semantic Color Roles

A design system maps named palette swatches to **semantic roles** — intent-based aliases that components consume. This decouples visual values from meaning:

| Role | Purpose | Typical Hue |
|---|---|---|
| `primary` | Main brand actions, links | Blue, purple, teal |
| `secondary` | Supporting actions, badges | Brand-adjacent hue |
| `success` | Confirmations, valid states | Green |
| `error` / `danger` | Failures, destructive actions | Red |
| `warning` | Caution, non-blocking issues | Amber / orange |
| `info` | Neutral informational messages | Cyan / blue |
| `neutral` | Text, borders, backgrounds | Gray scale |

**Anti-pattern:** Hardcoding raw hex values in components. Always reference semantic tokens (`var(--color-error-600)` not `#dc2626`). This enables theming, brand swaps, and dark mode without touching component code.

---

## Contrast Ratios & WCAG Accessibility

The Web Content Accessibility Guidelines (WCAG) define minimum contrast ratios between foreground text and background:

| Level | Normal Text | Large Text (≥18pt / 14pt bold) |
|---|---|---|
| **AA** (minimum) | 4.5 : 1 | 3 : 1 |
| **AAA** (enhanced) | 7 : 1 | 4.5 : 1 |

**Large text** = ≥18pt regular or ≥14pt bold. Non-text UI elements (icons, input borders, focus rings) require **3:1** at AA.

### Practical Contrast Workflow
1. Use OKLCH lightness to predict contrast before computing it — two colors with `L` difference ≥ 40–50 points typically pass AA.
2. Verify with a tool (browser DevTools contrast checker, Figma A11y plugins, or `chroma.js`).
3. For semantic colors, ensure both the *500* swatch on white *and* the *700* swatch on dark backgrounds pass minimum ratios.
4. Never rely on color alone to convey state — pair with icons, labels, or patterns for color-blind users.

---

## Dark Mode Palette Design

Dark mode is not simply color inversion. Key principles:

### Elevation Through Lightness (Not Shadows)
In dark UIs, surfaces at higher elevation are *lighter*, not darker. A base background might be `neutral-900`, cards sit at `neutral-800`, modals at `neutral-750`. This reverses the light-mode shadow metaphor into a lightness-as-elevation system (used by Material Design).

### Desaturate for Dark Mode
Saturated colors that work on white backgrounds often appear neon or harsh on dark surfaces. Reduce chroma (~30–40%) and increase lightness when adapting brand colors to dark contexts.

### Avoid Pure Black
Pure `#000000` creates harsh contrast and flattens depth. Use near-blacks: `oklch(12% 0.01 250)` adds a subtle cool tint that feels softer and allows elevation differentiation.

### Semantic Token Mapping Pattern
```
--color-bg-base:      light → neutral-50   |  dark → neutral-950
--color-bg-surface:   light → white        |  dark → neutral-900
--color-bg-elevated:  light → white        |  dark → neutral-800
--color-text-primary: light → neutral-900  |  dark → neutral-50
--color-border:       light → neutral-200  |  dark → neutral-700
```

---

## Brand Color Adaptation

When integrating a brand color (e.g., a logo red) into a full design-system palette:

1. **Extract the OKLCH values** of the brand hex. This becomes your hue and chroma anchor.
2. **Generate a tonal scale** by sweeping lightness while preserving hue and chroma (clamping chroma if it exceeds the sRGB gamut at extreme lightness values).
3. **Designate the brand swatch** as the primary-500 or primary-600 anchor (the one that appears on white or dark backgrounds in CTAs).
4. **Verify contrast** of the chosen CTA swatch against intended backgrounds — brand colors often fail AA at their "pure" value and need a darker variant for text on light backgrounds.
5. **Derive neutrals from the brand hue** — shift chroma to near-zero while keeping the hue. This gives warm or cool grays that feel brand-aligned without competing with the accent.

---

## Color Blindness Considerations

~8% of men and 0.5% of women have some form of color vision deficiency. The most common types:

| Type | What's Affected | Common Confusion |
|---|---|---|
| Deuteranopia/anomaly | Green receptors | Red ↔ Green |
| Protanopia/anomaly | Red receptors | Red ↔ Green (reds appear dark) |
| Tritanopia | Blue receptors | Blue ↔ Yellow |

**Design strategies:**
- **Never rely on red/green alone** to distinguish success from error states — pair with icons (✓ / ✗) and labels.
- **Test palettes** with simulation tools (Figma's Vision Simulator plugin, Colour Blindness Simulator).
- Use **shape + color** redundantly for charts and data visualizations.
- Prefer **blue + orange** for complementary contrast pairs over red + green.
- Ensure sufficient lightness *contrast* between colors even after hue meaning is stripped away.

---

## CSS Custom Properties for Theming

Design tokens should map to CSS custom properties scoped to `:root` (or a theme class), enabling component-level consumption and easy theme switching:

```css
/* Primitive tokens — raw palette values */
:root {
  --blue-500: oklch(55% 0.20 250);
  --blue-600: oklch(47% 0.20 250);
  --green-500: oklch(62% 0.18 145);
  --red-500: oklch(55% 0.22 25);
  --neutral-50: oklch(97% 0.005 250);
  --neutral-900: oklch(18% 0.01 250);
}

/* Semantic tokens — role-based aliases */
:root {
  --color-primary: var(--blue-600);
  --color-primary-hover: var(--blue-700);
  --color-success: var(--green-500);
  --color-error: var(--red-500);
  --color-bg-base: var(--neutral-50);
  --color-text-primary: var(--neutral-900);
}

/* Dark mode override */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-base: var(--neutral-950);
    --color-text-primary: var(--neutral-50);
    --color-primary: var(--blue-400); /* lighter for dark bg contrast */
  }
}
```

**Two-layer token architecture** (primitives → semantics) is the industry standard pattern — used by Tailwind, Radix, Material, and Atlassian Design System. Components only consume semantic tokens; only the semantic layer changes between themes.

---

## Tooling

| Tool | Purpose | Strengths |
|---|---|---|
| **[Coolors](https://coolors.co)** | Palette exploration & generation | Fast generation, contrast checker, export formats |
| **[Realtime Colors](https://realtimecolors.com)** | Live preview on UI templates | See palette in context immediately, OKLCH support |
| **[Leonardo (adobe)](https://leonardocolor.io)** | Perceptually-uniform scale generation | APCA contrast targeting, OKLCH-based scales, design token export |
| **[Radix Colors](https://www.radix-ui.com/colors)** | Pre-built accessible scales | 12-step OKLCH-informed scales, dark mode variants included |
| **[oklch.com](https://oklch.com)** | OKLCH color picker | Visual OKLCH exploration, gamut visualization |
| **[Colour Contrast Checker](https://colourcontrast.cc)** | WCAG ratio verification | Quick AA/AAA pass/fail for any color pair |

---

## Summary & Key Trade-offs

| Decision | Trade-off |
|---|---|
| HSL vs. OKLCH for scale generation | HSL is simpler/universal but perceptually inconsistent; OKLCH requires modern browser support but produces superior tonal scales |
| 60-30-10 vs. custom ratios | Rule is a reliable starting point but product context matters — data-dense UIs may skew more neutral |
| AA vs. AAA compliance | AAA is aspirational and can constrain brand color usage; target AA at minimum, AAA for body text |
| Per-component vs. global tokens | Global semantic tokens scale better; per-component tokens give more granular control but increase maintenance overhead |
| sRGB vs. wide gamut (P3) | P3 enables more vivid OKLCH colors but requires gamut-clamping fallbacks for sRGB displays |

A robust color system is a **living design artifact** — it requires audit cycles as products grow, brand guidelines evolve, and accessibility standards update (see APCA, the successor contrast algorithm to WCAG's relative luminance formula, currently in development for WCAG 3.0).
