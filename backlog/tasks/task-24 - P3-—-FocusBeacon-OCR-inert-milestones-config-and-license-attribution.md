---
id: TASK-24
title: 'P3 — FocusBeacon OCR: inert milestones config and license attribution'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:40Z'
labels:
  - focusbeacon
dependencies: []
ordinal: 24000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — FocusBeacon OCR: inert milestones config and license attribution** — flash (low) · `surgical-patch`. `.github/milestones.yml:1` — not a GitHub-native config path, and a codebase search finds no consumer: neither `build.yml` nor `pages.yml` reads it and the only reference is `IMPLEMENTATION_PLAN.md`, so all four milestones are silently inert and will never be created. The keys do match the milestones REST API, so wire up a `gh api` sync workflow or delete the file so it stops misleading contributors. `:10` — no `due_on` on any milestone, so GitHub's ordering will not reflect the intended v0.1 → v1.0 sequencing. `LICENSE:3` — the holder is "FocusBeacon Contributors" while `package.json` declares `author: Paul Kirby` and points at a personal repo; align them (the MIT body itself matches the canonical text and the 2026 year is current).
<!-- SECTION:DESCRIPTION:END -->
