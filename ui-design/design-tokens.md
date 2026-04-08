---
title: "Design Tokens"
category: ui-design
summary: "A deep-dive into design tokens — what they are, the W3C DTCG spec, token types and tiers, CTI naming conventions, JSON/YAML formats, tooling (Style Dictionary, Tokens Studio, Figma Variables), multi-platform output, theming, governance, and migration strategies."
sources:
  - web-research
updated: 2026-04-08T18:27:00.000Z
---

# Design Tokens

> A deep-dive into design tokens — what they are, the W3C DTCG spec, token types and tiers, CTI naming conventions, JSON/YAML formats, tooling (Style Dictionary, Tokens Studio, Figma Variables), multi-platform output, theming, governance, and migration strategies.

## What Are Design Tokens?

**Design tokens** are the smallest, named design decisions in a system — the atomic unit of a visual language. Rather than hard-coding `#3B82F6` into a component's stylesheet, you define a token `color.interactive.primary` that resolves to that value. The token *name* carries intent; the *value* is interchangeable.

Jina Anne and Jon Levine coined the term at Salesforce around 2014. The concept acknowledges a fundamental problem: design decisions live in multiple places simultaneously — Figma mocks, iOS Swift files, Android XML, web CSS — and drift apart the moment anyone changes one without updating the others. Tokens provide a **single source of truth** that platforms consume, not copy.

The value of tokens compounds across scale:
- A brand color change becomes a one-token edit, not a grep-and-replace across 40 files.
- Dark mode is a semantic token remap, not a new stylesheet.
- A new platform (watch OS, TV) inherits decisions automatically.

---

## W3C Design Token Community Group (DTCG) Spec

The [W3C Design Token Community Group](https://www.w3.org/community/design-tokens/) (est. 2019) is standardizing a cross-tool, cross-platform token format. The **DTCG spec** (Draft 2024) defines:

- A canonical **JSON schema** for token files (`.tokens.json`)
- A `$type` field for token typing (`color`, `dimension`, `fontFamily`, `duration`, etc.)
- A `$value` field for the resolved value
- A `$description` field for documentation
- Support for **token groups** (nested objects) and **references** (`{token.path}` syntax)

```json
{
  "color": {
    "blue": {
      "500": {
        "$type": "color",
        "$value": "#3B82F6",
        "$description": "Core blue, used for interactive elements"
      }
    }
  },
  "interactive": {
    "primary": {
      "$type": "color",
      "$value": "{color.blue.500}"
    }
  }
}
```

The DTCG spec is not yet a W3C Recommendation, but **Style Dictionary v4** and **Tokens Studio** already support it as the primary format. Adopting the spec now future-proofs your token files.

---

## Token Types

| Type | `$type` | Example Value | Platforms |
|------|---------|---------------|-----------|
| **Color** | `color` | `#3B82F6`, `rgba(59,130,246,0.5)` | CSS, iOS `UIColor`, Android `Color` |
| **Spacing / Dimension** | `dimension` | `16px`, `1rem` | CSS spacing/padding, SwiftUI padding |
| **Typography** | `typography` | Composite: `fontFamily`, `fontSize`, `fontWeight`, `lineHeight` | CSS `font-*`, iOS `UIFont`, Android `TextAppearance` |
| **Shadow** | `shadow` | Composite: `offsetX`, `offsetY`, `blur`, `spread`, `color` | CSS `box-shadow`, iOS `CALayer.shadowOffset` |
| **Border** | `border` | Composite: `color`, `width`, `style` | CSS `border`, Android `ShapeDrawable` |
| **Motion / Duration** | `duration` | `200ms` | CSS `transition-duration`, iOS `UIView.animateDuration` |
| **Motion / Easing** | `cubicBezier` | `[0.4, 0, 0.2, 1]` | CSS `cubic-bezier(...)`, iOS `CAMediaTimingFunction` |
| **Font Family** | `fontFamily` | `["Inter", "sans-serif"]` | CSS `font-family` |
| **Font Weight** | `fontWeight` | `700` | CSS `font-weight` |

**Composite tokens** (typography, shadow, border) bundle multiple values under one token. Tools expand them into platform-specific equivalents at build time.

---

## Token Tiers: Global → Alias → Component

A three-tier hierarchy prevents fragility and enables theming:

```
Global (primitive)  →  Alias (semantic)  →  Component
──────────────────     ─────────────────    ──────────────
color.blue.500         color.interactive    button.background
color.gray.100         color.surface        button.text
space.4 (16px)         spacing.md           button.padding
```

| Tier | Also Called | Purpose | Change Frequency |
|------|-------------|---------|-----------------|
| **Global** | Primitive, Scale | Raw palette — all possible values | Rarely; brand rebrand |
| **Alias** | Semantic, Reference | Named by *intent*, not value | On theme switch |
| **Component** | Specific | Scoped to one component | On component redesign |

**Rule**: Components only reference alias tokens. Alias tokens reference global tokens. Global tokens hold literal values. This means a dark-mode theme only needs to remap alias → global; every component updates automatically.

```css
:root {
  /* Global */
  --color-blue-500: #3B82F6;
  --color-slate-900: #0F172A;

  /* Alias */
  --color-interactive: var(--color-blue-500);
  --color-text-primary: var(--color-slate-900);

  /* Component */
  --button-bg: var(--color-interactive);
  --button-label: var(--color-text-primary);
}

[data-theme="dark"] {
  /* Only alias layer changes for dark mode */
  --color-interactive: #60A5FA;     /* blue-400, lighter for dark bg */
  --color-text-primary: #F8FAFC;    /* slate-50 */
}
```

---

## Naming Conventions: CTI Format

The **Category / Type / Item (CTI)** convention — popularized by Style Dictionary — provides a consistent, parseable namespace:

```
{category}.{type}.{item}.{sub-item}.{state}
  color    .text  .primary.default  .hover
  space    .gap   .md
  motion   .ease  .in-out
```

| Segment | Purpose | Examples |
|---------|---------|---------|
| `category` | Broad domain | `color`, `space`, `typography`, `motion`, `border` |
| `type` | Semantic role | `text`, `background`, `interactive`, `feedback` |
| `item` | Specific token | `primary`, `secondary`, `danger`, `success` |
| `sub-item` | Variant/scale | `default`, `subtle`, `strong`, `sm`, `md`, `lg` |
| `state` | Interaction state | `hover`, `focus`, `disabled`, `pressed` |

**Platform prefix conventions** (CSS custom properties drop the prefix; mobile uses dot notation):
```
CSS:      --color-text-primary-hover
iOS:      ColorTextPrimaryHover           (PascalCase)
Android:  color_text_primary_hover        (snake_case)
JS/TS:    color.text.primary.hover        (dot path)
```

Avoid abbreviations (`bg` vs. `background`) unless the team agrees on a glossary. Consistency beats brevity.

---

## Token Formats: JSON and YAML

### DTCG JSON (Preferred)

```json
{
  "spacing": {
    "$type": "dimension",
    "sm":  { "$value": "8px" },
    "md":  { "$value": "16px" },
    "lg":  { "$value": "24px" },
    "xl":  { "$value": "32px" }
  },
  "shadow": {
    "card": {
      "$type": "shadow",
      "$value": {
        "offsetX": "0",
        "offsetY": "4px",
        "blur": "12px",
        "spread": "0",
        "color": "{color.black.alpha.20}"
      }
    }
  }
}
```

### YAML (Human-Friendly Alternative)

```yaml
spacing:
  $type: dimension
  sm:
    $value: "8px"
  md:
    $value: "16px"

motion:
  duration:
    fast:
      $type: duration
      $value: "100ms"
    base:
      $type: duration
      $value: "200ms"
  easing:
    standard:
      $type: cubicBezier
      $value: [0.4, 0, 0.2, 1]
```

**Trade-offs**:

| Format | Pros | Cons |
|--------|------|------|
| DTCG JSON | Spec-compliant, tool-native, diff-friendly | Verbose for large files |
| YAML | More readable, fewer quotes, good for comments | Requires transpile step; indentation errors are silent |
| Legacy SD JSON | Wide tool support | Pre-spec, no `$type`; migrating adds friction |

---

## Tools

### Style Dictionary (Amazon)

[Style Dictionary](https://amzn.github.io/style-dictionary/) is the de-facto **token build system**. It reads token files and generates platform-specific output via configurable transforms and formats.

```js
// style-dictionary.config.js
export default {
  source: ['tokens/**/*.tokens.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      prefix: 'ds',
      buildPath: 'dist/css/',
      files: [{ destination: 'tokens.css', format: 'css/variables' }],
    },
    ios: {
      transformGroup: 'ios-swift',
      buildPath: 'dist/ios/',
      files: [{ destination: 'StyleDictionaryColor.swift', format: 'ios-swift/class.swift' }],
    },
    android: {
      transformGroup: 'android',
      buildPath: 'dist/android/res/values/',
      files: [{ destination: 'colors.xml', format: 'android/colors' }],
    },
  },
};
```

Style Dictionary v4 natively supports the DTCG `$type`/`$value` schema and ships with custom transform hooks for Tailwind, Panda CSS, and more.

### Tokens Studio (Figma Plugin)

[Tokens Studio](https://tokens.studio/) bridges the gap between Figma and code:
- Designers define tokens directly in Figma via the plugin UI
- Tokens sync to a **GitHub/GitLab repo** or S3 in DTCG JSON format
- Engineers pull the JSON and run Style Dictionary to generate CSS/Swift/XML
- Token changes in Figma become pull requests — reviewable, versioned, auditable

The bidirectional sync is the killer feature: a token rename in Figma propagates to all platforms via a normal CI pipeline.

### Figma Variables (Native, 2023+)

Figma's first-party **Variables** feature supports color, number, string, and boolean tokens with **modes** (e.g., Light / Dark / High Contrast). Variables can be exported via the REST API and are natively supported by Tokens Studio's sync layer.

**Limitations**: Figma Variables don't yet support composite types (typography, shadow, motion) — Tokens Studio fills that gap. Variables also lack the CTI naming enforcement that a design system team needs.

---

## Multi-Platform Output

| Platform | Output Format | Example |
|----------|--------------|---------|
| **Web** | CSS custom properties | `--color-interactive: #3B82F6;` |
| **iOS (Swift)** | `UIColor` extension / SwiftUI `Color` | `Color("colorInteractive")` |
| **Android** | XML resources | `<color name="color_interactive">#3B82F6</color>` |
| **Tailwind CSS** | `theme.extend` JS config | `colors: { interactive: 'var(--color-interactive)' }` |
| **JS/TS** | ESM constants | `export const colorInteractive = '#3B82F6'` |

**Tailwind integration** — reference CSS custom properties so tokens are single-source even inside Tailwind utilities:

```js
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        interactive: 'var(--color-interactive)',
        surface: 'var(--color-surface)',
        'text-primary': 'var(--color-text-primary)',
      },
      spacing: {
        sm: 'var(--spacing-sm)',
        md: 'var(--spacing-md)',
      },
    },
  },
};
```

Now `bg-interactive` in a Tailwind class uses the same token value as `var(--color-interactive)` in a CSS-in-JS component.

---

## Theming with Tokens

Themes are **alias-layer remaps**. Global tokens stay constant; semantic tokens are overridden per theme context:

```css
/* tokens.css — generated by Style Dictionary */
:root {
  --color-brand-blue:   #3B82F6;
  --color-brand-indigo: #6366F1;
  --color-surface-base: #FFFFFF;
  --color-interactive:  var(--color-brand-blue);   /* default theme */
}

/* Alternate brand/tenant theme */
[data-theme="indigo"] {
  --color-interactive: var(--color-brand-indigo);
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-surface-base: #0F172A;
    --color-interactive:  #60A5FA;
  }
}
```

**Multi-tenant theming**: Supply each tenant a small token override file. Style Dictionary merges it with the base token set to generate a tenant-specific CSS file. The component code never changes.

---

## Token Governance

Token governance prevents the "token sprawl" that defeats the purpose of a token system:

| Practice | Why It Matters |
|----------|---------------|
| **Token review process** | New tokens require a PR review; prevents duplicate/redundant tokens |
| **Token linting** | Tools like [Token Lint](https://github.com/drwpow/cobalt-ui) enforce naming conventions and catch broken references at CI time |
| **Deprecation runway** | Mark tokens `$deprecated: true` in DTCG metadata; flag in generated code; remove after one major version |
| **Changelog** | Every token add/rename/remove is a versioned entry — teams know what to update |
| **Token ownership** | Assign an owner (design system team) for global/alias tokens; component teams own component tokens |
| **No direct global references** | ESLint / Stylelint rules that warn when components bypass the alias layer |

**Token budget**: Audit quarterly. A healthy system has ~30–80 alias tokens. >200 alias tokens usually indicates semantic creep — tokens named for specific components rather than intent.

---

## Migration Strategies

### From Hard-Coded Values to Tokens (Brownfield)

1. **Audit** — Extract all unique color, spacing, and typography values via a CSS audit tool (e.g., `css-analyzer`). Identify clusters (near-duplicates that should be the same token).
2. **Define global tokens first** — Map audit values to a primitive scale. `#3B82F6` and `#3B81F5` both become `color.blue.500`.
3. **Layer alias tokens** — Assign semantic intent to each global token. `color.blue.500` → `color.interactive`.
4. **Strangler fig** — Replace one file/component at a time. Use a linter rule to flag raw hex values in new code.
5. **Never big-bang** — Trying to migrate all 400 CSS files at once guarantees a rollback.

### From Legacy Token Format to DTCG

```json
/* Before: Style Dictionary legacy format */
{
  "color": {
    "primary": { "value": "#3B82F6", "type": "color" }
  }
}

/* After: DTCG format */
{
  "color": {
    "primary": { "$value": "#3B82F6", "$type": "color" }
  }
}
```

The only change is `value` → `$value` and `type` → `$type`. A one-pass `jq` or Node script handles the migration. Style Dictionary v4's `usesDtcg: true` flag automates the switch.

### From Tokens Studio to Native Figma Variables

Export your Tokens Studio JSON → run through a DTCG-to-Variables converter (several open-source options exist) → import via Figma REST API. Preserve the three-tier structure; Variables' **mode** feature maps 1:1 to alias token themes.

---

## Trade-Off Summary

| Dimension | No Tokens | Tokens (Flat) | Tokens (Tiered) |
|-----------|-----------|---------------|-----------------|
| Consistency | Low | Medium | High |
| Theming cost | Very high | Medium | Low |
| Setup effort | None | Low | Medium |
| Cross-platform sync | Manual | Semi-automated | Fully automated |
| Governance overhead | None | Low | Medium |
| Onboarding clarity | Low | Medium | High |

**The bottom line**: Design tokens pay for themselves the first time you ship a dark mode, support a second platform, or onboard a new designer who needs to understand your visual language. The upfront cost is a token audit and tooling setup — roughly a week of engineering and design time. The long-term return is measured in avoided regressions, faster theming, and cross-platform consistency that survives team turnover.
