# AI Stars Gallery

A static web page on GitHub Pages showing the owner's GitHub starred repos organized into AI-generated categories. A daily GitHub Action fetches stars, uses GitHub Models to categorize every repo, regenerates the HTML, and auto-commits — zero maintenance after setup.

**Live:** https://therustyrobot.github.io/ai-tools-ref/

## Tech Stack

- **Python 3.12** — `generate.py` (HTML generator), `scripts/fetch_stars.py` (API fetch), `scripts/categorize.py` (AI categorization)
- **GitHub Actions** — daily cron at 06:00 UTC + `workflow_dispatch`; 7-step pipeline
- **GitHub Models** (gpt-4o-mini) — AI categorization via Bearer auth; free tier 150 req/day
- **Tailwind CDN** — styling only; never `npx tailwindcss` or a build step
- **GitHub Pages** — static HTML hosting; no server

## Architecture

**Single-file output:** `generate.py` reads `_data/categories.json` (ephemeral, gitignored) and writes `docs/index.html`. No templates, no Jinja2 — pure Python f-strings.

**Pipeline:** Fetch stars (`STARS_TOKEN`) → Categorize with GitHub Models (`GITHUB_TOKEN`) → Generate HTML → Commit with `[skip ci]`

**Client-side search/filter (v1.1):** `docs/search.js` — no build step. `generate.py` emits `data-name`, `data-desc`, `data-category` on repo cards and `data-filter` on sidebar buttons. Search and category filter AND-compose.

## Key Constraints

- **Static HTML only** — GitHub Pages; no server, no build step, no `node_modules`
- **Tailwind CDN only** — inline `tailwind.config` block; never `npx tailwindcss`
- **GitHub Models free tier** — 150 req/day; batch at 10 repos/call (≤37 calls for ~370 repos)
- **Token split** — `STARS_TOKEN` PAT (step-level override) for fetch; `GITHUB_TOKEN` for Models API + commit
- **`[skip ci]` on auto-commits** — mandatory; prevents infinite Action loop
- **Stable category slugs** — `category_to_slug()` derives slugs Python-side; never from model output
- **`_data/` gitignored** — only `docs/index.html` is ever committed

## Testing

Run: `pytest`

- `tests/test_generate.py` — HTML generation, data attribute presence, UX elements
- `tests/test_fetch.py` — fetch pipeline
- `tests/test_categorize.py` — AI categorization logic

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| GitHub Models for AI | Free with GitHub account; no external API keys; works natively in Actions |
| Full re-categorization on every run | Consistent taxonomy; simpler than diff logic |
| Python-side slug derivation | Prevents anchor ID drift from model output variation |
| BATCH_SIZE = 10 repos/call | Stays within 150 req/day free tier |
| `starred_repos.md` as taxonomy seed | Prevents category drift between runs |
| Bearer auth for GitHub Models | Different endpoint from REST API — `Authorization: Bearer {token}` |
