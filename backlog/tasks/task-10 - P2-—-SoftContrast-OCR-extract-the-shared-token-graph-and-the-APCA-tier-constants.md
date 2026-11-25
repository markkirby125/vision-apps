---
id: TASK-10
title: >-
  P2 — SoftContrast OCR: extract the shared token graph and the APCA tier
  constants
status: To Do
assignee: []
created_date: '2026-11-25T11:47:04Z'
labels:
  - softcontrast
dependencies: []
ordinal: 10000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — SoftContrast OCR: extract the shared token graph and the APCA tier constants** — pro (low) · `javascript-pro`. `js/export.js:1` — the `bg`/`text`/`accent` → primitive/semantic/component mapping is hand-repeated in `generateCSS`, `generateDTCG`, `generateTailwind`, `generateStylus` and again inside `generateUserscript`'s sheet (which already hardcodes its own `a:hover`), so one token rename needs five edits and will drift; make one table the single source of truth. `js/features.js:1` — the business-critical APCA thresholds (Lc 45/75/90), band widths (30/15/30) and 25 %-per-tier increment are inline magic numbers repeated across four branches plus the description strings, and the top tier's copy ("At or above APCA Lc 90") disagrees with its own `lc > 90` guard. Hoist a tier table and derive the labels from it. `js/features.js:36/70` — injected DOM dependencies are dereferenced without validation; validate once at factory time with an actionable error. `js/app.js` — `.then().catch()` on the clipboard path should be `async`/`await` (project standard), `buttonsContainer` is the one unguarded DOM lookup (`:75`) and can abort the whole bootstrap, and `restoreFromHash` leaves a stale invalid preset name in the URL after the unknown-preset fallback (`:95`). Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
