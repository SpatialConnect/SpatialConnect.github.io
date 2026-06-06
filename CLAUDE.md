# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Jekyll static site** for the *SpatialConnect* ACM SIGSPATIAL workshop series, served as an **organization GitHub Pages site** at `https://spatialconnect.github.io/`.

It is a **multi-year** site: every workshop edition lives at the same time under its own year path:

- `https://spatialconnect.github.io/2025/`
- `https://spatialconnect.github.io/2026/`
- the bare root `/` redirects to the latest edition.

This is a content/config site, not an application — most edits are Markdown content and YAML data.

## Branches (important)

- **`main`** — the multi-year branch (year folders `2025/`, `2026/`, …). This is the source of truth going forward.
- `new_structure3` — the prior single-year (2025-only-folder) state that `main` was built from.
- `master` — **stale** forked content from `bigspatial2025_bakcup`. Do not use it as a reference.

GitHub Pages must be configured (Settings → Pages → Source = *Deploy from a branch*) to serve the branch that holds the year folders. No GitHub Actions are used — it's a plain build-from-branch.

## Architecture: one shared Jekyll site, content split by year

- A **single** `_config.yml` at the repo root with `baseurl: ''` (the org site is served at the domain root).
- The **theme is shared** across all years at the root: `_layouts/`, `_includes/`, `_sass/`, `assets/`, `images/`.
- Each edition's **content pages** live in a year folder: `2025/index.md`, `2025/organizer/`, `2025/program/`, `2025/contact/`, and the parallel `2026/…`. Page URLs derive from the folder path — `2026/program/index.md` → `/2026/program/` — so **no per-page `permalink` is needed**.
- `index.html` at the repo root is a meta-refresh redirect to the latest edition. **Bump its target each time a new year goes live.**

### Year-aware navigation (the one piece of per-year logic)

The top menu must point within the current edition. This is handled in [`_includes/_navigation.html`](_includes/_navigation.html):

- It derives the year from the first path segment of `page.url` (e.g. `/2026/program/` → `2026`), then loads `_data/navigation_<year>.yml` (falling back to the latest year's file).
- So each edition has its own menu file: `_data/navigation_2025.yml`, `_data/navigation_2026.yml`. Links inside each point at that year's pages plus the external SIGSPATIAL link.

Everything else is already per-year safe: page banners use each page's own front matter (`page.header.title` / `title2` / `image_fullwidth`), and the `site.workshop` / `site.organizers` blocks in `_config.yml` are **vestigial** (not referenced by any template — content is hardcoded in the Markdown).

## Adding a new edition (e.g. 2027)

1. `cp -r 2026/ 2027/` and edit the content.
2. Add `_data/navigation_2027.yml` (copy a prior year, point links at `/2027/...`).
3. Repoint the root `index.html` redirect to `/2027/`.
   The year-aware nav picks up `2027` automatically from the URL.

## Build / preview

GitHub Pages builds server-side; there is no committed `Gemfile` on `main`. To preview locally:

```bash
gem install jekyll jekyll-paginate          # one-time
jekyll build --destination /tmp/sc_site     # or: jekyll serve
```

Verify after a build: `/2025/` and `/2026/` both generate, root `index.html` redirects to the latest year, and the menu on each year's pages links only within that year.

## Conventions & gotchas

- Markdown is rendered with **kramdown**; the `{: style="..."}` blocks in content are kramdown inline-attribute syntax, not literal text.
- Theme styling is the vendored **"Feeling Responsive"** theme (compiled to `assets/css/styles_feeling_responsive.css` from `_sass/`), despite the leftover `theme: jekyll-theme-cayman` line in `_config.yml`.
- Internal links/assets in templates must go through `{{ site.baseurl }}` / `{{ site.url }}` (baseurl is empty here, but keep the pattern).
- `docs/` (internal design specs) and `CLAUDE.md` are excluded from the published site via `_config.yml` `exclude:`.
