---
id: TASK-8
title: >-
  P2 — SoftContrast OCR: wire the fatigue meter, add a no-JS state, and disable
  inert controls
status: To Do
assignee: []
created_date: '2026-11-25T11:46:56Z'
labels:
  - softcontrast
dependencies: []
ordinal: 8000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — SoftContrast OCR: wire the fatigue meter, add a no-JS state, and disable inert controls** — pro (low) · `better-accessibility`. `index.html:48` — the visible `#fatigue-label` / `#fatigue-desc` are not programmatically associated with the `<progress>`; only `aria-label` names it, so screen readers never announce the dynamic risk value or its explanation, while redundant `aria-value{min,max,now}` copies risk divergence from `value`/`max`. Use `aria-labelledby` + `aria-describedby` and carry the numeric Lc in the label. `index.html:30` — the entire tool depends on `app.js`; with JS disabled, blocked or errored there is no `<noscript>` and no default palette, so the page is empty and unexplained. `index.html:68` — all five export buttons render enabled but do nothing until the module binds, turning a load failure into a silent no-op; ship them `disabled` and enable after hydration. `index.html:62` — `rel="noopener"` on a same-tab link is dead weight; either add `target="_blank"` (plus an SR-visible new-tab warning) or drop it. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
