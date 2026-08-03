# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single static webpage (no build step, no dependencies) that displays the user's personal Claude skills as a searchable, expandable card list. Hosted on GitHub Pages at `https://iedoptimum.github.io/my-skills-library/`, deployed from the `main` branch root.

## Local development

There is no build, lint, or test tooling — this is plain HTML/CSS/JS with no dependencies. To preview changes, serve the directory over HTTP rather than opening `index.html` via `file://`: browsers (Chrome in particular) block `fetch()` of local files under `file://`, so `skills.json` will fail to load. Any static server works, e.g. `python -m http.server` or `npx serve`, then visit `http://localhost:<port>/`.

## Architecture

- **`index.html`** — the entire site: inline CSS and vanilla JS in one file, no framework. On load it `fetch('./skills.json', { cache: 'no-store' })`s the data and renders it client-side. `cache: 'no-store'` is intentional — GitHub Pages' default 10-minute CDN cache previously caused stale/mismatched data to render after a schema change, so every load bypasses HTTP caching to guarantee fresh data.
  - `render(data)` builds one `<section class="card">` per skill (numbered `SK-01`, `SK-02`, …), each holding a collapsible `.subs` grid of its `sub_skills`. A card's searchable text is precomputed once into `card._data` (id + title + description + sub-skill names/descriptions, lowercased) rather than re-read from the DOM on every keystroke.
  - `setupToolbar()` wires the search input (filters cards by `card._data`, auto-expands matches, highlights hits via `<mark>`) and the expand/collapse-all button.
  - All user-supplied text is passed through `escapeHTML()` before being placed in `innerHTML` — required since the content is data-driven from JSON.
  - `id` falls back to a `name` field with a stripped leading slash (`s.id || (s.name || '').replace(/^\//, '')`) — a holdover for compatibility with `personal_skills.json`'s `/skill-name` convention; new entries should just set `id`.
- **`skills.json`** — the single source of truth for what's deployed. Shape:
  ```json
  {
    "updated": "YYYY-MM-DD",
    "skills": [
      {
        "id": "skill-slug",
        "title": "Human Readable Name",
        "glyph": "🛠️",
        "description": "One or two sentences.",
        "sub_skills": [{ "name": "sub-slug", "description": "..." }]
      }
    ]
  }
  ```
  `index.html` and `skills.json` must stay in sync on this schema — renaming a key in one requires updating the other (this exact mismatch previously caused an empty-list bug). `sub_skills` may be `[]` for a self-contained skill (e.g. `writing-partner`) — `render()` swaps in a "no sub-skills" placeholder for those cards instead of an empty grid.
- **`personal_skills.json`** — the original raw data dump the user supplied from their claude.ai account, using a slightly different shape (`name: "/skill-slug"` instead of `id`/`title`). It is intentionally left untracked (not `git add`ed, though there's no `.gitignore` enforcing it) and is not fetched by the site. Treat `skills.json` as derived from it, not the other way around — when the user adds a skill, check whether it already exists here first.

## Updating the skill list

No build step. To add or change a skill:
1. Edit `skills.json` directly (add/update an entry, bump `updated`).
2. `git add skills.json && git commit -m "..." && git push`
3. GitHub Pages redeploys automatically within about a minute.

## Deployment

Pages is configured via the GitHub API (`gh api repos/iedoptimum/my-skills-library/pages`) to deploy from `main` / `/` (root) — there is no `docs/` folder or GitHub Actions workflow involved.
