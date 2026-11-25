---
id: TASK-13
title: 'P3 — SoftContrast OCR: CI hygiene and preview-theme scoping'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:18Z'
labels:
  - softcontrast
dependencies: []
ordinal: 13000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — SoftContrast OCR: CI hygiene and preview-theme scoping** — flash (low) · `surgical-patch`. `css/style.css:1` — the preview custom properties (`--bg-color`, `--text-color`, `--accent-color`) are declared on `:root` but consumed only inside `.preview-card`, so any future `:root` reference or JS write to `documentElement` leaks themed values page-wide; scope them and rename the accent to avoid confusion with the `accent-color` property set on `.toggle input`. `css/style.css:75` — `transition: all` interpolates the `font-weight: bold` active state through synthetic weights, forcing layout/repaint of the whole button stack per toggle; restrict to the properties that change. `css/style.css:69` — the only control sized in `px` ignores the user's browser font-size preference; `css/style.css:204` — fading a link to 80 % opacity can drop it below 4.5:1 in a palette being judged for accessibility, and the change is outside the link's `transition: color`. `.github/workflows/test.yml` — no `permissions`, no `timeout-minutes`, no `concurrency`, `npm install` instead of `npm ci`; enable `actions/setup-node` npm caching once a lockfile is committed.
<!-- SECTION:DESCRIPTION:END -->
