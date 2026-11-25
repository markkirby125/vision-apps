---
id: TASK-31
title: >-
  P2 — Keratoscope OCR: publish only the site assets and gate the deploy on
  tests
status: To Do
assignee: []
created_date: '2026-11-25T11:47:52Z'
labels:
  - keratoscope
dependencies: []
ordinal: 31000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — Keratoscope OCR: publish only the site assets and gate the deploy on tests** — flash (low) · `surgical-patch`. `.github/workflows/pages.yml:28` — `upload-pages-artifact` excludes only `.git` and `.github`, so `tests/profile.test.mjs`, `package.json`, `README.md` and `.gitignore` are all deployed to the public site, while the actual site is just `index.html`, `profile.js` and `llms.txt`; any future root file is silently published too. Stage only the site assets into a directory and upload that. `:17` — the deploy is not gated on tests; `test.yml` also triggers on `push` to `main` and runs in parallel, so a push that breaks the calibration math in `profile.js` still reaches production. `:22` — no `timeout-minutes`; a hung deploy holds the `pages` concurrency group for 6 hours.
<!-- SECTION:DESCRIPTION:END -->
