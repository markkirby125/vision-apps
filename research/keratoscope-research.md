# Keratoscope — Per-Project Research Note

> Status: build-order gate **passed** (2026-09-10). Keratoscope is #3 in the build sequence and has a full planning package at `planning/keratoscope/IMPLEMENTATION_PLAN.md`. Planning only — no code until the user approves the build.
> Parent: `research/browser-vision-app-ideas.md` (shortlist #3, ⭐) · `research/browser-vision-app-ideas-design.md` (spec §3, decisions D5, revisions R7–R9).

## Problem

Astigmatism/keratoconus users often see doubled/ghosted text. The only surveyed astigmatism tool ([KeratoVision](https://www.producthunt.com/products/github-288)) targets code editors with fixed settings; no browser tool calibrates a personal compensation profile and exports it.

## Mechanism (grounded)

- Perceptual calibration wizard on a DOM-text test chart (not canvas, so font rendering is realistic): direction → magnitude → ghost opacity → verify on a paragraph → export. Target **< 2 minutes**.
- The ghost step is anchored to real text: "line up the pale copy with the shadow you see on real words."
- `profile.js` stores `{dx, dy, opacity, blur, fontWeightDelta, letterSpacingDelta}` in `localStorage` and emits a **deterministic** CSS string (small deltas).
- Exports: one default — **userstyle**. Bookmarklet injector and JSON profile under an Advanced disclosure.
- v1 targets **light-mode pages only**; dark/`prefers-color-scheme` variants deferred (YAGNI, R9).

Compensation is perceptual presentation tweaking, **not** optical deconvolution and **not** vision correction — the copy promise is "a presentation tweak that may reduce perceived doubling for some users — no guarantee" (R7).

## Landscape grounding

- [KeratoVision](https://www.producthunt.com/products/github-288) — editor theme/extension, fixed settings, no calibration/export loop.
- Global dimmers, magnifiers, reading rulers — none address ghosting compensation.

Gap: **calibration-driven astigmatism ghosting compensation** with an exportable personal profile.

## Honesty constraints (locked)

- Never claim it "corrects" vision or ghosting; adjust presentation to reduce perceived doubling.
- Prominent safety prompt at start: **"new or sudden double vision? see an eye professional today."**
- Comfort aid, not a medical device / not a diagnosis (visible note, AGENTS.md §7).

## Decisions & revisions folded

| # | Decision | Outcome |
| --- | --- | --- |
| D5 | Rendering: SVG displacement / canvas | DOM-text wizard (perceptual honesty, realistic fonts) |
| R7 | Efficacy wording | "may reduce perceived doubling for some users" — no guarantee |
| R8 | Calibration UX | Concrete anchor wording, mandatory preview, <2 min |
| R9 | v1 scope | Single light mode; one default export (userstyle); safety prompt at start |
| C3 | Determinism + local-only | Snapshot-tested CSS output; `localStorage` only; CSS-only export |

## Open questions

- None blocking. Whether a dark-mode variant is worth v2 is deferred until v1 feedback (explicit YAGNI).
