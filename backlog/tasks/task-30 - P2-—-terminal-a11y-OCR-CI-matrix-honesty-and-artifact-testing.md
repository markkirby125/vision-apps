---
id: TASK-30
title: 'P2 — terminal-a11y OCR: CI matrix honesty and artifact testing'
status: To Do
assignee: []
created_date: '2026-11-25T11:47:50Z'
labels:
  - terminal-a11y
dependencies: []
ordinal: 30000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — terminal-a11y OCR: CI matrix honesty and artifact testing** — flash (low) · `surgical-patch`. `.github/workflows/test.yml` — no `permissions` (`:9`), no `timeout-minutes` (`:10`, a pytest run can block on stdin for the 6-hour default), no `concurrency` (`:7`), no pip caching (`:9`), and `fail-fast` defaults to true (`:12`) so a break on 3.9 cancels the 3.12 leg and hides which versions are broken — wrong for a compatibility matrix. `:26` — the previous step builds an sdist and wheel into `dist/` but this step runs `pip install .`, which rebuilds from the source tree, so the built artifacts are never installed or exercised and packaging defects would pass CI while the published artifact is broken; install from the built wheel (and smoke-test the sdist). `pyproject.toml:36` — the classifiers advertise 3.9–3.13 while the matrix runs only 3.9 and 3.12; widen the matrix or trim the classifiers, and note 3.9 reached EOL in October 2025. `pyproject.toml:52` — no test extra despite a shipped pytest suite (CI installs pytest ad hoc). `pyproject.toml:55` — inline type hints but no `terminal_a11y/py.typed`, so wheels expose no types downstream (PEP 561).
<!-- SECTION:DESCRIPTION:END -->
