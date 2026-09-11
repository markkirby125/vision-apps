# Keratoscope — Implementation Plan

> Planning-only artifact (build-order gate passed 2026-09-10). No code until the user approves the build. Spec: `research/browser-vision-app-ideas-design.md` §3; revisions R7–R9 + C3 are mandatory and folded in below.

## Overview

Keratoscope is a calibration wizard where the user matches their perceived ghost/double image on real text, then generates a personal compensating CSS profile (directional edge emphasis, text-shadow cancellation, weight/spacing tweaks) exported as a userstyle. Copy promise: **"a presentation tweak that may reduce perceived doubling for some users — no guarantee."**

## Scope Definition

### In Scope

- DOM-based calibration wizard: direction → magnitude → ghost opacity → verify on a paragraph → export, target < 2 minutes.
- `profile.js`: `{dx, dy, opacity, blur, fontWeightDelta, letterSpacingDelta}` in `localStorage`; deterministic CSS generation; small deltas.
- Mandatory paragraph preview before export.
- One default export: **userstyle**; bookmarklet injector + JSON profile under an Advanced disclosure.
- Prominent safety prompt at start: "new or sudden double vision? see an eye professional today."
- v1 light-mode pages only; dark/`prefers-color-scheme` notice instead of a bad profile.

### Out of Scope

- Optical correction / deconvolution (explicitly not claimed).
- Dark-mode and `prefers-color-scheme` profile variants (v1 YAGNI — R9).
- SVG `feDisplacementMap` pre-distortion (rejected — unproven, could worsen ghosting).
- On-page live injector (later, after profile generation is validated).
- Cloud accounts / sync.

## Architecture Decisions

- **D5 — DOM-text wizard:** realistic font rendering; perceptual honesty over canvas tricks.
- **R7 — No-guarantee wording** on wizard, preview, and export.
- **R8 — Anchored calibration:** "line up the pale copy with the shadow you see on real words"; mandatory preview; <2 min.
- **R9 — v1 scope:** single light mode, one default export.
- **C3 — Deterministic, local-only:** snapshot-tested CSS; `localStorage` only; CSS-only export (no scripts).

## Model & Effort (per phase)

| # | Phase | Model | Effort |
| --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low |
| 2 | `profile.js` deterministic CSS generation | pro | medium |
| 3 | Calibration wizard (<2 min, anchored) | pro | high |
| 4 | Preview + exports (userstyle default) | pro | medium |
| 5 | Tool a11y + copy audit | pro | low |
| 6 | Deploy + README + llms.txt | flash | low |

## Repository Layout & File Map (planned)

```
keratoscope/
├── index.html                # Single-file wizard UI + preview + export (light mode)
├── profile.js                # Profile model + deterministic CSS generation (unit-testable, loaded by index.html)
├── tests/
│   └── profile.test.mjs      # CSS generation snapshots; delta clamps; sign conventions
├── README.md                 # Install, safety prompt, honesty notes, live URL
├── llms.txt
├── LICENSE                   # MIT
├── .gitignore
└── .github/workflows/
    ├── test.yml              # node --test gate
    └── pages.yml             # GitHub Pages deploy
```

## Implementation Phases

### Phase 1: Repo scaffold + gate wiring

- **Goal:** Stand up the repo skeleton and the AGENTS.md §8 gate before any feature code.
- **Files touched:** `package.json`, `.gitignore`, `LICENSE`, `.github/workflows/test.yml`, `.github/workflows/pages.yml`, `README.md` (skeleton), `llms.txt`.
- **Steps:**
  1. Create repo `markkirby125/keratoscope` (public, MIT) on build approval.
  2. `package.json` with `"type": "module"`, `"test": "node --test tests/*.test.mjs"`.
  3. `.github/workflows/test.yml` and `pages.yml` (buildless static deploy).
  4. Placeholder README with "in planning" wording + safety prompt + medical-device disclaimer.
- **Acceptance criteria:**
  - [c1] `npm run test` green with a placeholder test.
  - [c2] Repo deploys statically; no build step.
- **Verification:** `npm run test` in the new repo; CI green after push.

### Phase 2: `profile.js` — deterministic CSS generation

- **Goal:** Pure profile model that turns stored deltas into a deterministic CSS string.
- **Files touched:** `profile.js`, `tests/profile.test.mjs`.
- **Steps:**
  1. Define profile schema `{dx, dy, opacity, blur, fontWeightDelta, letterSpacingDelta}` with validation and small-delta clamps.
  2. Emit deterministic CSS: directional `text-shadow` cancellation, `font-weight`/`letter-spacing` tweaks, edge emphasis.
  3. Store/load profile in `localStorage` with a versioned key; never emit scripts (CSS only).
- **Acceptance criteria:**
  - [c1] Snapshot tests: identical inputs produce byte-identical CSS.
  - [c2] Out-of-range deltas are clamped; invalid profile falls back to a safe no-op.
  - [c3] Export contains CSS only — no `<script>`, no `url()`.
- **Verification:** `node --test tests/profile.test.mjs`; manual inspect of generated CSS.

### Phase 3: Calibration wizard

- **Goal:** Anchor the ghost adjustment to real text and keep it under 2 minutes.
- **Files touched:** `index.html`.
- **Steps:**
  1. Build steps: direction → magnitude → ghost opacity → verify on a paragraph → export; live preview updates on every change.
  2. Direction/magnitude anchored to real text with the exact wording: "line up the pale copy with the shadow you see on real words."
  3. Safety prompt appears at start and is not dismissible without a "Got it" confirmation; large test letters readable at 200% zoom.
  4. Time the flow; target < 2 minutes from start to export.
- **Acceptance criteria:**
  - [c1] Every step has native controls with bound labels and a live preview.
  - [c2] Safety prompt is visible at start and requires confirmation.
  - [c3] Median calibration completes in < 2 minutes (manual timing).
- **Verification:** manual calibration-flow timing test; keyboard-only walk; 200% zoom.

### Phase 4: Preview + exports

- **Goal:** Mandatory preview and the single default userstyle export.
- **Files touched:** `index.html`, `profile.js`.
- **Steps:**
  1. Paragraph preview is mandatory: export is disabled until preview has been shown at least once.
  2. Default export = userstyle (`@-moz-document`/Stylus-compatible header + generated CSS); bookmarklet injector and JSON profile under an Advanced `<details>`.
  3. Dark-mode pages show a "light mode only for now" notice rather than a bad profile.
- **Acceptance criteria:**
  - [c1] Export blocked until preview shown; default is userstyle.
  - [c2] Advanced disclosure contains bookmarklet + JSON only (no hidden primary path).
  - [c3] Dark-mode detection shows the v1 notice, not an incorrect profile.
- **Verification:** `node --test` for export gating logic (where extracted); manual export + Stylus import check.

### Phase 5: Tool a11y + copy audit

- **Goal:** Pass `better-accessibility` and AGENTS.md §7, with no-guarantee wording everywhere.
- **Files touched:** `index.html`, `README.md`.
- **Steps:**
  1. Audit: native range inputs with labels; `:focus-visible`; step status announced via polite `role="status"`; 44×44px targets; no color-only cues; 200% zoom + 320px reflow.
  2. Copy audit: "may reduce perceived doubling for some users — no guarantee" on wizard, preview, and export; no "corrects vision" wording; "comfort aid, not a medical device" note.
  3. Safety prompt readable at 200% zoom.
- **Acceptance criteria:**
  - [c1] Keyboard-only walk completes the entire calibration and export.
  - [c2] Every step announces progress; controls announce name/role/state.
  - [c3] Copy passes `no-ai-slop`; no guarantee/medical claims anywhere.
  - [c4] 200% zoom + 320px reflow without horizontal scrolling.
- **Verification:** manual keyboard + screen-reader walk; `no-ai-slop` review; zoom/reflow check.

### Phase 6: Deploy + README + llms.txt

- **Goal:** Ship to GitHub Pages with honest docs.
- **Files touched:** `README.md`, `llms.txt`, `index.html`.
- **Steps:**
  1. Enable GitHub Pages (workflow) on `markkirby125/keratoscope`.
  2. Finalize README: safety prompt, honesty notes, calibration instructions, live URL.
  3. Publish `llms.txt`; verify live URL serves `<title>Keratoscope</title>`.
- **Acceptance criteria:**
  - [c1] Live URL returns 200 with the real tool.
  - [c2] README/llms.txt contain the no-guarantee wording and safety prompt.
- **Verification:** `curl -sI` on Pages URL; README claim audit.

## Checkpoints

- **After Phase 2:** `npm run test` green; CSS output deterministic and script-free.
- **After Phase 4:** full calibration → preview → userstyle export flow works; <2 min target met.
- **After Phase 6:** Pages live; docs honest; build approval closed.

## Test List

1. `profile.test.mjs` — deterministic CSS snapshots; delta clamps; sign conventions; CSS-only assertion.
2. Manual — calibration-flow timing (<2 min); keyboard-only walk; screen-reader step announcements; 200% zoom/320px reflow.
3. Copy audit — no-guarantee wording present at wizard, preview, export; safety prompt present.

## AGENTS.md §8 gate wiring (planned)

- **Repo:** `markkirby125/keratoscope` (to be created on build approval).
- **Test gate:** `npm run test` → `node --test tests/*.test.mjs`.
- **Build:** none (buildless static page).
- **Deploy:** `git push origin main`; Pages via `.github/workflows/pages.yml`.
- **No `lint` script** (toolkit convention).

## Risks & Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Users expect optical correction | High (trust) | No-guarantee wording on wizard, preview, export (R7); safety prompt |
| Generated CSS hurts some fonts | Med | Small deltas + mandatory preview (R8) |
| Effect imperceptible for some users | Med | Disclosed, not hidden |
| Dark-mode pages | Low (v1) | "light mode only for now" notice (R9) |
| Sudden new doubling red-flag | High (safety) | Prominent "see an eye professional today" prompt (R9) |

## Open Questions

- None blocking. Dark-mode variant and on-page live injector are deferred to v2 pending v1 feedback.
