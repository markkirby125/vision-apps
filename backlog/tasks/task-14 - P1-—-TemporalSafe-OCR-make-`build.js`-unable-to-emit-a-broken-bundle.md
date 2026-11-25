---
id: TASK-14
title: 'P1 — TemporalSafe OCR: make `build.js` unable to emit a broken bundle'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:20Z'
labels:
  - temporalsafe
dependencies: []
ordinal: 14000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — TemporalSafe OCR: make `build.js` unable to emit a broken bundle** — pro (high) · `javascript-pro`. `build.js:10` — the export-strip regex is incomplete for valid ES module syntax and fails silently. (1) The lookahead accepts `default` but only `export ` is removed, so `export default function foo(){}` becomes `default function foo(){}`, a SyntaxError inside the emitted IIFE. (2) Named-export lists (`export { a, b };`) and re-exports (`export * from './x.js';`) do not match the lookahead at all and pass through verbatim into both outputs. Today `temporalsafe.js` only uses `export function/const/class`, so this is latent — any refactor to one of the common forms ships a broken userscript with no build-time warning. `build.js:27` — only `export` is stripped, so any `import … from '…'` left inside the IIFE is a SyntaxError in a function body; add a post-strip validation guard on the body before writing either output. `build.js:28` — the artifact named `temporalsafe.bookmarklet.js` is plain JS with raw newlines, spaces, a leading comment and no `javascript:` scheme, so it cannot be pasted into a bookmark URL field; emit a genuinely paste-ready URI-encoded bookmarklet or rename it. Also: interpolate the version from `package.json` instead of the hardcoded `0.1.0` in `build.js:18` (userscript managers key updates off `@version` and the two will drift), extract the duplicated IIFE wrapper into one `wrap()` helper, and fail with a targeted diagnostic instead of a raw ENOENT when the source is missing or empty (`build.js:8`). Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
