---
id: TASK-2
title: P1 — Publish terminal-a11y to PyPI
status: To Do
assignee: []
created_date: '2026-11-25T11:46:40Z'
labels:
  - open-tasks
dependencies: []
ordinal: 2000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — Publish terminal-a11y to PyPI** — pro (low) · `python-packaging`. **Blocked on user credentials** (no `~/.pypirc`, no token env var, `twine` not installed system-wide). Create a PyPI API token, then `twine upload -u __token__ -p <token> dist/*` from a clone. sdist + wheel already build with `twine check` PASSED and the `terminal-a11y` name is free. Verify afterwards with `pip install terminal-a11y` in a scratch venv.
<!-- SECTION:DESCRIPTION:END -->
