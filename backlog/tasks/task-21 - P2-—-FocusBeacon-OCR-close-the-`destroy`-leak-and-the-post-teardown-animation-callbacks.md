---
id: TASK-21
title: >-
  P2 — FocusBeacon OCR: close the `destroy()` leak and the post-teardown
  animation callbacks
status: To Do
assignee: []
created_date: '2026-11-25T11:47:35Z'
labels:
  - focusbeacon
dependencies: []
ordinal: 21000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — FocusBeacon OCR: close the `destroy()` leak and the post-teardown animation callbacks** — pro (medium) · `javascript-pro`. `focusbeacon.js:203` — `renderSaccade()` appends a `#focusbeacon-saccade` div to `document.body`, but `destroy()` removes beacon, radar, trail, arrow, HUD and styles and never the saccade; the element stays orphaned with a stale inline transform, and since the double-inject guard only checks `focusbeacon-ring`, a re-injected instance silently reuses the stale node. Track it in a module variable and remove it. `focusbeacon.js:331` — both `animation.onfinish` closures in `triggerRadar()` dereference the module-level `radar`, which `destroy()` sets to `null`, so a teardown during the 600 ms animation throws a TypeError and skips the rest of the cleanup; capture the element in a local `const`, guard it, feature-detect `el.animate` (this script otherwise supports legacy browsers via `mql.addListener`), and cancel running animations in `destroy()` via `getAnimations()`. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
