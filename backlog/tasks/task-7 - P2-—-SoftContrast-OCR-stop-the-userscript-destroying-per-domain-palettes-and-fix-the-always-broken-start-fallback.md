---
id: TASK-7
title: >-
  P2 — SoftContrast OCR: stop the userscript destroying per-domain palettes, and
  fix the always-broken start fallback
status: To Do
assignee: []
created_date: '2026-11-25T11:46:52Z'
labels:
  - softcontrast
dependencies: []
ordinal: 7000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — SoftContrast OCR: stop the userscript destroying per-domain palettes, and fix the always-broken start fallback** — pro (high) · `javascript-pro`. `js/export.js:118` — when `JSON.parse` throws (corrupt storage, or a manager whose `GM_getValue` returns a non-string), `palette` falls back to `defaultPalette` and the trailing `GM_setValue` then **overwrites** the user's saved palette; the code comment says "ignore corrupted storage" but the behaviour is data loss. A stored `"null"` also parses to `null`, so the later `palette.fontSmooth` / `palette.ruler` reads throw. Distinguish corrupt from absent, merge stored values over defaults so new fields survive, and skip the write-back when the read failed. `js/export.js:153` — the `else` branch of the `document.head`/`documentElement` check is only reachable when both are falsy, so `observer.observe(document.documentElement, …)` is guaranteed to receive `null`: the "document-start fallback" always fails in the exact situation it exists for and never injects the style. Also make the `GM_addStyle` path set `dataset.softcontrast` so injected styles are tagged consistently, and guard the ruler mount (`js/export.js:175`) on `document.readyState` so a late-injected script does not leave a detached node with a live `mousemove` handler reading `offsetHeight` forever. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
