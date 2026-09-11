# TemporalSafe — Implementation Plan

> Planning-only artifact (build-order gate passed 2026-09-10). No code until the user approves the build. Spec: `research/browser-vision-app-ideas-design.md` §2; revisions R4–R6 + C2 are mandatory and folded in below.

## Overview

TemporalSafe detects and freezes page-level flicker/flash (blinking text, marquees, fast CSS animations, autoplaying video) for photosensitive/migraine/vestibular users. Bookmarklet/userscript-first; every action is per-element, reversible, and announced. No backend, no heavy libraries.

## Scope Definition

### In Scope

- Rule classifier: CSS animations/transitions below a duration floor with high iteration counts; `<marquee>`/`<blink>`; sampled JS-driven blinking; `<video autoplay>`.
- Reducer: pause/dim/mute per element with "Paused" badge, per-element "Show" undo, per-site allowlist.
- Two profiles: `Reduced` (default) and `Photosensitive` (opt-in).
- Motion-triggered, throttled sampling: 500ms debounce, idle sleep.
- Keyboard-accessible floating panel; polite live-region announcements; 10-second first-run explainer.
- Per-site memory in `localStorage` keyed by hostname.

### Out of Scope

- Cross-origin iframes (labeled only — untouchable).
- Reliably stopping canvas/rAF-loop animation (best-effort: dim cover + note; never breaks the app).
- Display PWM / OS-level flicker (browsers cannot measure backlight flicker).
- Extension packaging (userscript first; extension later).
- Hiding navigation or removing layout (never).

## Architecture Decisions

- **D3 — Live reducer:** acts directly on the page rather than auditing.
- **D4 — Profiles:** `Reduced` honors `prefers-reduced-motion` by default; `Photosensitive` is opt-in for stricter needs.
- **R4–R6:** false positives mitigated by default-gentle policy + undo + allowlist; honest scope statement; throttled sampling.
- **C2 — Injected clock:** rule classification is deterministic under test.

## Model & Effort (per phase)

| # | Phase | Model | Effort |
| --- | --- | --- | --- |
| 1 | Repo scaffold + gate wiring | flash | low |
| 2 | Rule classifier core (injected clock) | pro | high |
| 3 | Reducer strategies + undo + allowlist | pro | high |
| 4 | Profiles + throttled observer | pro | medium |
| 5 | Floating panel + live-region announcements | pro | medium |
| 6 | Tool a11y + copy audit | pro | low |
| 7 | Userscript build + deploy + docs | flash | medium |

## Repository Layout & File Map (planned)

```
temporalsafe/
├── temporalsafe.js           # Source: scanner (classifier) + reducer + profiles + panel
├── build.js                  # Zero-dep node script: bundles temporalsafe.js into temporalsafe.user.js + .min.js
├── index.html                # Demo/fixture page + install instructions (userscript + bookmarklet)
├── tests/
│   ├── classifier.test.mjs   # Injected-clock rule classification (deterministic timing)
│   ├── reducer.test.mjs      # Pause/dim/mute strategies, badge, undo, allowlist
│   └── fixtures/
│       └── motion-fixture.html# CSS blink, marquee, autoplay video, blinking caret
├── README.md                 # Install, scope statement, honesty notes, live URL
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
  1. Create repo `markkirby125/temporalsafe` (public, MIT) on build approval.
  2. `package.json` with `"type": "module"`, `"test": "node --test tests/*.test.mjs"`, `"build": "node build.js"`.
  3. `.github/workflows/test.yml` (run `npm test` and `npm run build`) and `pages.yml` (deploy static files).
  4. Placeholder README with "in planning" wording + medical-device disclaimer.
- **Acceptance criteria:**
  - [c1] `npm run test` green with a placeholder test; `npm run build` emits `temporalsafe.user.js`.
  - [c2] Repo deploys statically (no backend).
- **Verification:** `npm run test && npm run build` in the new repo; CI green after push.

### Phase 2: Rule classifier core (injected clock)

- **Goal:** Deterministic classification of risky motion from DOM + computed styles.
- **Files touched:** `temporalsafe.js`, `tests/classifier.test.mjs`.
- **Steps:**
  1. Implement classifier rules: animation/transition duration below floor with high iteration count; `<marquee>`/`<blink>`; sampled style toggling; `<video autoplay>`.
  2. Accept an injected clock/scheduler so timing is deterministic in tests.
  3. Score elements into a risk model (none/low/high) used by profile policy.
- **Acceptance criteria:**
  - [c1] Fixture elements (CSS blink, marquee, autoplay video, blinking caret) classify with expected scores under an injected clock.
  - [c2] Blinking text caret is excluded from the default `Reduced` profile but flagged by `Photosensitive`.
  - [c3] Classifier makes zero writes to the DOM (pure function).
- **Verification:** `node --test tests/classifier.test.mjs`; fixture page run.

### Phase 3: Reducer strategies + undo + allowlist

- **Goal:** Apply per-element, reversible reductions with visible state.
- **Files touched:** `temporalsafe.js`, `tests/reducer.test.mjs`.
- **Steps:**
  1. Strategies: `animation-play-state: paused`; dim cover; pause/mute `<video autoplay>`; badge each affected element with a "Paused" marker.
  2. Per-element "Show" undo restores the exact prior state; per-site allowlist ("always allow on this site") persisted in `localStorage` by hostname.
  3. Never hide navigation, never remove layout, never delete content.
- **Acceptance criteria:**
  - [c1] Undo restores the original inline styles/playback state exactly.
  - [c2] Allowlisted hostname skips future reductions; stored per-site.
  - [c3] Badges are non-flashing and have text ("Paused"), not color-only.
- **Verification:** `node --test tests/reducer.test.mjs`; fixture page with undo + allowlist flows.

### Phase 4: Profiles + throttled observer

- **Goal:** Profile policy and resource-bounded observation.
- **Files touched:** `temporalsafe.js`.
- **Steps:**
  1. `Reduced` profile (default): honors `prefers-reduced-motion`; skips caret; per-element undo enabled.
  2. `Photosensitive` profile (opt-in): additionally freezes fast blinking text/cursors and pauses autoplay.
  3. `MutationObserver` is motion-triggered: 500ms debounce, sleeps when idle, never runs continuous rAF.
  4. Newly added nodes are re-classified; cross-origin iframes are labeled only (scope statement).
- **Acceptance criteria:**
  - [c1] Default profile applies gentle reductions; Photosensitive applies the stricter set after explicit opt-in.
  - [c2] No continuous sampling loop — observer sleeps when idle (verifiable via injected scheduler).
  - [c3] Scope statement rendered in the panel: "does not affect cross-origin iframes; canvas/rAF-loop animation is best-effort."
- **Verification:** `node --test` with injected scheduler; manual SPA run showing idle sleep.

### Phase 5: Floating panel + announcements

- **Goal:** Accessible control panel with honest copy and live-region announcements.
- **Files touched:** `temporalsafe.js`.
- **Steps:**
  1. Panel: on/off, profile switch, per-site allowlist toggle, "Paused N flashing elements" summary; keyboard-accessible (`tabindex="0"`, Escape closes, focus returns; non-modal).
  2. Polite `role="status"` live region announces "Paused 3 flashing elements" updates; the tool's own UI contains no flashing elements.
  3. 10-second first-run explainer; reduced-motion-safe panel (no animation).
- **Acceptance criteria:**
  - [c1] Keyboard-only walk completes every flow (open, switch profile, undo, allowlist, close).
  - [c2] Live region is polite and announces state changes; no flashing in the tool's own UI.
  - [c3] First-run explainer appears once and is dismissible.
- **Verification:** manual keyboard + screen-reader walk; reduced-motion toggle.

### Phase 6: Tool a11y + copy audit

- **Goal:** Pass `better-accessibility` and AGENTS.md §7 for the tool itself.
- **Files touched:** `temporalsafe.js`, `index.html`, `README.md`.
- **Steps:**
  1. Audit native controls, labels, `:focus-visible`, 44×44px targets, 200% zoom + 320px reflow.
  2. Copy audit: "page content only, not display PWM"; "comfort aid, not a medical device"; no guarantee wording.
  3. Demo page (`index.html`) includes the fixture elements and install instructions.
- **Acceptance criteria:**
  - [c1] All controls named/role/state announced; keyboard flows complete.
  - [c2] 200% zoom + 320px reflow without horizontal scrolling.
  - [c3] Copy passes `no-ai-slop`; scope statement present.
- **Verification:** manual keyboard + screen-reader walk; `no-ai-slop` review; zoom/reflow check.

### Phase 7: Userscript build + deploy + docs

- **Goal:** Ship `temporalsafe.user.js` and the Pages demo.
- **Files touched:** `build.js`, `index.html`, `README.md`, `llms.txt`.
- **Steps:**
  1. `build.js` emits `temporalsafe.user.js` (Tampermonkey `@run-at document-start`) and a minified variant.
  2. Enable GitHub Pages (workflow) on `markkirby125/temporalsafe`; demo page links the userscript + bookmarklet.
  3. Finalize README/llms.txt with the scope statement and live URL.
- **Acceptance criteria:**
  - [c1] `npm run build` output is a valid userscript with `@run-at document-start`.
  - [c2] Live URL returns 200 with the demo page.
  - [c3] README makes no unverifiable claims; scope statement visible.
- **Verification:** `npm run build`; `curl -sI` on Pages URL; README claim audit.

## Checkpoints

- **After Phase 3:** classifier + reducer tested headless; undo/allowlist verified on the fixture page.
- **After Phase 5:** full flow works on a real SPA and a news site; idle sleep observed.
- **After Phase 7:** userscript + Pages live; docs honest; build approval closed.

## Test List

1. `classifier.test.mjs` — injected-clock classification of CSS blink, marquee, autoplay, caret; score thresholds.
2. `reducer.test.mjs` — pause/dim/mute; exact undo; allowlist persistence; no-layout-removal assertion.
3. `profiles.test.mjs` (fold into classifier) — Reduced vs Photosensitive policy boundaries.
4. Manual — SPA + news site smoke; keyboard walk; screen-reader announcements; reduced-motion; 200% zoom/320px reflow.

## AGENTS.md §8 gate wiring (planned)

- **Repo:** `markkirby125/temporalsafe` (to be created on build approval).
- **Test gate:** `npm run test` → `node --test tests/*.test.mjs`.
- **Build:** `node build.js` → `temporalsafe.user.js` + `.min.js`.
- **Deploy:** `git push origin main`; Pages via `.github/workflows/pages.yml`.
- **No `lint` script** (toolkit convention).

## Risks & Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| False positives freeze legitimate UI | High (trust) | `Reduced` default, per-element undo, allowlist (R4) |
| Cross-origin iframes untouchable | Med | Scope statement in panel (R5); labeled only |
| Sampling burns CPU on busy SPAs | Med | Motion-triggered, 500ms debounce, idle sleep (R6) |
| Canvas/rAF animation unstoppable | Med | Best-effort dim cover + note; never break the app |
| Claims safety it can't deliver | High (trust) | "page content only"; no PWM/medical claims |

## Open Questions

- None blocking. Whether to publish `temporalsafe` to npm is deferred until the userscript is validated.
