---
id: TASK-4
title: >-
  P1 — SoftContrast OCR: harden the URL-hash codec and the generated-CSS
  exporters
status: To Do
assignee: []
created_date: '2026-11-25T11:46:47Z'
labels:
  - softcontrast
dependencies: []
ordinal: 4000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — SoftContrast OCR: harden the URL-hash codec and the generated-CSS exporters** — pro (high) · `javascript-pro`. Two injection paths out of the same trust gap. `js/storage.js:20` — `encodeState()` runs the preset name through `sanitizePresetName()` (charset `[a-z0-9\s-]`, trim, 40-char cap) but `decodeState()` returns `decodeURIComponent(parts[1])` verbatim, so a crafted hash (`#v1;%3Cscript%3E…`) injects arbitrary characters and length into `state.name` and bypasses the sanitiser that exists to constrain it. `js/export.js:4` — `preset.bg`/`text`/`accent` are interpolated raw into the generated CSS, and that CSS is then embedded inside the userscript's own template literal, so a backtick or `${` in a persisted preset escapes the string and executes with `GM_*` privileges; `;`, `}` or a newline corrupts every output. Note the file is already inconsistent: `defaultPalette` is embedded via `JSON.stringify`, the CSS block is not. Validate colours against a strict pattern at one shared helper and run the decoded name back through `sanitizePresetName()` (or reject hashes whose name does not survive sanitisation unchanged). Also fix the unvalidated flag segment (`js/storage.js:26` accepts any non-`1` char as false and silently truncates extra segments) and `encodeState('dark', null)` throwing on the default-parameter path (`js/storage.js:18`). Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
