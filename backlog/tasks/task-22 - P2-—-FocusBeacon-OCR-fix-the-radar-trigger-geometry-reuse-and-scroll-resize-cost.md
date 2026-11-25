---
id: TASK-22
title: >-
  P2 — FocusBeacon OCR: fix the radar trigger, geometry reuse and scroll/resize
  cost
status: To Do
assignee: []
created_date: '2026-11-25T11:47:37Z'
labels:
  - focusbeacon
dependencies: []
ordinal: 22000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — FocusBeacon OCR: fix the radar trigger, geometry reuse and scroll/resize cost** — pro (medium) · `javascript-pro`. `focusbeacon.js:296` — Ctrl double-press detection ignores key repeat: holding Ctrl auto-repeats `keydown`, and after the ~500 ms repeat delay the 2nd/3rd repeats fall inside the 350 ms window, so the radar fires from a single press; add an `e.repeat` guard. `:311` — `pointerX`/`pointerY` start at `0` and are only updated on `pointermove`, so a keyboard-only user (the primary audience) double-tapping Ctrl gets the radar painted in the top-left corner; default to the viewport centre or the last focused rect behind a `hasPointerMoved` flag. `:316` — `triggerRadar()`, `renderTrail()` and `renderSaccade()` each construct a fresh `matchMedia('(prefers-reduced-motion: reduce)')` even though `init()` already tracks the query and keeps `window.FocusBeacon.isReducedMotion` in sync; use the tracked one. `:435` — `renderTrail()` wipes `trailContainer.innerHTML` and recreates every halo plus a new Web Animation on each push, which cancels the in-flight animations (only `oncancel` fires, so older halos pop) and restarts the stagger; append one halo per push and prune the oldest. `:176` — the capture-phase `scroll` listener is unthrottled and forces synchronous layout via `getBoundingClientRect` per event (plus a second read in `updateDevHud`); coalesce into one rAF pass and mark the listeners passive. `:473`/`:27` — the rect-centre maths is duplicated and `lastKeyTime`/`lastPointerTime` are assigned but never read. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
