# SpatialConnect multi-year site — design spec

**Date:** 2026-06-06
**Repo:** `SpatialConnect/SpatialConnect.github.io` (organization GitHub Pages site, served at `https://spatialconnect.github.io/`)

## Goal

Serve multiple workshop editions from one branch so all stay live at once:

- `https://spatialconnect.github.io/2025/`
- `https://spatialconnect.github.io/2026/`

The bare root `/` redirects to the latest edition (`/2026/`).

## Source of truth & branches

- The real, current 2025 site is on branch **`new_structure3`** (NOT `master` — `master` is stale forked content).
- Create a new branch **`main`** from `new_structure3`. This branch holds the multi-year pattern (`2025/` + `2026/`).
- GitHub Pages will need its Source pointed at `main` (manual step — see Deployment).

## Chosen model: single shared Jekyll site (no Actions)

`new_structure3` already implements this model, and it works with default GitHub Pages "build from branch":

- One `_config.yml` at the repo root with `baseurl: ''`.
- Shared theme at the root: `_layouts/`, `_includes/`, `_sass/`, `assets/`, `images/`.
- Each edition's **content pages** live in a year folder. Page URLs derive from the folder path, so `2025/program/index.md` → `/2025/program/`, `2026/program/index.md` → `/2026/program/`. No per-page `permalink` needed.
- A root `index.html` does a meta-refresh redirect to the latest edition.

Rejected alternative: isolated per-year sites + GitHub Actions. More work, would rework already-working code, and requires switching the Pages source to "GitHub Actions". Not warranted.

## What already works per-year (no change needed)

- **Page banners** use each page's own front matter (`page.header.title`, `page.header.title2`, `page.header.image_fullwidth`). 2026 pages set their own.
- **`site.workshop` / `site.organizers`** blocks in `_config.yml` are vestigial — not referenced in any layout/include. Content is hardcoded in the markdown.

## The one coupling to fix: navigation

[`_includes/_navigation.html`](../../../_includes/_navigation.html) loops over a single global [`_data/navigation.yml`](../../../_data/navigation.yml) whose links are hardcoded to `/2025/...`. Copying the folder as-is would give 2026 pages a menu that points back into 2025.

**Fix — year-aware navigation:**

1. Replace `_data/navigation.yml` with per-year files: `_data/navigation_2025.yml` and `_data/navigation_2026.yml` (each with that year's links: Home/Organizers/Program/Contacts pointing at `/<year>/...`, plus the external ACM SIGSPATIAL link).
2. In `_navigation.html`, derive the current page's year from the first path segment of `page.url` (e.g. `/2026/program/` → `2026`), then select `site.data["navigation_<year>"]` for the menu loop. Fall back to the latest year if no segment matches.

This keeps each edition's menu self-contained and lets future years' menus diverge (e.g. add Registration).

## Implementation steps

1. From `new_structure3`, create branch `main`.
2. `cp -r 2025/ 2026/` (duplicate content pages).
3. Leave 2026 pages as a working duplicate of 2025 (a template skeleton). The folder path makes URLs `/2026/...` automatically. **Content is intentionally NOT rewritten here** — a separate person will fill in the real 2026 details later. The only goal of this task is the structure/routing so both editions are live at once.
4. Split navigation into `navigation_2025.yml` + `navigation_2026.yml`; edit `_navigation.html` to select by page year (both nav loops in the file).
5. Update root `index.html` redirect target from `/2025/` → `/2026/`. `/2025/` stays directly reachable.
6. Update `CLAUDE.md` to describe the new multi-year structure (the existing draft documents the stale `master` layout).
7. Exclude this `docs/` spec folder from the Jekyll build (add `docs` to `exclude` in `_config.yml`) so it isn't published.

## Deployment (manual step, by repo owner)

In **Settings → Pages**, set **Source = Deploy from a branch**, **Branch = `main`** (root). Optionally make `main` the default branch. No GitHub Actions required.

## Adding future years (2027+)

Copy the newest year folder → `2027/`, update its content, add `navigation_2027.yml`, and repoint the root `index.html` redirect to `/2027/`. The year-aware nav picks it up automatically.

## Out of scope

- No redesign of the theme.
- **No 2026 content authoring** — 2026 pages are an exact template copy of 2025; a separate person writes the real content later.
- No dependency/Gemfile changes unless a build error forces one.
