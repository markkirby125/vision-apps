---
id: TASK-6
title: >-
  P1 — SoftContrast OCR: fix non-text contrast and the invisible ruler edges in
  `css/style.css`
status: To Do
assignee: []
created_date: '2026-11-25T11:46:50Z'
labels:
  - softcontrast
dependencies: []
ordinal: 6000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — SoftContrast OCR: fix non-text contrast and the invisible ruler edges in `css/style.css`** — pro (medium) · `better-accessibility`. `css/style.css:154` — three of four fatigue-bar fills fail WCAG 1.4.11 (3:1) against the `#e9ecef` track: `#ffc107` ≈ 1.4:1, `#28a745` ≈ 2.6:1, `#17a2b8` ≈ 2.6:1 (only `#dc3545` ≈ 3.8:1 passes). The amber "elevated" state is near-invisible for exactly the audience (AMD, high myopia, photophobia) the tool serves, and the project claims WCAG 2.1 AA. Darken the fills or the track and pin the computed ratios in the palette verification. `css/style.css:267` — under `mix-blend-mode: difference` a black source is a no-op (`|backdrop − black| = backdrop`), so the 70 %-alpha black bottom border renders as the unchanged page and is invisible on every background; only the white top border ever draws. The rule also declares no `top`, so the JS-created fixed element sits at its static position until the first JS update. Prefer solid contrasting borders without blend, or a single inverted edge, plus a safe default offset. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
