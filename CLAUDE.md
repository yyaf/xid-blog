# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm install          # Install dependencies (requires pnpm)
pnpm dev              # Dev server at http://localhost:4321
pnpm build            # Production build to dist/
pnpm postbuild        # Pagefind search index (run after build)
pnpm preview          # Preview production build locally
pnpm check            # Type-check with astro check + biome lint
pnpm lint             # Auto-fix lint issues with Biome
pnpm format           # Format with Prettier
```

## Architecture

This is a static blog built on **Astro 6** with the **astro-theme-cactus** theme (minimal blog with dark/light toggle). Custom domain: `xid.pp.ua`. Deployed to GitHub Pages via `withastro/action@v3`.

### Content Collections (`src/content.config.ts`)

Three collections, all using glob loader:

| Collection | Path | Key frontmatter |
|---|---|---|
| **post** | `src/content/post/**/*.{md,mdx}` | `title` (≤60c), `description` (required), `publishDate`, `tags[]`, `draft`, `coverImage`, `pinned` |
| **note** | `src/content/note/**/*.{md,mdx}` | `title`, `publishDate` (ISO 8601), `description` (optional) |
| **tag** | `src/content/tag/**/*.{md,mdx}` | `title` (optional), `description` (optional) — overrides auto-generated tag pages |

Tags are auto-lowercased and deduplicated. `draft: true` posts are excluded from production builds.

### Page Layouts

- **`src/layouts/Base.astro`** — Root HTML shell: `<head>` via `BaseHead.astro`, `ThemeProvider` (dark/light init), header, footer, skip-link. Sets `max-w-3xl`, font-mono, global CSS variables.
- **`src/layouts/BlogPost.astro`** — Wraps `Base.astro`; renders Masthead (date/tags/reading-time), TOC (desktop sticky), post body, Webmentions (if configured). Reading time comes from `remarkReadingTime` plugin via `remarkPluginFrontmatter`.

### Key Config Files

- **`src/site.config.ts`** — Exports `siteConfig` (url, title, author, lang/ogLocale, date options), `menuLinks` (nav items), `expressiveCodeOptions` (dracula + github-light themes).
- **`astro.config.ts`** — Integrations: Expressive Code, astro-icon, sitemap, MDX, astro-robots-txt, astro-webmanifest. Custom `rawFonts` Vite plugin inlines `.ttf`/`.woff` as base64.
- **`tailwind.config.ts`** — `@tailwindcss/typography` plugin with prose overrides for code, blockquote, tables, admonitions, github-cards.

### Custom Remark/Rehype Plugins (`src/plugins/`)

Order matters — they run sequentially in `astro.config.ts`:

**Remark (MDAST → MDAST):**
1. `remark-reading-time` — adds `readingTime` to frontmatter
2. `remark-directive` — parses `:::` directive syntax
3. `remark-github-card` — `::github{repo="owner/repo"}` → GitHub repo card
4. `remark-admonitions` — `:::note/tip/important/caution/warning` callouts
5. `remark-gemoji` — `:shortcode:` emoji via `gemoji`/`emoji-regex`
6. `remark-unknown-directives` — unhandled `:::` back to markdown text
7. `remark-math` — LaTeX math parsing

**Rehype (HAST → HAST):**
1. `rehype-heading-ids` + `rehype-autolink-headings` — heading anchors
2. `rehype-title-figure` — images with `title` attr → `<figure>` with `<figcaption>`
3. `rehype-external-links` — `target="_blank" rel="noreferrer noopener"`
4. `rehype-unwrap-images` — unwrap lone images from paragraphs
5. `rehype-pixelated` — images with `#pixelated` alt → `data-pixelated` attr (CSS: `image-rendering: pixelated`)
6. `rehype-katex` — render LaTeX to HTML

### Search (Pagefind)

`Search.astro` component uses `@pagefind/default-ui`. Search index is built via `pnpm postbuild` (runs `pagefind --site dist`). Only pages with `data-pagefind-body` attribute are indexed. Search only works in production builds.

### OG Images

Generated at build time via Satori in `src/pages/og-image/[...slug].png.ts`. Template in `_ogMarkup.ts`. Assets cached aggressively (`max-age=31536000`). Posts can override with `ogImage` frontmatter field.

### Theme System

Dark/light toggle via `ThemeProvider.astro` (inline script sets `data-theme` on `<html>`). CSS variables defined in `src/styles/global.css` with `[data-theme='dark']` / `[data-theme='light']` selectors. Expressive Code themes: dracula (dark), github-light (light).

### Deployment (`.github/workflows/deploy.yml`)

Triggered on push to `main`. Uses `withastro/action@v3` (auto-detects pnpm) → `actions/deploy-pages@v4`. CNAME file at `public/CNAME` sets the custom domain.
