<h1 align="center">ckelley.dev</h1>

<p align="center">
  <strong>Personal portfolio and technical showcase for Christopher Kelley</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Astro-BC52EE?style=for-the-badge&logo=astro&logoColor=white" alt="Astro">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind CSS">
  <img src="https://img.shields.io/badge/pnpm-F69220?style=for-the-badge&logo=pnpm&logoColor=white" alt="pnpm">
  <img src="https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="NGINX">
</p>

<p align="center">
  <a href="https://ckelley.dev">ckelley.dev</a>
</p>

---

## Overview

A fast, static portfolio site that doubles as a technical resume. It covers professional
experience, open source projects, the homelab and production infrastructure, the local and
cloud AI stack, and academic background. Everything is server-rendered to static HTML at build
time with no client-side framework, so pages load instantly and the whole site deploys as plain
files behind NGINX.

## Pages

| Route | Description |
|-------|-------------|
| `/` | Landing page with hero, experience, and an infrastructure teaser |
| `/technologies` | Technology stack grouped by category, including monitoring and security |
| `/projects` | Open source work across Rust, Zig, MSP tooling, and Linux/DevOps |
| `/lab` | Homelab and production infrastructure, observability, and security write-ups |
| `/ai` | Local and cloud AI hardware, the inference stack, and harness engineering |
| `/education` | Degree, certifications, capstone, and academic projects |

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Framework** | Astro 6 (static output) |
| **Styling** | Tailwind CSS 4 via the `@tailwindcss/vite` plugin |
| **Language** | TypeScript |
| **Package Manager** | pnpm |
| **Hosting** | NGINX serving static files behind HTTPS |

## Quick Start

### Prerequisites
- Node.js 18+
- pnpm

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

```
ckelley.dev/
├── public/                 # Static assets served as-is (logos, images, favicons)
├── src/
│   ├── components/         # Reusable Astro components (Hero, Navigation, etc.)
│   ├── layouts/            # Base page layout
│   ├── pages/              # One file per route (index, lab, ai, projects, ...)
│   └── styles/             # Global Tailwind styles
├── astro.config.mjs        # Astro + Tailwind (Vite plugin) configuration
└── package.json
```

## Deployment

The site builds to a static `dist/` directory and is served by NGINX out of
`/var/www/ckelley.dev`. Deploying is a matter of building and syncing the output:

```bash
pnpm build
# copy the contents of dist/ to the web root on the server
```

## License

Proprietary - All rights reserved.

---

<p align="center">
  Built and maintained by <a href="https://github.com/GhostKellz">Christopher Kelley</a>
</p>
