# Design System

The visual language of the site: colors, typography, the reusable `@utility` classes, and the
inline-SVG flow primitives.

Everything is defined CSS-first in `src/styles/global.css`. Tailwind v4's `@theme` block holds
the design tokens (colors, fonts, animations), and `@utility` definitions hold the reusable
component classes. There is no JavaScript Tailwind config.

## Brand Colors

Defined as `--color-*` tokens in the `@theme` block, consumed through Tailwind utilities
(`text-accent`, `bg-navy-light`, `border-slate-dark`, etc.).

### Base palette

| Token | Hex | Usage |
|-------|-----|-------|
| `navy` | `#0a192f` | Primary background |
| `navy-light` | `#112240` | Cards, raised surfaces, alternating sections |
| `navy-dark` | `#020c1b` | Deepest background, scrollbar track |
| `accent` | `#64ffda` | Primary accent — teal: links, headings accents, buttons, glows |
| `accent-muted` | `#4dc9b0` | Accent hover / secondary button hover |
| `accent-dark` | `#48d1a0` | Darker accent variant |
| `aqua` / `aqua-light` / `aqua-dark` | `#0891b2` / `#22d3ee` / `#0e7490` | Secondary accent (gradients, aqua tone) |
| `slate-light` | `#ccd6f6` | Primary body / heading text |
| `slate` | `#8892b0` | Secondary text |
| `slate-dark` | `#495670` | Muted text, borders, dividers |

### Per-domain diagram accents

Layered on the navy base, these drive the tone of `/architecture` diagrams:

| Token | Hex | Token | Hex |
|-------|-----|-------|-----|
| `violet` / `violet-light` | `#a855f7` / `#c084fc` | `danger` / `danger-light` | `#f87171` / `#fca5a5` |
| `amber` / `amber-light` | `#fbbf24` / `#fcd34d` | `blue` / `blue-light` | `#3b82f6` / `#60a5fa` |
| `emerald` / `emerald-light` | `#34d399` / `#6ee7b7` | | |

Use the tokens through Tailwind utilities rather than hard-coding hex values so the palette stays
consistent.

## Typography

- **Fonts** (loaded from Google Fonts in `BaseLayout.astro`, with `system-ui, sans-serif`
  fallbacks):
  - `--font-sans`: [Inter](https://fonts.google.com/specimen/Inter) (weights 400/500/600/700) —
    body and headings.
  - `--font-mono`: [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) (weights
    400/500), with Fira Code fallback — eyebrow labels, terminal, diagram text, code-style accents.
- **Body:** `font-sans antialiased`, `bg-navy text-slate-light`.
- `<html>` uses `scroll-behavior: smooth`.
- A teal text selection (`::selection`) and custom scrollbar styling are defined in the base
  layer.

## Animations

Keyframes are declared in the `@theme` block and exposed as `--animate-*` tokens:

| Token | Effect |
|-------|--------|
| `--animate-float` | Gentle vertical float (6s loop) |
| `--animate-pulse-glow` | Pulsing teal box-shadow glow |
| `--animate-fade-in-up` | Fade + rise on load (used by the hero) |
| `--animate-dataflow` | Glowing dot travelling along a diagram connector |
| `--animate-flow-dash` | The "flowing dash" along SVG flow edges |

The `.reveal` / `.reveal.visible` classes (driven by an `IntersectionObserver` in
`BaseLayout.astro`) animate sections in on scroll. A blinking `.cursor` powers the hero typing
effect. All motion respects `@media (prefers-reduced-motion: reduce)`.

## Component Classes (`@utility`)

Defined as `@utility` rules in `src/styles/global.css`. Prefer these over re-typing long utility
strings.

### Layout & type

| Class | Description |
|-------|-------------|
| `section` | Vertical + horizontal section padding (`py-20 md:py-28 px-6`) |
| `container-narrow` | Centered max-width container (`max-w-5xl mx-auto`) |
| `heading` | Section heading (`text-3xl md:text-4xl font-bold text-slate-light`) |
| `heading-accent` | Mono eyebrow label above a heading (`text-accent font-mono text-sm`) |

### Buttons

| Class | Description |
|-------|-------------|
| `btn` | Base: inline-flex, centered, `px-6 py-3`, rounded, medium weight, animated |
| `btn-primary` | Transparent with a 2px teal border + teal text → faint teal fill on hover |
| `btn-secondary` | Solid teal background, navy-dark text → muted teal on hover |

### Surfaces & links

| Class | Description |
|-------|-------------|
| `card` | `navy-light` surface, rounded, padded, subtle border; on hover the border tints teal, a soft teal shadow appears, and it lifts (`-translate-y-1`) |
| `link` | Teal text with an underline-on-hover |
| `social-link` | Slate icon → teal on hover |
| `text-gradient` | Teal→aqua clipped text gradient |
| `glow` / `glow-sm` | Teal box-shadow glow (large / small) |

## SVG Flow Primitives

The `/architecture` diagrams are built from a small set of CSS classes (also in `global.css`)
shared by every hand-authored SVG. Tone is driven by the `--node` and `--edge` CSS custom
properties set on a wrapper or per element.

| Class | Role |
|-------|------|
| `flow-svg` | The responsive `<svg>` itself (`width:100%; height:auto`) |
| `flow-node` / `flow-node--glow` | A node rectangle; the `--glow` variant adds a colored drop-shadow for terminal nodes |
| `flow-title` / `flow-sub` | Node title and sub-label text (mono) |
| `flow-edge` / `flow-edge--dot` | An edge path with the animated "flowing dash"; the `--dot` variant is a sparser dotted (observe-only) edge |
| `flow-edge-chip` / `flow-edge-label` | A small chip + label drawn on an edge |
| `flow-group` / `flow-group-label` | A dashed "subgraph" box and its uppercase label |
| `.diagram-conn` | The vertical connector with a travelling glowing dot used between `DiagramLane`s |

These primitives are composed by three reusable Astro components — `DiagramFrame`,
`DiagramLane`, and `DiagramNode` — plus the standalone `flows/*.astro` SVGs. See
[content.md](content.md#diagram-components) for the inventory and
[architecture.md](architecture.md#the-diagram-system) for the rationale.

## Imagery

Brand and content imagery lives in `public/`:

- `profile.jpg` — headshot (About section).
- `infrastructure.png` — infrastructure visual.
- `og-image.png` — Open Graph social card (referenced by `BaseLayout`).
- Favicons: `favicon.png`, `favicon-16.png`, `favicon-32.png`, `apple-touch-icon.png`.
- `logos/` — technology logos used by the Skills section and Technologies page.

See [content.md](content.md) for the full asset inventory.
