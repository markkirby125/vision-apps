---
id: TASK-32
title: >-
  P2 — Keratoscope OCR: pin the `profile.js` contracts in
  `tests/profile.test.mjs`
status: To Do
assignee: []
created_date: '2026-11-25T11:47:54Z'
labels:
  - keratoscope
dependencies: []
ordinal: 32000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — Keratoscope OCR: pin the `profile.js` contracts in `tests/profile.test.mjs`** — pro (low) · `javascript-pro`. `:20` — the clamp tests only use far-out-of-range values (99, −99, 1.5, −0.1); the inclusive boundaries (±8, opacity 0/1, blur 0/4, deltas −100/300 and −0.05/0.15) are never exercised and there is no in-range pass-through assertion, so an off-by-one in `clamp()` passes; `fontWeightDelta`/`letterSpacingDelta` clamping is entirely untested. `:35` — the non-numeric fallback covers 4 of 6 numeric fields, and ±Infinity (rejected by `Number.isFinite` but accepted by a `typeof`-only check, then clamped to the max) is untested. `:51` **[security]** — the injection guard only scans numeric output for `<script` and `url(`; the real defence is `normalizeProfile` rejecting non-finite input, so a regression that starts coercing (e.g. `Number(input)`, or accepting numeric strings) would let attacker-controlled text reach the CSS body and no test would fail — add CSS-breakout string payloads. `:63` — blur-only (`dx=0, dy=0, blur>0`) is never tested and *does* emit a shadow, so tightening the condition to `dx !== 0 && dy !== 0` would silently drop blur compensation and still pass; the asymmetric-offset case is untested too. `:80` — `generateUserstyle` is only smoke-checked for header tokens and never verified to embed the generated CSS body. `:68` — the base weight `400` is hardcoded in both the implementation and the test with no shared constant.
<!-- SECTION:DESCRIPTION:END -->
