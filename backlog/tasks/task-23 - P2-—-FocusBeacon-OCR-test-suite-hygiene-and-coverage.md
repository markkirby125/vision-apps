---
id: TASK-23
title: 'P2 — FocusBeacon OCR: test-suite hygiene and coverage'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:39Z'
labels:
  - focusbeacon
dependencies: []
ordinal: 23000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — FocusBeacon OCR: test-suite hygiene and coverage** — pro (low) · `javascript-pro`. `tests/focusbeacon.test.mjs:11` — `createWindow` builds a fresh JSDOM per test and never closes it, so every window keeps its listeners, DOM and any library timer alive for the whole process; register `window.close()` teardown. `:73` — the readyState fallback conflates "init deferred" with "the script threw": under `runScripts: 'dangerously'` an exception is reported to the VirtualConsole and swallowed by default, so the test fails later with a misleading "missing #focusbeacon-ring" instead of the real error; wire a VirtualConsole that fails loudly. `:124` — both synthetic keydowns run in the same tick, so the double-press test passes only incidentally (`Date.now()` delta of 0) and has no expiry case; control the clock and assert the radar stays hidden beyond 350 ms. `:66` — `devMode` and `trailLength` are never exercised, so the trail, dev HUD, saccade and skip-link arrow have zero coverage; test them or drop the options. `:23` — the `CSS.supports` mock has a dead branch (both paths return false) and is skipped whenever jsdom already exposes `CSS.supports`; assign unconditionally since each JSDOM has its own realm. Also mark the purely visual overlays `aria-hidden="true"` (`focusbeacon.js:34`) to match the skip arrow.
<!-- SECTION:DESCRIPTION:END -->
