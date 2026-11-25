---
id: TASK-34
title: P2 — Re-run the GlareMap OCR scan
status: To Do
assignee: []
created_date: '2026-11-25T11:47:57Z'
labels:
  - cross-project-ocr-follow-ups
dependencies: []
ordinal: 34000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — Re-run the GlareMap OCR scan** — pro (medium) · `code-review-and-quality`. The only project not scanned: all 14 files failed with `429 达到 5 小时的使用上限` (5-hour provider cap, which expired 2026-10-01 10:59:22 — verify with `ocr llm test` before starting). This row stays open until a GlareMap result file exists. **Do not run it concurrently with other OCR scans** — the parallel attempt on 2026-09-30 exhausted the quota twice and also ran `focusbeacon` twice at once, with one session's result file silently overwriting the other's. Run scans one at a time with `--timeout 30` and space them out. Exact command in `HANDOVER.md` §4.
<!-- SECTION:DESCRIPTION:END -->
