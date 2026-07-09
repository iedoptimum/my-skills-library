# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single static webpage (no build step, no dependencies) that displays the user's personal Claude skills as a searchable, expandable card list. Hosted on GitHub Pages at `https://iedoptimum.github.io/my-skills-library/`, deployed from the `main` branch root.

## Architecture

- **`index.html`** — the entire site: inline CSS and vanilla JS in one file. On load, it `fetch('./skills.json', { cache: 'no-store' })`s the data and renders it client-side (no server, no framework). `cache: 'no-store'` is intentional — GitHub Pages' default 10-minute CDN cache previously caused stale/mismatched data to render after a schema change, so every load bypasses HTTP caching to guarantee fresh data.
- **`skills.json`** — the single source of truth for content. Shape:
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
  `index.html` and `skills.json` must stay in sync on this schema — renaming a key in one requires updating the other (this exact mismatch is what caused the empty-list bug above).
- **`personal_skills.json`** — the original raw data dump the user supplied from their claude.ai account (kept untracked, not part of the deployed site). Treat `skills.json` as derived from it, not the other way around.

## Updating the skill list

No build step. To add or change a skill:
1. Edit `skills.json` directly (add/update an entry, bump `updated`).
2. `git add skills.json && git commit -m "..." && git push`
3. GitHub Pages redeploys automatically within about a minute.

## Deployment

Pages is configured via the GitHub API (`gh api repos/iedoptimum/my-skills-library/pages`) to deploy from `main` / `/` (root) — there is no `docs/` folder or GitHub Actions workflow involved.
