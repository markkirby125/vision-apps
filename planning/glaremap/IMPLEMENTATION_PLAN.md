# GlareMap — Implementation Plan

> Planning-only artifact (build-order gate passed 2026-09-10). No code until the user approves the build. Spec: `research/browser-vision-app-ideas-design.md` §1; revisions R1–R3 + S5 are mandatory and folded in below.

## Overview

GlareMap estimates per-region luminance from computed CSS colors, renders a heatmap, and softens only the hottest regions with an SVG mask. End-user comfort aid for photophobia/migraine. All client-side; no backend, no accounts, no heavy libraries.

## Scope Definition

### In Scope

- Shared scanner/renderer core powering both a lab page and a bookmarklet.
- Luminance grid (≤2,000 sampled elements, ~96×54 cells) via `requestIdleCallback`; zero network calls.
- Canvas heatmap + SVG softening mask (`pointer-events: none`, skips interactive elements).
- Lab page: URL input (iframe best-effort with paste-HTML fallback), threshold + strength sliders, export-bookmarklet button.
- Bookmarklet: floating keyboard-accessible panel, instant off switch, reduced-motion support, CSP-blocked fallback message.
- Plain-language labels and visible "comfort aid, not a medical device" note.

### Out of Scope

- Photometric measurement of real glare (heuristic only, labeled).
- Shadow DOM traversal (v1 skips shadow roots).
- Scoring image/video/gradient regions (marked "unmeasured").
- Browser extension packaging (bookmarklet first; extension later).
- Backend crawling / cloud accounts.

## Architecture Decisions

- **D1 — Hybrid form:** shared core + lab page + bookmarklet; one codebase, testable and universal.
- **D2 — Rendering:** native `<canvas>` for the heatmap grid; SVG for the softening mask (fast, styleable, no deps).
- **R1–R3, S5:** mask never intercepts input; honest "estimate" wording; hard perf budget; explicit CSP fallback.

## Model & Effort (per phase)

| # | Phase | Model | Effort |
| --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low |
| 2 | `scanner.js` luminance core | pro | high |
| 3 | `renderer.js` heatmap + softening mask | pro | medium |
| 4 | Lab page (URL/paste, sliders, export) | flash | medium |
| 5 | Bookmarklet + floating panel | pro | medium |
| 6 | Tool a11y + copy audit | pro | low |
| 7 | Deploy + README + llms.txt | flash | low |

## Repository Layout & File Map (planned)

```
glaremap/
├── index.html                # Lab page (controls, preview, export)
├── scanner.js                # DOM walk → sRGB decode → WCAG luminance → grid (pure, no DOM writes)
├── renderer.js               # Canvas heatmap + SVG softening mask
├── panel.js                  # Floating panel UI (off switch, sliders, badges) — bookmarklet runtime
├── bookmarklet.js            # Bootstrap: injects scanner/renderer/panel, observes DOM changes
├── build.js                  # Zero-dep node script: bundles scanner+renderer+panel into bookmarklet.min.js
├── tests/
│   ├── scanner.test.mjs      # Luminance math, thresholding, element cap, interactive-element skip
│   ├── renderer.test.mjs     # Mask generation, unmeasured-region legend
│   └── fixtures/
│       └── glare-fixture.html# Known-color fixture page for smoke tests
├── README.md                 # Install, usage, honesty notes, live URL
├── llms.txt                  # LLM-facing manifest
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
  1. Create repo `markkirby125/glaremap` (public, MIT) on build approval.
  2. Add `package.json` with `"type": "module"` and `"test": "node --test tests/*.test.mjs"`.
  3. Add `.github/workflows/test.yml` (run `npm test`) and `pages.yml` (deploy static files).
  4. Add placeholder README with the "not yet published / in planning" wording and the medical-device disclaimer.
- **Acceptance criteria:**
  - [c1] `npm run test` runs green with a trivial placeholder test (gate wired before features).
  - [c2] Repo has no build step beyond `node build.js` (buildless static deploy).
- **Verification:** `npm run test` in the new repo; `git push origin main` triggers `test.yml` green.

### Phase 2: `scanner.js` — luminance core

- **Goal:** Pure, dependency-free scanning module that walks the DOM and emits a luminance grid.
- **Files touched:** `scanner.js`, `tests/scanner.test.mjs`.
- **Steps:**
  1. Implement sRGB → linear → WCAG relative luminance (`0.2126/0.7152/0.0722`), with alpha composited against white and black bounds.
  2. Walk visible elements up to the 2,000 cap (deterministic sampling when the DOM is larger); skip non-rendered nodes.
  3. Aggregate element luminance into a ~96×54 viewport grid of `cells[{x,y,w,h,luma}]`; mark image/video/gradient-backed regions `unmeasured`.
  4. Schedule work with `requestIdleCallback` (fallback `setTimeout`); make zero network calls.
- **Acceptance criteria:**
  - [c1] Known sRGB fixtures produce luminance values within 0.001 of hand-computed values.
  - [c2] DOMs > 2,000 elements are sampled and still bounded (no walk past the cap).
  - [c3] `unmeasured` flag is set for image/video/gradient regions, never scored as glare.
- **Verification:** `node --test tests/scanner.test.mjs`; jsdom fixture with known colors and an oversized DOM.

### Phase 3: `renderer.js` — heatmap + softening mask

- **Goal:** Draw the heatmap and apply softening only to cells above the user threshold.
- **Files touched:** `renderer.js`, `tests/renderer.test.mjs`.
- **Steps:**
  1. Draw the grid to `<canvas>` with a perceptual color scale (not color-only: legend + text alternative).
  2. Build the SVG mask as dark translucent blurred patches over threshold-exceeding cells.
  3. Enforce `pointer-events: none` on the mask and skip cells whose bounding box intersects interactive elements (links, buttons, inputs, form fields).
  4. Add a text alternative listing the top hotspot regions.
- **Acceptance criteria:**
  - [c1] Mask patches cover only cells above threshold; interactive-element cells are never covered.
  - [c2] Mask is `pointer-events: none` — a click under a patch reaches the page element.
  - [c3] Heatmap has a non-color redundant cue (text list of top hotspots).
- **Verification:** `node --test tests/renderer.test.mjs`; manual check that a button under a patch still receives clicks.

### Phase 4: Lab page

- **Goal:** Static lab UI with URL/paste input, controls, preview, and bookmarklet export.
- **Files touched:** `index.html`, `build.js`.
- **Steps:**
  1. Build the page shell: one `<h1>`, `<main>`, semantic sections, "Skip to content" link.
  2. URL input → iframe best-effort (blocked by `X-Frame-Options`/CSP shows friendly fallback) plus paste-HTML mode.
  3. Native `<input type="range">` threshold + strength sliders, each with `<label>`; 44×44px touch targets.
  4. `build.js` concatenates `scanner.js` + `renderer.js` + `panel.js` into `bookmarklet.min.js`; export button copies the `javascript:` URI.
- **Acceptance criteria:**
  - [c1] Paste-HTML mode renders a live heatmap without network access.
  - [c2] Iframe failure surfaces a fallback message, never a silent blank.
  - [c3] Every control has a bound label, a visible `:focus-visible` ring, and ≥44×44px hit area.
  - [c4] Exported bookmarklet string is self-contained and minified by `node build.js`.
- **Verification:** `npm run test`; manual paste + iframe flows; keyboard-only walk of the page.

### Phase 5: Bookmarklet + floating panel

- **Goal:** Inject scanner/renderer/panel on any page with safe, honest behavior.
- **Files touched:** `panel.js`, `bookmarklet.js`, `build.js`.
- **Steps:**
  1. Panel: off switch (instant), threshold/strength, "unmeasured" legend, plain-language label "relative brightness estimate, not a glare measurement."
  2. Panel is keyboard-accessible: `tabindex="0"`, Escape closes, focus returns to trigger; no focus trap needed (non-modal).
  3. Honor `prefers-reduced-motion`: static heatmap, no fade animation.
  4. Detect strict CSP injection failure → panel shows "this site blocks injected styles" instead of failing silently.
  5. Debounced `MutationObserver` re-scans dynamic content.
- **Acceptance criteria:**
  - [c1] Off switch fully removes mask and heatmap immediately.
  - [c2] Keyboard: panel opens, all controls operable, Escape closes and restores focus.
  - [c3] Reduced motion → no animation; CSP-blocked → visible fallback text.
  - [c4] Panel never traps focus and never covers the whole viewport.
- **Verification:** manual bookmarklet run on 3 real sites (including one strict-CSP site); keyboard walk; reduced-motion toggle.

### Phase 6: Tool a11y + copy audit

- **Goal:** Make the tool itself pass `better-accessibility` and AGENTS.md §7.
- **Files touched:** `index.html`, `panel.js`, `README.md`.
- **Steps:**
  1. Audit: native controls only, labels bound, `:focus-visible` styles, no color-only status, 44×44px targets, 200% zoom + 320px reflow.
  2. Copy audit: no "corrects/guarantees/calibrated" wording; "comfort aid, not a medical device / not a diagnosis" visible on both lab and panel.
  3. Heatmap legend and top-hotspots list read correctly at 200% zoom.
- **Acceptance criteria:**
  - [c1] Keyboard-only walk completes every flow (open, adjust, off, export).
  - [c2] Screen-reader names: every control announces name/role/state.
  - [c3] 200% zoom and 320px reflow — no horizontal scrolling.
  - [c4] Copy audit passes `no-ai-slop` (no banned tokens, no unverifiable claims).
- **Verification:** manual keyboard + screen-reader walk; `no-ai-slop` copy review; zoom/reflow check.

### Phase 7: Deploy + README + llms.txt

- **Goal:** Ship the static tool to GitHub Pages with honest docs.
- **Files touched:** `README.md`, `llms.txt`, `index.html`.
- **Steps:**
  1. Enable GitHub Pages (workflow type) on `markkirby125/glaremap`.
  2. Finalize README: install/use, honesty notes, live URL, links to sibling tools.
  3. Publish `llms.txt`; verify the live URL serves `<title>GlareMap</title>`.
- **Acceptance criteria:**
  - [c1] Live URL returns 200 with the real tool.
  - [c2] README makes no unverifiable claims; "not a medical device" note present.
  - [c3] `llms.txt` matches README structure.
- **Verification:** `curl -sI` on the Pages URL; README claim audit.

## Checkpoints

- **After Phase 3:** `npm run test` green; pure core works headless; human reviews heatmap behavior on the fixture page before UI work.
- **After Phase 5:** full flow works end-to-end from both lab and bookmarklet on a real site.
- **After Phase 7:** Pages live; README/llms.txt honest; build approval closed.

## Test List

1. `scanner.test.mjs` — sRGB→luminance golden values; alpha bounds; 2,000-element cap; `unmeasured` flags; zero-network assertion.
2. `renderer.test.mjs` — threshold patch coverage; interactive-element skip; `pointer-events` enforcement; top-hotspots text.
3. `build.test.mjs` (optional) — `node build.js` output is self-contained and contains no external URLs.
4. Manual — 3-site bookmarklet smoke (incl. strict CSP); keyboard-only walk; reduced-motion; 200% zoom/320px reflow.

## AGENTS.md §8 gate wiring (planned)

- **Repo:** `markkirby125/glaremap` (to be created on build approval).
- **Test gate:** `npm run test` → `node --test tests/*.test.mjs`.
- **Build:** `node build.js` (bookmarklet bundle only; page itself is buildless).
- **Deploy:** `git push origin main`; Pages via `.github/workflows/pages.yml`.
- **No `lint` script** (toolkit convention).

## Risks & Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| "Glare" heuristic misread as science | High (trust) | "relative brightness estimate" label; unmeasured regions; no photometric claims |
| Heavy pages jank (INP blowout) | Med | ≤2,000 cap, ~96×54 grid, `requestIdleCallback`, debounced observer |
| Mask covers buttons | High (input) | `pointer-events: none` + interactive-element skip (R1) |
| Strict CSP blocks styles | Med | Detect + visible "this site blocks injected styles" (S5) |
| Image/video glare unaddressed | Med | Marked "unmeasured", disclosed in UI (R2) |

## Open Questions

- None blocking. Real-site smoke list to be chosen at implementation (one strict-CSP site required).
