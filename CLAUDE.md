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
  - `.card-head` sets `user-select: none` so tapping to expand/collapse on mobile doesn't also select text; `.name` and `.sid` explicitly override this back to `user-select: text` so the skill title/id stays copyable. Any new text added inside `.card-head` needs the same override or it silently becomes unselectable on mobile.
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
- **`personal_skills.json`** — a transient staging file, not a second source of truth. The user drops a fresh raw data dump from their claude.ai account here, shaped as `{ "personal_skills": [{ "name": "/skill-slug", "description": "...", "sub_skills": [...] }] }` (top-level key `personal_skills`, not `skills`; `name` instead of `id`/`title`, still slash-prefixed). It is intentionally left untracked (not `git add`ed, though there's no `.gitignore` enforcing it) and is not fetched by the site. `skills.json` is always the file to read from for what's actually deployed — `personal_skills.json` only exists to be diffed against it and merged in, then deleted.

## Updating the skill list

`skills.json` is the single source of truth for what's deployed — always read/edit it directly, not `personal_skills.json`.

When the user says "update my skills" (or similar) and `personal_skills.json` is present, that means they've dropped a fresh export to merge in:
1. Diff `personal_skills.json` against `skills.json` (by id/name, and by each skill's `sub_skills` — they can drift at the sub-skill level, e.g. a sub-skill added upstream but not yet ported).
2. Port every difference into `skills.json` (add/update entries, bump `updated`), preserving `skills.json`'s existing style (concise `title`/`description`, hyphenated sub-skill slugs like `soccer-session-plans`, not `personal_skills.json`'s slashed/verbose ones).
3. Delete `personal_skills.json` — once its contents are folded into `skills.json`, it has served its purpose and should not stick around.
4. `git add skills.json && git commit -m "..." && git push` (only `skills.json`; `personal_skills.json` is untracked and now deleted).
5. GitHub Pages redeploys automatically within about a minute.

For any other skill edit (no `personal_skills.json` involved), just edit `skills.json` directly and follow steps 4–5.

## Deployment

Pages is configured via the GitHub API (`gh api repos/iedoptimum/my-skills-library/pages`) to deploy from `main` / `/` (root) — there is no `docs/` folder or GitHub Actions workflow involved.
