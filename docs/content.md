# Content & Integrations

A map of the site's pages, components, diagrams, static assets, contact details, and external
services.

## Pages

File-based routing — each file in `src/pages/` maps to a route.

| File | Route | Purpose |
|------|-------|---------|
| `index.astro` | `/` | Home: hero, about, skills, infrastructure teaser, experience timeline, contact |
| `architecture.astro` | `/architecture` | Whole-stack deep-dive rendered as animated inline-SVG flow diagrams |
| `ai.astro` | `/ai` | Local & cloud AI hardware, inference stack, and agent harness |
| `lab.astro` | `/lab` | Homelab / production infrastructure deep-dive (observability, security, backup) |
| `projects.astro` | `/projects` | Open-source projects (Zig, Rust, MSP/Linux/DevOps tooling) |
| `technologies.astro` | `/technologies` | MSP/IT technology stack: virtualization, M365, networking, security, DevOps |
| `education.astro` | `/education` | Degree, certifications, capstone, and academic projects |

The home page composes section components and uses `SectionNav` for scroll-spy; the
`architecture` page passes its own `navSections` list to `SectionNav`. The `ai`, `lab`,
`projects`, `technologies`, and `education` pages render `Navigation` and author their content
inline (with page-local data arrays).

## Section Components

Reusable `.astro` components in `src/components/` (primarily composed on the home page).

| Component | Purpose |
|-----------|---------|
| `Hero.astro` | Hero: greeting, name, Alpine-driven rotating role "typing" animation, tagline, social links, CTA, an Alpine-driven looping terminal widget, and a vanilla-JS particle-network canvas |
| `About.astro` | Bio, recent-tech list, and headshot with a decorative hover border |
| `Skills.astro` | Technology categories (Systems & Infrastructure, Languages, Cloud & Enterprise, Security & Networking, OSes, Linux distros) rendered as logo cards |
| `Infrastructure.astro` | Homelab stats grid (with a subtle scroll parallax) and a CTA into `/lab` |
| `Experience.astro` | Vertical timeline of work, education, and certifications |
| `Contact.astro` | Social cards, an email CTA, and the site footer |
| `Navigation.astro` | Fixed top nav; scroll-aware background and Alpine mobile-menu toggle |
| `SectionNav.astro` | Floating right-side scroll-spy nav (Alpine + `IntersectionObserver`) |

`layouts/BaseLayout.astro` is the shared HTML shell: `<head>` meta + Open Graph/Twitter tags,
Inter + JetBrains Mono font links, favicons, the deferred Alpine.js CDN script, and the
reveal-on-scroll `IntersectionObserver`.

## Diagram Components

The diagram system lives in `src/components/diagram/`.

### Primitives

| Component | Key props | Purpose |
|-----------|-----------|---------|
| `DiagramFrame.astro` | `title?`, `eyebrow?`, `tone?`, `scroll?` + `legend` slot | Titled, tone-tinted card with a top accent bar, optional horizontal-scroll wrapper, and a legend slot |
| `DiagramLane.astro` | `label?`, `tone?`, `cols?` (1–5), `connector?` | A labeled responsive band of nodes with an optional animated dataflow connector below it |
| `DiagramNode.astro` | `title`, `sub?`, `icon?`, `tone?`, `variant?`, `glow?` | A single themed node box (icon + title + sub-label); `solid`/`outline` variants, optional glow |

All three accept a `tone` of `teal`, `aqua`, `violet`, `red`, `amber`, `blue`, `emerald`, or
`slate`. Full literal Tailwind class strings are spelled out per tone so the v4 scanner sees them.

### Flow diagrams (`src/components/diagram/flows/`)

22 hand-authored inline-SVG flow components, coordinate-authored to real topology, with a CSS
"flowing dash" along edges, per-domain accent tones, scoped `<defs>` arrowhead markers, and
`role="img"` + `<title>`/`<desc>` accessibility. Grouped here by the `/architecture` section that
renders them:

| Domain section | Flow components |
|----------------|-----------------|
| **Topology** | `DeploymentTopologyFlow` |
| **Edge & SD-WAN** | `FortiGateSdwanFlow`, `SdwanLoadBalanceFlow`, `ServerLoadBalanceFlow`, `SslInspectionFlow`, `Layer7FilterFlow`, `LookingGlassFlow` |
| **Virtualization** | `LiveMigrationFlow`, `IommuSplitFlow` |
| **CUDA** | `CudaDeliveryFlow`, `CudaClusterFlow`, `NvidiaContainerToolkitFlow`, `ContainerDriverSplitFlow` |
| **RAID & Redundancy** | `RaidRedundancyFlow` |
| **Backup & DR** | `BackupStrategyFlow`, `ArchBackupFlow`, `ProxmoxBackupFlow`, `VeeamBackupFlow`, `BackupEncryptionFlow` |
| **Security** | `SecurityOverlayFlow` (plus `DiagramFrame`/`Lane`/`Node` lane diagrams for distributed CrowdSec and the threat-feed pipeline) |
| **Observability** | `LogPipelineFlow`, `MetricsPipelineFlow` |

The `/architecture` page's `SectionNav` anchors are: Top, Topology, Edge & SD-WAN,
Virtualization, Workstation, Backup & DR, RAID & Redundancy, CUDA, AI, Security, Observability,
and "Follow the Data".

## Static Assets (`public/`)

Served as-is from the site root.

| Path | Purpose |
|------|---------|
| `profile.jpg` | Headshot (About section) |
| `infrastructure.png` | Infrastructure visual |
| `og-image.png` | Open Graph / Twitter social card |
| `favicon.png`, `favicon-16.png`, `favicon-32.png`, `apple-touch-icon.png` | Favicons / touch icon |
| `logos/` | Technology logos (Proxmox, Docker, NGINX, Rust, Zig, Azure, Cloudflare, Tailscale, Fortinet, etc.) |

## Contact Information

These identity values appear in `Hero.astro` and `Contact.astro`. Update them together so they
stay in sync.

| Field | Value |
|-------|-------|
| Name | Christopher Kelley |
| Email | ckelley@ghostkellz.sh |
| GitHub | [@ghostkellz](https://github.com/ghostkellz) |
| LinkedIn | [/in/christopherkelley89](https://linkedin.com/in/christopherkelley89) |
| Twitter / X | [@gh0stKellz](https://twitter.com/gh0stKellz) |

## External Services

| Service | Purpose | Notes |
|---------|---------|-------|
| Google Fonts | Inter + JetBrains Mono | Preconnected and loaded in `BaseLayout` |
| [Alpine.js](https://alpinejs.dev) CDN | Mobile nav, scroll-spy `SectionNav`, hero typing + terminal | Loaded with `defer` from jsDelivr; the only client framework |

No analytics or third-party tracking scripts are loaded. The contact path is a plain `mailto:`
link — there is no form backend.

## License

Proprietary. Copyright © 2026 Christopher Kelley / CK Technology LLC — all rights reserved. See
the repository `LICENSE` file.
