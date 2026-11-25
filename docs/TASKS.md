# TASKS.md — Vision Apps task register

Source of truth for task priority, model/effort, and skill (AGENTS.md §1).
Row shape: `- [ ] **P1 — Short name** — pro (high) · `skill-name`. One-line summary.`
Model: `flash` = fetch / patch / implement · `pro` = judgement / architecture / legal.
Effort scale (per task/phase): `low` = deterministic edit / registry / simple lookup · `medium` = model-generated code, copy, or multi-file edit · `high` = standard implementation / multi-step logic · `max` = complex architecture / deep reasoning / cross-cutting invariants.
Legacy mapping: flash `off` ≈ low · flash `on` ≈ medium · pro `low`/`high`/`max` unchanged.
Priorities: P0 = breaks build / launch / money flow · P1 = breaks core promise · P2 = correctness / trust / perf risk · P3 = polish / enhancement.

---

## Pending Tasks

<!--
USER STAGING AREA:
Manually input raw task notes, ideas, feature requests, or bug reports below.
Whenever an AI agent reads this file, it will automatically:
1. Parse and rewrite each item into the standard 5-field Open Task format.
2. Move it down to ## Open Tasks (in priority order).
3. Clear it from ## Pending Tasks to keep the staging area clean.
-->

---

## Open Tasks

Open tasks live in Backlog.md (`backlog/tasks/`, 35 tasks, imported 2026-11-25 from this file). Use `npx backlog.md task list --plain` to list them, `npx backlog.md task <id> --plain` to read one, and `npx backlog.md task create` or `task edit` to change them.

Finished work: docs/TASKS-ARCHIVE.md (includes the former Closed Tasks section). Long evidence: docs/TASKS-EVIDENCE.md. The pre-migration copy of this file is in git history and in `~/backups/tasks-backlog-2026-11-25/vision-apps/`.

### OCR remediation rows — `ocr scan` sweep 2026-09-30 → 2026-10-01

Source: `HANDOVER.md` §OCR review sweep. Full findings JSON per project: `/tmp/opencode/ocr/ocr-scan-<project>.json` (ephemeral — re-run the scan if gone). Session IDs for `ocr scan --resume` are listed in `HANDOVER.md`. All rows assume the same remediation order: fix one project end-to-end, then gate (`npm test` / `pytest -q`) before pushing (AGENTS.md §8).



> Environment note (not a task): `Desktop/Hosted-Services` is a symlink into the Google Drive/Insync tree, so the four clones sat inside a synced folder for ~2 h before being moved out to `~/dev/clones`. Drive trash/version history may still hold snapshots of those `.git` trees; purge from the Drive UI if that sync noise matters.

---

## Phased Implementation Plans (model & effort)

Per-phase model (`flash`/`pro`) and effort (`low`/`medium`/`high`/`max`) for the three build-approved projects. Phase-level source of truth (steps, acceptance criteria, verification, gate wiring): `planning/<name>/IMPLEMENTATION_PLAN.md`. All UI/copy phases also carry support skills `better-accessibility` + `no-ai-slop`.

### GlareMap — `planning/glaremap/IMPLEMENTATION_PLAN.md`

| # | Phase | Model | Effort | Skill |
| --- | --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low | `surgical-patch` |
| 2 | `scanner.js` luminance core | pro | high | `javascript-pro` |
| 3 | `renderer.js` heatmap + softening mask | pro | medium | `javascript-pro` |
| 4 | Lab page (URL/paste, sliders, export) | flash | medium | `frontend-ui-engineering` |
| 5 | Bookmarklet + floating panel | pro | medium | `javascript-pro` |
| 6 | Tool a11y + copy audit | pro | low | `better-accessibility` |
| 7 | Deploy + README + llms.txt | flash | low | `no-ai-slop` |

### TemporalSafe — `planning/temporalsafe/IMPLEMENTATION_PLAN.md`

| # | Phase | Model | Effort | Skill |
| --- | --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low | `surgical-patch` |
| 2 | Rule classifier core (injected clock) | pro | high | `javascript-pro` |
| 3 | Reducer strategies + undo + allowlist | pro | high | `javascript-pro` |
| 4 | Profiles + throttled observer | pro | medium | `javascript-pro` |
| 5 | Floating panel + live-region announcements | pro | medium | `javascript-pro` |
| 6 | Tool a11y + copy audit | pro | low | `better-accessibility` |
| 7 | Userscript build + deploy + docs | flash | medium | `javascript-pro` |

### Keratoscope — `planning/keratoscope/IMPLEMENTATION_PLAN.md`

| # | Phase | Model | Effort | Skill |
| --- | --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low | `surgical-patch` |
| 2 | `profile.js` deterministic CSS generation | pro | medium | `javascript-pro` |
| 3 | Calibration wizard (<2 min, anchored) | pro | high | `frontend-ui-engineering` |
| 4 | Preview + exports (userstyle default) | pro | medium | `javascript-pro` |
| 5 | Tool a11y + copy audit | pro | low | `better-accessibility` |
| 6 | Deploy + README + llms.txt | flash | low | `no-ai-slop` |

> Queued (not implementation tasks yet): Photopia, AmslerWatch, ReadingLab — deferred until they pass their own `multi-agent-brainstorming` peer review, per `research/browser-vision-app-ideas-design.md` Next steps.

---
