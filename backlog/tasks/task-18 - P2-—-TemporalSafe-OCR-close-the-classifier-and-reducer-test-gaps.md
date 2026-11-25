---
id: TASK-18
title: 'P2 — TemporalSafe OCR: close the classifier and reducer test gaps'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:29Z'
labels:
  - temporalsafe
dependencies: []
ordinal: 18000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — TemporalSafe OCR: close the classifier and reducer test gaps** — pro (medium) · `javascript-pro`. `tests/classifier.test.mjs:77` **[high]** — the test claims "needs 3 transitions inside 1000ms" but neither boundary is pinned: the passing fixture has 4 and the failing one 2, so an off-by-one (`>= 4`) passes all three assertions, and no fixture places its earliest transition exactly 1000 ms before the last sample so the inclusive `>=`/`>` comparison is untested. Add fixtures with exactly 3 transitions and an exactly-1000 ms span. `:95` — only the single-node happy path and one reset are covered; per-node isolation, `has()`, `prune()` (the disconnected-element cleanup that guards the unbounded-map leak) and `reset()` of an untracked node are all untested. `:17` — both `'0'` and unparseable `'nope'` are pinned to return 0, enshrining an API where a typo or `var(--dur)` is indistinguishable from a legitimate zero, silently masking detection failures. `:37` — `name in attrs` consults the prototype chain, so `hasAttribute('toString')` returns true on the mock but false on a real element; use an own-property check. `:29` — the asymmetric criterion (transitions need no repetition evidence, animations do) is undocumented, and the `Number(iter) > 3` branch has zero coverage. `tests/reducer.test.mjs:26` — the mock never sets `child.parentElement`, so the `attachBadge` parent-hosting branch (badge on the parent, static parent switched to `relative`, previous position restored on undo) is never executed; the "mutes autoplay video" test uses a `<video>` — exactly a replaced tag — and therefore silently exercises the fallback path. `:30` — `remove()` only sets a flag and never detaches from `parent.children`, so no test can prove `undo()` removes the badge/cover; for this tool a permanently dark overlay that blocks visual inspection is a serious failure mode.
<!-- SECTION:DESCRIPTION:END -->
