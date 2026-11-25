---
id: TASK-27
title: 'P3 — ChromaCalm OCR: ignore-file and deploy-script polish'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:45Z'
labels:
  - chromacalm
dependencies: []
ordinal: 27000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P3 — ChromaCalm OCR: ignore-file and deploy-script polish** — flash (low) · `surgical-patch`. `.gitignore:13` — `package.json` is tracked while `package-lock.json` is ignored, so installs drift between machines and reproducible testing is lost; commit the lockfile, or ignore both. `:19` — no rules for credential files; add `.env*` plus a `!.env.example` negation as cheap insurance even on a zero-build site. `:16` — lockfile handling is asymmetric (`yarn.lock`/`pnpm-lock.yaml` not ignored, `pnpm-debug.log*` missing); pick one policy. `.github/workflows/pages.yml:32` — `|| true` suppresses *all* `cp` failures, not just "LICENSE absent", and LICENSE does exist, so the fallback only ever triggers on the failures you would want surfaced; use an explicit existence check. `:31` — add an `::error::` guard so a rename of `chromacalm.html` fails legibly instead of with a raw `cp: cannot stat`. `:13` — consider `cancel-in-progress: true` if pushes to main are frequent.
<!-- SECTION:DESCRIPTION:END -->
