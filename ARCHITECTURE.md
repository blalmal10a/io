# ARCHITECTURE.md

Technical structure of the IO Preparation Hub. Update this in the same session whenever the structure changes (see PLAN.md rule 9).

## Overview

Pure client-side static site. No backend, no build step, no framework. Pages are hand-written HTML sharing one stylesheet; the only runtime data is `data/questions.json`, fetched by the MCQ tester.

## Serving (verified)

- **Laravel Herd** serves this repo with **`public/` as the document root**: `http://io.test` → `public/index.html` (nginx, PHP 8.4 proxy).
- All runtime asset URLs assume this. In particular `mcq.html` fetches `../data/questions.json`, which resolves to `/data/questions.json` — correct **only while `public/` is the docroot**. Serving from the repo root would break the MCQ data load.
- Planned production: Firebase Hosting with `firebase.json` → `"public": "public"`, preserving the same paths. Not yet configured; no `firebase.json` exists.

## Directory layout

```text
public/
├── index.html                 # Home hub: topic links, quick links
├── mcq.html                   # MCQ Tester — inline vanilla-JS SPA
├── syllabus.md                # Parsed syllabus (source of truth)
├── posts-286-notified-syllabus-for-direct-recruitment-to-the-post-of-io.pdf
├── assets/
│   └── style.css              # Single shared stylesheet (all classes)
├── data/
│   └── questions.json         # MCQ bank, fetched at runtime
└── notes/
    ├── index.html             # Notes TOC (papers → topics)
    └── <topic>.html           # 14 topic pages (scaffolds today)
```

## Page depths & relative paths

Two depths; link prefixes differ:

- `public/*.html` → `assets/style.css`, `notes/index.html`, `data/…`
- `public/notes/*.html` → `../assets/style.css`, `index.html`, `../mcq.html`

When adding pages, copy the link style of the existing page at that depth.

## Syllabus structure (from `public/syllabus.md`)

Written exam 800 marks (3 h per paper) + personal interview 100 = 900 total.

| Paper | Marks | Topics |
| --- | --- | --- |
| General English | 100 | — |
| General Studies | 100 | — |
| Technical Subject Paper-I | 200 | Discrete Math 40, Computer Architecture 40, DSA 60, Operating Systems 60 |
| Technical Subject Paper-II | 200 | OOP 40, Web Technologies 60, DBMS 60, Cloud Computing 40 |
| Technical Subject Paper-III | 200 | Computer Networks 60, Software Engineering 60, AI/ML 40, Cybersecurity 40 |

## Notes page anatomy (scaffold)

Each topic page follows `public/notes/dbms.html`:

- `header.site` nav + `.breadcrumbs` line
- `.banner.info` with Phase-1 status
- `h3` "Modules (syllabus outline)" as `ul.checklist` (syllabus subtopics)
- `.placeholder` blocks marking unprompted content
- Footer citing the syllabus source

Population ("populate [Topic Name]", see PLAN.md) replaces scaffold placeholders with syllabus-aligned notes and adds matching MCQs to `data/questions.json`.

## MCQ tester (`public/mcq.html`)

Single inline `<script>`, vanilla JS, no build:

- State globals: `currentPaper`, `currentQuestionIndex`, `selectedOption`, `score`, `totalQuestions`, …
- `loadQuestions()` `fetch()`es `../data/questions.json` and replaces the hardcoded fallback array (13 hardcoded; JSON currently has 18 questions, near-duplicated content with differing ids/answers).
- Rendering builds `innerHTML` templates with inline `onclick` handlers → helper functions must stay global; the script must remain a **classic script (not `type="module"`)**.
- Keyboard: ←/→ = previous/next question.
- Hash routing (`#quiz` / `#settings` / `#result`) is advertised but broken (see defects). Settings/localStorage described in page copy are not implemented.

### `questions.json` schema

```json
{
  "id": 1,
  "paper": "General Studies",
  "topic": "Geography",
  "question": "…",
  "options": ["…", "…", "…", "…"],
  "answer": 0,
  "explanation": "…"
}
```

- `answer` is a **0-based index** into `options`.
- `paper` uses the syllabus topic name.
- Scoring convention (exam-style): **2 marks per correct, −1/3 per wrong** (stated in site copy; the implemented math is a placeholder, see defects).

## Known defects / quirks

Fixed this session (2026-10-27, small on-sight fixes per PLAN.md rule 6, browser-verified after):

- `const questions` → `let questions` — it was throwing `TypeError: Assignment to constant variable`, so the fetched JSON was always discarded and the hardcoded array won (even the catch's `questions = []` threw).
- `totalQuestions` is now recomputed after fetch (was captured from the hardcoded 13; JSON has 18).
- `currentQuestion` is now refreshed in `renderQuiz()` — it was captured at parse time, so navigation showed a new index with the old question body.

Still open (design-level or non-trivial — user go-ahead needed):

1. Dead-start state: on load the quiz stays hidden ("Select a Paper") and nothing in the UI starts it — nav links are placeholders, the hash handler throws on `document.hash.name`, `init()` never renders. Keyboard ←/→ does work.
2. Score logic placeholder: `submitAnswer()` uses `questions.findIndex(q => q.answer === selectedOption)` (matches the first question with that answer index, not the current one); `renderResult()` computes `totalCorrect = (totalQuestions - currentQuestionIndex) + (currentQuestionIndex + 1)` (always `totalQuestions + 1`).
3. Alpine.js loaded from unpkg (`https://unpkg.com/alpinejs@3.x.x/dist/alpine.js`) but never used — no `x-*` attributes anywhere.
4. `mcq.html` nav links all point to `index.html`; questions are not filtered by `currentPaper` (every "paper" shows all questions).
5. `fetch("../data/questions.json")` works only because the docroot is `public/`. Fragile by design; keep it and keep the docroot assumption.
6. Settings/localStorage described in page copy are not implemented; no favicon (404).

## Styling

One stylesheet: `public/assets/style.css`. CSS variables in `:root` (`--bg`, `--card`, `--accent`, `--ok`, `--bad`, `--warn`, …). Reuse the existing class vocabulary — `card`, `banner info/warn`, `checklist`, `placeholder`, `toc`, `breadcrumbs`, `badge`, `btn`, `q-card`, `option`, `explanation`, `review-item`, `grid-2`, `row`, `tabs`, `muted`, `small`, `section-title` — instead of adding per-page CSS.

## Deployment

Target: Firebase Hosting with `public` as site root (not set up). Git remote: `git@github.com:blalmal10a/io.git`. `gh` CLI installed but unauthenticated.
