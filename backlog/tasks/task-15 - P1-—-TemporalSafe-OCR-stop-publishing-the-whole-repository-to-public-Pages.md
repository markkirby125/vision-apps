---
id: TASK-15
title: 'P1 — TemporalSafe OCR: stop publishing the whole repository to public Pages'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:22Z'
labels:
  - temporalsafe
dependencies: []
ordinal: 15000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — TemporalSafe OCR: stop publishing the whole repository to public Pages** — flash (low) · `surgical-patch`. `.github/workflows/pages.yml:29` — `path: .` means `upload-pages-artifact` tars and publishes the entire repo root; its internal tar only excludes VCS directories, so the live site includes `temporalsafe.js`, `build.js`, `tests/`, `package.json` and the workflow file itself (and would include `node_modules/` if it ever exists). Because `build.js` writes its bundles into the repo root beside `index.html` there is no site directory to point at — this is also fragile, since any future root file is silently published. Have `build.js` emit into `dist/` (with `index.html` copied in) and upload only that. Related at `:17`: the deploy is not gated on tests — `test.yml` also triggers on `push` to `main` and runs in parallel, so a push that breaks `npm test` still reaches production; make tests a prerequisite job or consolidate the workflows. At `:21`, `npm run build` runs without `npm ci` (works only because there are zero dependencies) — add it so the first dependency does not break the pipeline with a "module not found"; note `cache: npm` must stay off until a lockfile exists or `setup-node` fails. At `:22`, the deploy job has no `timeout-minutes` and can hold the `pages` concurrency group for 6 hours.
<!-- SECTION:DESCRIPTION:END -->
