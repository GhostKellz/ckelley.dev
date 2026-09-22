<h1 align="center">ckelley.dev</h1>

<p align="center">
  <strong>Personal portfolio, technical resume, and infrastructure showcase for Christopher Kelley</strong>
</p>

<p align="center">
  <a href="https://ckelley.dev"><img src="https://img.shields.io/badge/Live-ckelley.dev-64ffda?style=for-the-badge" alt="Live Site"></a>
  <img src="https://img.shields.io/badge/Astro-7-BC52EE?style=for-the-badge&logo=astro&logoColor=white" alt="Astro">
  <img src="https://img.shields.io/badge/TypeScript-6-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind CSS">
  <img src="https://img.shields.io/badge/pnpm-10-F69220?style=for-the-badge&logo=pnpm&logoColor=white" alt="pnpm">
  <img src="https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="NGINX">
  <img src="https://img.shields.io/badge/Static-Site-12121a?style=for-the-badge" alt="Static Site">
</p>

---

## Overview

`ckelley.dev` is a fast, static portfolio site that doubles as a living technical resume. It
covers professional experience, open source work, the homelab and production infrastructure, the
local and cloud AI stack, and academic background.

Everything is server-rendered to static HTML at build time with **no client-side framework** — pages
load instantly and the entire site deploys as plain files behind NGINX. The standout piece is the
`/architecture` page, which renders the infrastructure as hand-authored, animated inline-SVG flow
diagrams (no Mermaid, no D3, zero JS libraries) themed to the site's navy/teal system.

## Features

### Content & Pages
- Landing page with hero, experience timeline, and an infrastructure teaser
- Technology stack grouped by category, including monitoring and security
- Open source projects across Rust, Zig, MSP tooling, and Linux/DevOps
- Homelab and production infrastructure deep-dive with observability and security write-ups
- Local and cloud AI hardware, the inference stack, and harness engineering
- Education: degree, certifications, capstone, and academic projects

### Architecture Diagrams
- 20+ hand-authored inline-SVG flow components (`src/components/diagram/flows/`)
- True branching/merging graphs with labeled and dotted (observe-only) edges
- CSS-only animated "flowing dash" along edges; terminal nodes glow
- Per-domain accent theming via `--node` / `--edge` CSS custom properties
- Responsive `viewBox` scaling with horizontal scroll for wide diagrams on mobile
- Accessible: `role="img"` with `<title>`/`<desc>`, decorative markers `aria-hidden`
- Honors `prefers-reduced-motion`

### Build & Delivery
- Static HTML/CSS output — no runtime, no hydration
- Section navigation with scroll-spy highlighting
- `IntersectionObserver` reveal-on-scroll transitions
- Branded Open Graph metadata and favicons

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Framework** | Astro 7 (static output) |
| **Styling** | Tailwind CSS 4 via the `@tailwindcss/vite` plugin |
| **Language** | TypeScript |
| **Diagrams** | Hand-authored inline SVG (no charting libraries) |
| **Package Manager** | pnpm |
| **Build Tool** | Vite 8 |
| **Output** | Static HTML/CSS |
| **Hosting** | NGINX serving static files behind HTTPS |

## Quick Start

### Prerequisites
- Node.js 22.12+ (developed on Node 24)
- pnpm 10+

### Development

```bash
# Install dependencies
pnpm install

# Start the dev server (http://localhost:4321)
pnpm dev
```

### Production Build

```bash
# Build static output into dist/
pnpm build

# Preview the production build locally
pnpm preview
```

## Project Structure

```text
ckelley.dev/
├── public/                     # Static assets served as-is (logos, images, favicons)
├── src/
│   ├── components/
│   │   ├── diagram/
│   │   │   ├── flows/          # 20+ inline-SVG flow diagrams
│   │   │   ├── DiagramFrame.astro   # Titled/toned frame + legend slot
│   │   │   ├── DiagramLane.astro    # Lane-based stacked primitive
│   │   │   └── DiagramNode.astro    # Reusable node primitive
│   │   ├── Hero.astro
│   │   ├── Navigation.astro
│   │   ├── SectionNav.astro    # Scroll-spy section nav
│   │   └── ...                 # About, Experience, Skills, Infrastructure, Contact
│   ├── layouts/
│   │   └── BaseLayout.astro    # Base document shell
│   ├── pages/                  # One file per route
│   │   ├── index.astro
│   │   ├── architecture.astro  # Animated SVG infrastructure showcase
│   │   ├── ai.astro
│   │   ├── lab.astro
│   │   ├── projects.astro
│   │   ├── technologies.astro
│   │   └── education.astro
│   └── styles/
│       └── global.css          # Tailwind 4 theme tokens + flow-diagram classes
├── astro.config.mjs            # Astro + Tailwind (Vite plugin) configuration
└── package.json
```

## Deployment

The site builds to a static `dist/` directory and is served by NGINX out of
`/var/www/ckelley.dev`. Deploying is a matter of building and syncing the output:

```bash
pnpm build
# sync the contents of dist/ to the web root on the server
rsync -avz --delete dist/ user@host:/var/www/ckelley.dev/
```

## Notes

- Tailwind CSS 4 theme tokens and the SVG flow-diagram primitives live in
  [`src/styles/global.css`](src/styles/global.css), including the `flow-dash` animation keyframe.
- Flow diagrams are coordinate-authored to match real topology; each carries its own scoped
  `<defs>` (uniquely-suffixed arrowhead marker) so multiple diagrams coexist on one page.
- No JavaScript framework or charting dependency is used — diagrams are static SVG with CSS
  animation only.

## License

Copyright © 2026 Christopher Kelley / CK Technology LLC. Proprietary — all rights reserved.
See [`LICENSE`](LICENSE) for details.

---

<p align="center">
  Built and maintained by <a href="https://github.com/GhostKellz">Christopher Kelley</a>
</p>
