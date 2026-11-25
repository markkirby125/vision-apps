---
id: TASK-35
title: P2 — Resume the OCR sessions for the files skipped on quota/deadline
status: To Do
assignee: []
created_date: '2026-11-25T11:47:59Z'
labels:
  - cross-project-ocr-follow-ups
dependencies: []
ordinal: 35000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — Resume the OCR sessions for the files skipped on quota/deadline** — pro (medium) · `code-review-and-quality`. Substantial source went unreviewed: terminal-a11y 15 of 19 files (including `ansi.py`, `color.py`, `cli.py`, `budget.py`, `__init__.py`, `detection.py`, `errors.py`, `screen_reader.py`), chromacalm `chromacalm.html` + `tests/chromacalm.test.mjs`, softcontrast `js/oklch.js` + `js/presets.js`, keratoscope `index.html` + `profile.js`, temporalsafe `temporalsafe.js` + `package.json`, focusbeacon 6 files. Session IDs and the exact `ocr scan --resume` commands are in `HANDOVER.md`. Note `ocr review` does not support workspace resume — `ocr scan --resume <id>` does.
<!-- SECTION:DESCRIPTION:END -->
