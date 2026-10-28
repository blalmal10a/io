# ASTRO.md — Astro migration plan

**Status: approved by user 2026-10-28, implementation not started.** This file is the source of truth for the planned migration. Execute only on the user's explicit go-ahead. When the migration completes, fold the verified facts into `ARCHITECTURE.md` (PLAN.md rule 9) and mark this file's phases done.

## Why Astro (decided 2026-10-28)

- Full MCQ scope requested (timers, score stats, cross-note search, review modes) = generator territory.
- Notes become **markdown content collections** — cleaner for agent-authored notes, reviewable diffs.
- Shared layout/nav rendered once — ends the hand-copied header/nav across 16 pages.
- `questions.json` becomes a **build-time import with schema validation** — wrong `answer` indexes, duplicate ids, or unknown paper names fail the build instead of silently rendering wrong.
- No runtime fetch → the docroot-dependent `../data/questions.json` fragility disappears entirely.
- Output is still a fully static site → Firebase Hosting plan unchanged.

## Target structure

```text
io/
├── package.json, astro.config.mjs, tsconfig.json
├── firebase.json                       # Phase 7: "public": "dist"
├── src/
│   ├── layouts/Base.astro              # header.site nav + container + footer (props: title, crumbs)
│   ├── styles/site.css                 # ported public/assets/style.css verbatim (same class vocabulary)
│   ├── content.config.ts               # notes collection: glob loader + zod schema
│   ├── content/notes/*.md              # 14 topic pages, frontmatter: title/paper/marks/order
│   ├── data/questions.json             # moved from public/data (schema unchanged)
│   ├── scripts/mcq.js                  # tester island logic (vanilla module script)
│   └── pages/
│       ├── index.astro                 # home (ported)
│       ├── mcq.astro                   # mounts the tester island
│       └── notes/
│           ├── index.astro             # TOC generated from collection (grouped by paper, sorted by order)
│           └── [slug].astro            # renders one markdown topic
├── public/                             # Astro static assets: syllabus.md + syllabus PDF
└── dist/                               # build output (gitignored)
```

### Notes frontmatter schema (validated at build)

```ts
{ title: string, paper: string, marks: number, order: number }
```

`paper` values frozen from `public/syllabus.md`: General English (100), General Studies (100), Technical Subject Paper-I/II/III (200). Per-topic marks: 40/60 as in the syllabus — zod schema + explicit paper/marks pair makes inventing weights impossible (enforces PLAN.md rule 4 mechanically).

## Phases

**Phase 1 — Checkpoint (DONE 2026-10-28, commit `abbda9b`)**: `.gitignore` (`node_modules/`, `dist/`, `.astro/`, `.DS_Store`, `.playwright-mcp/`), commit of docs suite + mcq state fixes.

**Phase 2 — Scaffold + port (pending)**
1. Hand-scaffold Astro (no `npm create` interactivity in a non-empty repo): `package.json` (astro latest), `astro.config.mjs` (static output), `tsconfig.json` (extends `astro/tsconfigs/base`), `src/env.d.ts`.
2. `npm install`.
3. Port `assets/style.css` → `src/styles/site.css` unchanged.
4. `Base.astro` layout; port `index.html` → `index.astro` (drop the "Style Guide" nav link).
5. Generate the 14 markdown scaffolds from `public/notes/*.html` (extract `h1` title, marks, `ul.checklist` items via a one-off node script; hardcode the slug→paper/order mapping from `notes/index.html`). Entity-decode `&mdash;`, `&amp;`, …
6. Generated notes TOC + `[slug].astro` renderer (title, marks badge, breadcrumbs, checklist → first draft outline).

**Phase 3 — MCQ tester island (pending)**
- Port the tester to `src/scripts/mcq.js` + `mcq.astro`. Preserve the working state fixes from 2026-10-27 (let-scoped questions, recomputed totals, refreshed current question).
- Add: paper selection tabs (start UI — fixes the dead-start defect), per-paper filtering (fixes the "shows all questions" defect), correct scoring (2 marks correct, −1/3 wrong — replaces the `findIndex` placeholder), result view with review list.
- Keep: keyboard ←/→, existing CSS classes (`q-card`, `option`, `explanation`, `review-item`, `tabs`).
- Defer to Phase 3.5/data pass: the 18-question bank has near-duplicate content with differing ids/answers (vs the old 13 hardcoded) — needs a user-reviewed dedupe batch before stats count on it.

**Phase 4 — Full-scope features (pending)**
- Exam timers (per-paper 3 h + section timers, localStorage-persisted), settings (shuffle, marks display), score stats/history per topic (localStorage), cross-note search via build-time generated MiniSearch index (still fully static).

**Phase 5 — Cutover (pending)**
1. Investigate the Herd site entry (config/`sites.json` or symlink) — repoint `io.test` docroot from `public/` → `dist/` (if it's a symlink, update it; otherwise ask the user to change it in the Herd UI).
2. Move `syllabus.md` + the syllabus PDF into Astro `public/`.
3. Delete legacy `public/*.html`, `public/notes/*.html`, `public/assets/`, `public/data/` (all superseded; content ported).
4. Verify: `curl` 200s for `/`, `/notes/`, `/mcq/`, `syllabus.md`, PDF at `io.test`; Playwright pass on `io.test` (same checklist as Phase-2 verification).

**Phase 6 — Docs refresh (pending, at cutover)**
- `AGENTS.md`: web root becomes `dist/` (Herd), commands `npm install` / `npm run dev` / `npm run build` / `npm run preview`; verification = build + curl + Playwright.
- `ARCHITECTURE.md`: rewrite structure sections for the src/content/island layout; drop the fetch-fragility defect (fixed by design).
- `PLAN.md`: rule 1 → "no ad-hoc servers; project tooling (`npm run …`) and Herd are the serving paths"; rule 2 → "`dist/` is the served root"; rule 6 → the mcq rewrite was commissioned via this file.
- `opencode.json`: add `watcher.ignore`: `dist/**`, `node_modules/**`, `.astro/**`.

**Phase 7 — Firebase (pending, blocked on user creating a Firebase project)**
- `firebase.json` (`"public": "dist"`, `cleanUrls`), `npx firebase-tools` deploy. Verify exact config via context7 at that time.

## Verification plan (every phase)

- `npm run build` passes with zero errors.
- `npm run preview` + Playwright MCP against the preview server: home, notes TOC, one topic page, MCQ page (select paper → answer → score → keyboard nav → result), **0 console errors**.
- After cutover: same Playwright pass against `io.test`.

## Standing decisions

- Vanilla module script for the island; add Preact/Svelte only if the Full scope demands it later.
- `questions.json` schema unchanged (PLAN.md rule 5 preserved).
- Sequencing: migration happens **before** populating any topic (scaffolds are tiny now; later the cost grows with content volume). First topic population resumes after cutover.
- `syllabus.md` remains the syllabus source of truth; content collections only restructure the notes around it.
