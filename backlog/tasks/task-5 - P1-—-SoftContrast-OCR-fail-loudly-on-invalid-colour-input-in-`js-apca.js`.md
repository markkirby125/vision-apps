---
id: TASK-5
title: 'P1 — SoftContrast OCR: fail loudly on invalid colour input in `js/apca.js`'
status: To Do
assignee: []
created_date: '2026-11-25T11:46:48Z'
labels:
  - softcontrast
dependencies: []
ordinal: 5000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — SoftContrast OCR: fail loudly on invalid colour input in `js/apca.js`** — pro (medium) · `javascript-pro`. `parseColor()` (`:42`) accepts any 3/6/8-char string after stripping `#`, so `#zzz`, `#12 45 67` and `#abcxyz` produce `[NaN, NaN, NaN]`; NaN survives `sRGBtoY` (`Math.min`/`Math.max` do not sanitise) and `calcAPCA`'s `isNaN` guard then returns `0.0` — a legitimate "no contrast" verdict that masks bad input inside the tool that decides pass/fail. Validate with a strict hex pattern and assert every channel is finite (also in the `Array.isArray` branch). Related at `:50`: 8-digit hex has its alpha byte silently discarded, so `#00000080` is scored as opaque black and the computed Lc can flip an accessibility verdict — reject it explicitly until alpha compositing exists rather than dropping it quietly. Include the offending value in the error so callers can tell text from background. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
