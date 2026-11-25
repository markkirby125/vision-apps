---
id: TASK-26
title: 'P2 — ChromaCalm OCR: CI least-privilege, timeouts and determinism'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:43Z'
labels:
  - chromacalm
dependencies: []
ordinal: 26000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — ChromaCalm OCR: CI least-privilege, timeouts and determinism** — flash (low) · `surgical-patch`. `.github/workflows/test.yml:3` — no `permissions` block, so `GITHUB_TOKEN` falls back to the repo default, which may be broad (read/write); a check-out-and-test job needs `contents: read`. `:10` — neither `test.yml` nor `pages.yml` declares `timeout-minutes`, so an interactive prompt or a stalled upload holds a runner for up to 360 minutes. Add tight per-job timeouts (a few minutes for the suite, ~10–15 for the static deploy). `:9` — no `concurrency` group, so rapid PR commits queue redundant runs. `:19` — `npm install` is non-deterministic in CI without a lockfile; use `npm ci` and commit the lockfile, which also enables `actions/setup-node` npm caching (`:15`).
<!-- SECTION:DESCRIPTION:END -->
