---
id: TASK-25
title: 'P2 — ChromaCalm OCR: delete or wire up the inert `.github/milestones.yml`'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:42Z'
labels:
  - chromacalm
dependencies: []
ordinal: 25000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — ChromaCalm OCR: delete or wire up the inert `.github/milestones.yml`** — flash (low) · `surgical-patch`. Same defect class as FocusBeacon: not a GitHub-recognised config path, no workflow or script consumes it (only `IMPLEMENTATION_PLAN.md` references it), so the four milestone definitions will never be created. Delete it, or add an `actions/github-script` / `gh api` step that parses the file and creates the milestones and document that dependency. If a Settings-style consumer is intended, both filename and schema are wrong (Probot expects `.github/settings.yml` with a top-level `milestones:` key wrapping the list).
<!-- SECTION:DESCRIPTION:END -->
