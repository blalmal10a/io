# Current tasks — 2026-10-28

## Goal

Migrate the site to Astro (static site generator) — then resume topic-by-topic content population in markdown.

## Tasks

- [x] Daily task roll (2026-10-27 → history; this file created)
- [ ] Phase 1: `.gitignore` + checkpoint commit (docs suite + mcq fixes)
- [ ] Phase 2: Astro scaffold + port (layout, styles, home, generated notes TOC, 14 markdown scaffolds)
- [ ] Phase 3: MCQ tester as island (paper filtering, correct 2-marks/−⅓ scoring, fixed state bugs)
- [ ] Build + preview + Playwright verification
- [ ] Phase 5 (if green): cutover — Herd docroot → `dist/`, retire legacy `public/` HTML
- [ ] Phase 6 (after cutover): refresh `AGENTS.md` / `ARCHITECTURE.md` / `PLAN.md` for the build-based workflow
- [ ] Choose first topic to populate (now in markdown)

## Notes / decisions

- Astro chosen (user approved 2026-10-28): Full MCQ scope (timers, stats, search) = generator territory.
- MCQ island: vanilla module script first; a UI framework only if the Full scope demands it.
- `questions.json` schema unchanged; near-duplicate ids/content (18 JSON entries vs 13 hardcoded) need a user-reviewed dedupe batch later.
- Exact Astro APIs verified via context7 at implementation time.

## Carry-over

- (user) Create a GitHub fine-grained PAT and `export GITHUB_PERSONAL_ACCESS_TOKEN=…` in `~/.zshrc` — required by the local GitHub MCP server; `gh` is unauthenticated.
- (user) Start Docker Desktop — github MCP fails until the daemon is up.
- (user) Later: create a Firebase project for hosting (Phase 7).
