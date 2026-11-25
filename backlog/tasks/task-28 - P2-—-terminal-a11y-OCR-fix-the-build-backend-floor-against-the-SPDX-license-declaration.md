---
id: TASK-28
title: >-
  P2 — terminal-a11y OCR: fix the build-backend floor against the SPDX license
  declaration
status: To Do
assignee: []
created_date: '2026-11-25T11:47:47Z'
labels:
  - terminal-a11y
dependencies: []
ordinal: 28000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — terminal-a11y OCR: fix the build-backend floor against the SPDX license declaration** — pro (low) · `python-packaging`. `pyproject.toml:2` — `license = "MIT"` is the PEP 639 SPDX string form, supported only by setuptools ≥77 (77.0.3+), but the floor is `setuptools>=61.0.0`; under 61–76 (e.g. `pip install --no-build-isolation` or a pinned older environment) metadata validation fails with "project.license must be valid exactly by one definition". Raise the floor to `setuptools>=77.0.0`, or revert to `license = { text = "MIT" }` if 3.9-era tooling must be supported; while bumping, drop the explicit `wheel` requirement (setuptools has vendored `bdist_wheel` since 70.1).
<!-- SECTION:DESCRIPTION:END -->
