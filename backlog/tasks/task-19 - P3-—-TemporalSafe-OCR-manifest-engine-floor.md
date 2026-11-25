---
id: TASK-19
title: 'P3 — TemporalSafe OCR: manifest engine floor'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:31Z'
labels:
  - temporalsafe
dependencies: []
ordinal: 19000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — TemporalSafe OCR: manifest engine floor** — flash (low) · `surgical-patch`. `package.json:7` — no `engines`, and the runner-side glob (`tests/*.test.mjs`) only expands cross-shell on Node ≥21, so on Node 18/20 it works solely because POSIX shells pre-expand it and breaks on Windows; declare `">=18.0.0"` and use `node --test tests/`, which searches recursively for `*.test.mjs` on every platform. All imports are `node:*` built-ins, so nothing needs declaring as a dependency.
<!-- SECTION:DESCRIPTION:END -->
