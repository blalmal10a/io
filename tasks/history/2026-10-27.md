# Current tasks — 2026-10-27

## Goal

Bootstrap the agent docs suite and project config; select the first topic to populate.

## Tasks

- [x] Create docs suite: `AGENTS.md`, `ARCHITECTURE.md`, `PLAN.md`, `opencode.json`
- [x] Smoke-test MCP servers against `http://io.test` (playwright / chrome-devtools)
- [ ] Discuss and choose the first topic to populate (scaffold → detailed notes + MCQs)

## Notes / decisions

- Herd serves `public/` as docroot at `http://io.test` (verified 200s, `application/json` for data).
- Deployment target: Firebase Hosting, `public` as site root (config pending).
- MCQ fetch path is docroot-dependent — keep `public/` as the root (PLAN.md rule 2).
- MCP: playwright, chrome-devtools, context7, ddg-search all connected; github pending (Docker daemon not running + PAT needed).
- Fixed 3 small `mcq.html` state bugs on sight (PLAN.md rule 6), browser-verified: `const questions` → `let`, `totalQuestions` recomputed after fetch, `currentQuestion` refreshed at render. Fetch now works (18 questions), 0 console errors.
- Open `mcq.html` issues deferred for user go-ahead: dead-start UI, score logic, hash routing, placeholder nav links, no paper filtering, unused Alpine, missing favicon. (Details: ARCHITECTURE.md.)

## Carry-over

- (user) Create a GitHub fine-grained PAT and `export GITHUB_PERSONAL_ACCESS_TOKEN=…` in `~/.zshrc` — required by the local GitHub MCP server; `gh` is unauthenticated.
- (user) Start Docker Desktop — github MCP fails until the daemon is up.
