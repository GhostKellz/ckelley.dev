# Development

How to set up the project locally and work on it.

## Prerequisites

- **Node.js** 18 or newer (developed on Node 24)
- **pnpm** 10 or newer — install with `npm install -g pnpm`

## Setup

```bash
pnpm install
```

## Commands

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start the dev server at `http://localhost:4321` with hot reload |
| `pnpm build` | Build the static site into `dist/` |
| `pnpm preview` | Serve the production build locally |

These are the scripts defined in `package.json` (`astro dev` / `astro build` /
`astro preview`).

## Project Layout

```
src/
├── components/
│   ├── diagram/
│   │   ├── flows/          # 22 hand-authored inline-SVG flow diagrams
│   │   ├── DiagramFrame.astro   # Titled, tone-tinted frame + legend slot
│   │   ├── DiagramLane.astro    # Labeled band of nodes + dataflow connector
│   │   └── DiagramNode.astro    # Reusable themed node primitive
│   ├── Hero.astro          # Hero: typing roles, terminal widget, particle canvas
│   ├── Navigation.astro    # Fixed top nav + Alpine mobile menu
│   ├── SectionNav.astro    # Scroll-spy section nav (Alpine + IntersectionObserver)
│   ├── About.astro
│   ├── Skills.astro
│   ├── Infrastructure.astro
│   ├── Experience.astro
│   └── Contact.astro
├── layouts/
│   └── BaseLayout.astro    # HTML shell: head meta, fonts, favicons, Alpine, reveal script
├── pages/                  # One file per route (file-based routing)
│   ├── index.astro
│   ├── architecture.astro  # Animated SVG infrastructure showcase
│   ├── ai.astro
│   ├── lab.astro
│   ├── projects.astro
│   ├── technologies.astro
│   └── education.astro
└── styles/
    └── global.css          # Tailwind v4 @theme tokens + @utility + flow-* primitives
public/                     # Static assets copied as-is (logos, images, favicons)
```

See [content.md](content.md) for the full page, component, and diagram inventory.

## Conventions

### Path aliases

`tsconfig.json` defines aliases so imports stay readable:

```ts
import BaseLayout from '@layouts/BaseLayout.astro';
import Hero from '@components/Hero.astro';
import '@styles/global.css';
```

| Alias | Resolves to |
|-------|-------------|
| `@components/*` | `src/components/*` |
| `@layouts/*` | `src/layouts/*` |
| `@styles/*` | `src/styles/*` |

### Styling

- Use Tailwind v4 utility classes directly in markup.
- Use the brand color tokens (`text-accent`, `bg-navy-light`, `text-slate-light`, etc.) defined
  in the `@theme` block of `src/styles/global.css` rather than raw hex values, so the palette
  stays consistent. See [design-system.md](design-system.md).
- For repeated patterns use the existing `@utility` classes (`section`, `container-narrow`,
  `heading`, `card`, `btn`, `btn-primary`) instead of re-typing long utility strings.
- Tailwind v4 scans source for class names; where a class is built dynamically (the diagram
  primitives map a `tone` prop to full literal class strings) keep the full strings spelled out
  so the scanner picks them up.

### Interactivity (Alpine.js)

Alpine is loaded once in `BaseLayout.astro` from the CDN with `defer`. Components use it
declaratively:

- `Navigation.astro` — `x-data="{ scrolled, open }"` for the scroll-aware background and the
  mobile menu toggle.
- `SectionNav.astro` — an `IntersectionObserver` inside `x-data` drives scroll-spy highlighting
  of the active section.
- `Hero.astro` — `x-data` blocks power the rotating role "typing" animation and the looping
  terminal widget.

Pure DOM effects that do not need Alpine (the reveal-on-scroll `IntersectionObserver`, the
particle canvas, the Infrastructure parallax) live in plain `<script>` tags.

## Common Tasks

### Add a new page

1. Create `src/pages/my-page.astro` (the filename becomes the route: `/my-page`).
2. Import and use `BaseLayout` plus `Navigation` (and `SectionNav` if the page has anchored
   sections):

   ```astro
   ---
   import BaseLayout from '@layouts/BaseLayout.astro';
   import Navigation from '@components/Navigation.astro';
   import '@styles/global.css';
   ---
   <BaseLayout title="My Page" description="...">
     <Navigation />
     <main class="pt-24">
       <!-- content -->
     </main>
   </BaseLayout>
   ```

3. Add a link to it in `Navigation.astro` (`navLinks`) if it should be in the top nav.

### Add a new diagram

1. Create `src/components/diagram/flows/MyFlow.astro`. Define a node map (`x`/`y`/`w`/`title`/
   `subs`) and an edge list, then render `<rect class="flow-node">` / `<path class="flow-edge">`
   inside an `<svg class="flow-svg" role="img">` with a `<title>`/`<desc>`.
2. Give the `<defs>` arrowhead marker a **unique id suffix** (e.g. `arrow-myflow`) so multiple
   diagrams can coexist on one page.
3. Set tone via the `--node` / `--edge` CSS custom properties on the wrapper.
4. Import and place it inside a `<section>` on the relevant page (usually
   `architecture.astro`), optionally wrapped in a `DiagramFrame`.

### Update contact / identity info

Contact details and social handles appear in `Hero.astro` and `Contact.astro`. See
[content.md](content.md#contact-information) so every reference stays in sync.

## Build Output

`pnpm build` produces a fully static `dist/` directory — HTML, CSS, and the small bit of client
JS (Alpine from CDN plus the vanilla scripts). There is no server-side runtime. See
[deployment.md](deployment.md) for shipping it.
