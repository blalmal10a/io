# AGENTS.md

## Purpose

Static study portal for the **Mizoram ICT Department Informatics Officer (Level-10) recruitment exam**: syllabus-structured study notes plus an MCQ practice tester. Hand-maintained HTML/CSS/JS — no build step, package manager, CI, or lint.

## Technical definition

- **Web root:** `public/` — Laravel Herd serves it locally at `http://io.test` (verified). Planned deploy: Firebase Hosting with `public` as the site root (not yet configured). All links and asset paths must stay relative.
- **Serving:** already running via Herd. Do **not** start ad-hoc servers (`python -m http.server`, `npx serve`, …). Check pages with `curl`:
  - `curl -s io.test` — homepage
  - `curl -sI io.test/data/questions.json` — MCQ data (expect `200`, `application/json`)
- **Verification:** no test tooling exists. Verify by curling changed pages; for browser behavior (MCQ tester JS) use the Playwright or chrome-devtools MCP against `io.test`. Validate JSON with `jq . public/data/questions.json`.
- **Layout:** top-level pages in `public/*.html` (`index.html`, `mcq.html`); one notes page per syllabus topic in `public/notes/*.html`; shared stylesheet `public/assets/style.css`; MCQ bank `public/data/questions.json`; source of truth `public/syllabus.md` (parsed from the PDF next to it).
- **Git:** remote `github.com/blalmal10a/io` (SSH). `gh` CLI is installed but unauthenticated.

## Doc map — read before working

| File | Contents |
| --- | --- |
| `ARCHITECTURE.md` | Full structure: layout, data flow, schemas, CSS vocabulary, known defects |
| `PLAN.md` | Hard rules. Never violate. |
| `tasks/current_task-YYYY-MM-DD.md` | Today's tasks (auto-loaded into every session) |

All three are auto-loaded via `instructions` in `opencode.json`.

## Daily task roll

At session start, if today's date differs from the existing `tasks/current_task-*.md`:

1. Move the old file to `tasks/history/<that-date>.md`.
2. Create `tasks/current_task-<today>.md`, carrying over unfinished items.

`tasks/history/` is append-only.
