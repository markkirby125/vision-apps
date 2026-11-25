---
id: TASK-11
title: >-
  P2 — SoftContrast OCR: pin the codec and tier contracts in
  `tests/storage.test.mjs`
status: To Do
assignee: []
created_date: '2026-11-25T11:47:10Z'
labels:
  - softcontrast
dependencies: []
ordinal: 11000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — SoftContrast OCR: pin the codec and tier contracts in `tests/storage.test.mjs`** — pro (low) · `javascript-pro`. The round-trip suite only uses names that pass `sanitizePresetName()` unchanged, so the deliberately lossy contract (character stripping, 40-char cap, `'A;B%C#D'` → `'ABCD'`, fully-stripped name → `'#v1;;00'`) is untested — a regression that stopped stripping would let a raw `;` or `%` into the hash and pass. Flag-bit coverage tests only `'10'` and `'01'`, missing `'11'`, `'00'` and non-binary garbage. Truncated-hash paths are unpinned (`#v1` must be rejected; `#v1;X` takes the `parts[2] || '00'` default). Fatigue tiers are sampled only at interior points, so an off-by-one at any threshold (`< 45` → `<= 45`) passes; add values either side of 45/75/90, and derive the arbitrary `130` upper bound from the documented Lc-120 saturation point. Consolidate the four duplicate `decodeState` calls into a table and merge the redundant `assert.doesNotThrow`. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
