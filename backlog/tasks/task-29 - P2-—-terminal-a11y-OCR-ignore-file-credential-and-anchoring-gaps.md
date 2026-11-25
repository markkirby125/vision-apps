---
id: TASK-29
title: 'P2 — terminal-a11y OCR: ignore-file credential and anchoring gaps'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:49Z'
labels:
  - terminal-a11y
dependencies: []
ordinal: 29000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — terminal-a11y OCR: ignore-file credential and anchoring gaps** — flash (low) · `surgical-patch`. `.gitignore:30` — only the exact filename `.env` is ignored, so `.env.local`, `.env.production` and `.env.*.local` are all committable — a live risk for a tool that wraps terminal and subprocess output. Broaden to `.env*` with a `!.env.example` negation, and note that gitignore only affects future tracking: run `git ls-files -i -c --exclude-standard`, `git rm --cached` any offenders, and rotate anything that reached history. `.gitignore:17` — unanchored `lib/`, `lib64/`, `parts/`, `var/`, `env/`, `venv/`, `ENV/`, `env.bak/`, `venv.bak/`, `downloads/` match at *any* depth, so a legitimate `terminal_a11y/lib/` or `tests/env/` fixture would be silently skipped by `git add`; anchor them with a leading `/`. `.gitignore:38` — missing `.DS_Store`, `Thumbs.db`, `.mypy_cache/`, `.tox/`, `.coverage`, `.coverage.*`, `htmlcov/` — all artifacts this project will produce given it is type-hint-audited and has a pytest suite.
<!-- SECTION:DESCRIPTION:END -->
