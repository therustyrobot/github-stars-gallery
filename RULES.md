# Rules — AI Stars Gallery

## Static Site Constraints

- Never introduce a build step. Tailwind CDN only — never `npx tailwindcss`, never `node_modules`.
- `docs/index.html` is the only committed output file. `_data/*.json` are ephemeral and must stay in `.gitignore`.
- No JavaScript framework. Client-side JS (`search.js`) must work without a bundler or module system.

## GitHub Actions

- All auto-commits must include `[skip ci]` in the message — prevents infinite workflow loops.
- `STARS_TOKEN` must be set as a **step-level** env override on the fetch step only, not at the workflow level. `GITHUB_TOKEN` is used for the Models API and commit steps.
- Never hard-code the Models endpoint — use `https://models.github.ai/inference/chat/completions`.

## AI Categorization

- Always batch at 10 repos per API call. Never 1 call per repo — 150 req/day free tier.
- Slug derivation must happen Python-side via `category_to_slug()`. Never use the slug field from model output — it drifts between runs and breaks bookmarks.
- Use `starred_repos.md` as the taxonomy seed on every run to prevent category drift.

## HTML Generation

- Data attributes (`data-name`, `data-desc`, `data-category`, `data-filter`) must use raw strings, not double-HTML-escaped values. Escaping `&` breaks search matching.
- Category slugs must be deterministic across runs — derived from canonical names, never regenerated arbitrarily.

## Testing

- TDD for changes to `generate.py`, `fetch_stars.py`, or `categorize.py` — write assertions before modifying behavior.
- Use fixture responses for GitHub Models API in tests — do not make live API calls.
