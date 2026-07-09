# My Claude Skills

A live directory of my personal Claude skills, hosted on GitHub Pages.

## Updating

1. Edit `skills.json` — add or update a skill entry (`id`, `title`, `glyph`, `description`, `sub_skills`) and bump the `updated` date.
2. Commit and push:
   ```
   git add skills.json
   git commit -m "Update skills"
   git push
   ```
3. GitHub Pages redeploys automatically within about a minute.

No build step — `index.html` fetches `skills.json` directly at page load.
