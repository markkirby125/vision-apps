---
id: TASK-33
title: 'P3 — Keratoscope OCR: manifest engine floor'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:56Z'
labels:
  - keratoscope
dependencies: []
ordinal: 33000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — Keratoscope OCR: manifest engine floor** — flash (low) · `surgical-patch`. `package.json:5`/`:7` — no `engines`, and the test script depends on the shell expanding `tests/*.test.mjs`; on Windows cmd/PowerShell never expand globs, so the literal string is only usable if the runner itself supports globs (Node ≥21). Use `node --test tests/` (portable back to Node 18) and declare the matching floor; on POSIX an empty tests directory currently passes the literal pattern through and fails confusingly.
<!-- SECTION:DESCRIPTION:END -->
