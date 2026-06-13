# ckelley.dev — Documentation

Project documentation for [ckelley.dev](https://ckelley.dev) — the static Astro portfolio and
technical resume for Christopher Kelley (CK Technology LLC).

This directory explains how the project is built, how to work on it, and how it ships to
production behind Cloudflare. For a high-level summary and quick start, see the
[root README](../README.md).

## Contents

| Document | Description |
|----------|-------------|
| [architecture.md](architecture.md) | Tech stack rationale, key decisions, and the inline-SVG diagram system |
| [development.md](development.md) | Local setup, project conventions, adding pages/components/diagrams |
| [deployment.md](deployment.md) | Build, transfer pipeline, NGINX config, Cloudflare DNS-01 TLS |
| [cloudflare.md](cloudflare.md) | Cloudflare Pro edge: proxied DNS, WAF, caching, TLS Full (Strict) |
| [security.md](security.md) | Trust planes, network posture, CrowdSec, observability, zero-trust mgmt |
| [design-system.md](design-system.md) | Navy/teal tokens, typography, `@utility` classes, flow primitives |
| [content.md](content.md) | Page map, components, diagram inventory, contact info, external services |

## At a Glance

| | |
|---|---|
| **Owner** | Christopher Kelley / CK Technology LLC |
| **Type** | Personal portfolio + technical resume |
| **Framework** | Astro 6.4.6 (static output) |
| **Styling** | Tailwind CSS v4 (CSS-first `@theme`, via `@tailwindcss/vite`) |
| **Language** | TypeScript (`astro/tsconfigs/strict`) |
| **Build tool** | Vite 7 |
| **Interactivity** | Alpine.js (CDN) — mobile menu, scroll-spy nav, hero typing/terminal |
| **Diagrams** | 22 hand-authored inline-SVG flow components (CSS animation, reduced-motion aware) |
| **Package manager** | pnpm |
| **Hosting** | NGINX VM on the PVE1 bare-metal cloud node (Proxmox SDN) |
| **Edge** | Cloudflare (Pro plan), proxied (orange-cloud) records |
| **Deploy** | tar + rclone over SSH via Tailscale (no public SSH) |
| **TLS** | acme.sh + Let's Encrypt, DNS-01 via a scoped Cloudflare API token |
| **Domains** | ckelley.dev, www.ckelley.dev |
| **License** | Proprietary (Christopher Kelley / CK Technology LLC, 2026) |

## Quick Links

- **Live site:** https://ckelley.dev
- **Source:** `src/` (pages, components, layouts, styles)
- **Diagrams:** `src/components/diagram/` (frame/lane/node primitives + `flows/`)
- **Tokens & primitives:** `src/styles/global.css`
