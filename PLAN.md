# PLAN.md — hard rules for agents

These rules were agreed with the user. Never violate them; ask first if a task seems to require an exception.

1. **No ad-hoc servers.** The site is served by Laravel Herd at `http://io.test` with `public/` as document root. Never start `python -m http.server`, `npx serve`, or anything similar.
2. **`public/` is always the docroot.** Relative paths — including `mcq.html`'s `../data/questions.json` fetch — depend on it. Never reorganize `public/` without updating every link, the fetch path, and `ARCHITECTURE.md`.
3. **Topic-by-topic content.** Notes and MCQ test data are built one topic at a time, after discussing with the user. Never mass-populate multiple topics unprompted. Content must be syllabus-aligned (source: `public/syllabus.md`).
4. **Syllabus is frozen.** Use the topic list and mark weights from `public/syllabus.md`. Do not restructure papers/topics or invent mark weights.
5. **MCQ data integrity.** Preserve the `questions.json` schema exactly (`answer` is a 0-based index into `options`; `paper` uses the syllabus topic name). Exam scoring convention: 2 marks per question, 1/3 negative marking.
6. **No wholesale `mcq.html` rewrites.** Small verified bugs may be fixed on sight and flagged in the response; larger changes need the user's explicit go-ahead.
7. **Write only inside this project directory.** Reads anywhere on the system are allowed; edits/writes outside this repo are not.
8. **Daily task roll is mandatory** on date change (see AGENTS.md). `tasks/history/` is append-only — never edit or delete archived task files.
9. **Keep docs truthful.** When structure changes, update `ARCHITECTURE.md` in the same session. Never document something unverified.
