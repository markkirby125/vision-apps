# Browser-Based Vision App Ideas — Buildable Specs (Brainstorming Pass)

> Status: Understanding Lock **confirmed by user** (2026-09-10), all six ideas, buildable-spec depth. Peer review (`multi-agent-brainstorming`) **completed**; mandatory revisions R1–R9 **folded into the ⭐ specs below**. No implementation approved — next gate is build-order selection + per-project planning.
> Parent research: `research/browser-vision-app-ideas.md`.

## Shared constraints (all six)

- Single static HTML/JS (or buildless repo with tiny modules), GitHub Pages deployable, no backend, no accounts, no heavy libraries.
- Bookmarklet/userscript-first where useful; extension only later.
- Every tool carries a visible "comfort aid, not a medical device / not a diagnosis" note (AGENTS.md §7).
- The tools themselves must pass `better-accessibility`: native controls, visible focus, keyboard paths, `prefers-reduced-motion`, labels, no color-only cues, 44×44px touch targets.
- Shared modules only when a second tool actually needs them (YAGNI); no speculative abstraction.

---

## 1. GlareMap — spatial glare heatmap + targeted softening

**What:** Estimate per-region luminance from computed CSS colors, render a heatmap, soften only the hottest regions with an SVG mask. End-user tool for photophobia/migraine.

**Approaches considered**
1. Bookmarklet-only — universal but hard to test and fragile on hostile pages. ❌
2. Static lab only (iframe URL) — clean UX but CORS/`X-Frame-Options` blocks most sites. ❌
3. **Hybrid: shared core + lab page + bookmarklet (recommended)** — same scanner/renderer module powers both. ✅

**Architecture (chosen)**
- `scanner.js` — walks at most ~2,000 visible elements (sampled when the DOM is larger), reads computed background/foreground colors, converts sRGB → WCAG relative luminance, aggregates into a ~96×54 viewport grid (`cells[{x,y,w,h,luma}]`). Uses `requestIdleCallback`; never makes network calls.
- `renderer.js` — draws heatmap on native `<canvas>`; applies softening as SVG overlay patches (dark translucent rectangles with blur) only on cells above the user threshold. The mask is `pointer-events: none` and **skips interactive elements** (links, buttons, inputs, form fields).
- Lab page — URL input (iframe best-effort with friendly fallback), paste-HTML mode, threshold + strength sliders, export-bookmarklet button.
- Bookmarklet — injects scanner+renderer+floating panel; panel is keyboard accessible, honors reduced motion, and has an **instant off switch**. Plain-language label in the panel: "relative brightness estimate, not a glare measurement."

**Data flow:** DOM → scanner → luminance grid → threshold → heatmap → user adjust → mask applied.

**Edge cases:** image/video/gradient regions are marked **"unmeasured"** and never scored as glare; `position:fixed` and shadow DOM (v1: skip shadow roots); dynamic content (debounced MutationObserver); alpha colors (compute against white and black bounds); reduced motion (static heatmap, no fade animation); strict CSP blocks injected styles → panel shows "this site blocks injected styles" instead of failing silently.

**A11y of the tool:** text alternative listing top hotspot regions; native range inputs with labels; no color-only indication; panel doesn't trap focus; heatmap has a plain-language explanation and off switch.

**Testing:** `node --test` + jsdom for luminance math, thresholding, element cap and interactive-element skip; fixture-page smoke test; manual bookmarklet test on 3 real sites (including one strict-CSP site).

**Risks:** "Glare" is a heuristic, not photometry (labeled in UI); heavy pages may still jank (budget + idle scheduling + cap); iframe mode limited by CORS; unmeasured image regions may leave real glare unaddressed (disclosed in UI).

---

## 2. TemporalSafe — page-level flicker & flash reducer

**What:** Bookmarklet/userscript that detects and freezes page-level flicker/flash (blinking text, marquees, fast CSS animations, autoplaying video) for photosensitive/migraine/vestibular users.

**Approaches considered**
1. Audit-only report — useful for devs but leaves the user unprotected. ❌
2. **Live reducer (bookmarklet/userscript, recommended)** — acts directly on the page. ✅
3. Extension — later, after userscript validation.

**Architecture (chosen)**
- `scanner.js` — MutationObserver + computed-style rules: animations/transitions below a duration floor with high iteration counts; `<marquee>`/`<blink>`; sampled style toggling for JS-driven blinking; `<video autoplay>`. Sampling is **motion-triggered and throttled**: 500ms debounce, sleeps when idle.
- `reducer.js` — per-element strategies: `animation-play-state: paused`, dim cover, pause/mute video, always with a "Paused" badge and per-element "Show" undo plus a per-site **allowlist** ("always allow on this site"). Never silently deletes content; never hides navigation or removes layout.
- Profiles — `Reduced` (**default**; honors `prefers-reduced-motion`) and `Photosensitive` (**opt-in**; also freezes fast blinking text/cursors and pauses autoplay).
- Scope statement in the panel: **"does not affect cross-origin iframes; canvas/rAF-loop animation is best-effort."**
- Per-site memory — `localStorage` keyed by hostname; 10-second first-run explainer.

**Data flow:** observe → classify risk score → profile policy → apply → persist per-site override/allowlist.

**Edge cases:** JS rAF-driven animation (sample-based detection is best-effort; dim cover + note when detected, never breaks the app); cross-origin iframes (label only, can't touch); canvas/game content (never break interactivity — skip); user dismisses a freeze (per-element undo); newly added nodes (observer); blinking text caret (excluded from default Reduced profile).

**A11y of the tool:** panel keyboard-accessible; polite live-region announcements ("Paused 3 flashing elements"); the tool's own UI contains no flashing elements and no flashing badges.

**Testing:** jsdom unit tests for rule classification with an **injected clock** (deterministic timing); fixture page (CSS blink, marquee, autoplay video, blinking caret); manual browser run on an SPA and a news site.

**Risks:** false positives pausing legitimate content (mitigate with default Reduced, per-element undo, allowlist); cannot measure display PWM — scope says "page content only"; sampling can still cost CPU on busy SPAs (throttling + idle sleep).

---

## 3. Keratoscope — astigmatism ghosting calibration lab

**What:** Calibration wizard where the user matches their perceived ghost/double image, then generates a personal compensating CSS profile (directional edge emphasis, text-shadow cancellation, weight/spacing tweaks) exported as userstyle/bookmarklet. Copy promise: **"a presentation tweak that may reduce perceived doubling for some users — no guarantee."**

**Approaches considered**
1. SVG `feDisplacementMap` pre-distortion — unproven, could worsen ghosting. ❌
2. **DOM-text calibration wizard + CSS export (recommended)** — perceptual, honest, testable. ✅
3. On-page live injector — later, once profile generation is validated.

**Architecture (chosen)**
- Wizard (DOM-based test chart): steps = direction → magnitude → ghost opacity → verify on a paragraph → export. The ghost step is anchored to real text: "line up the pale copy with the shadow you see on real words." Target: **< 2 minutes**. A **prominent safety prompt** appears at start: "new or sudden double vision? see an eye professional today."
- `profile.js` — stores `{dx, dy, opacity, blur, fontWeightDelta, letterSpacingDelta}` in `localStorage`; emits a deterministic CSS string. Keeps deltas small; a paragraph **preview is mandatory** before export.
- Exports — **one default: userstyle**. Bookmarklet injector and JSON profile under an Advanced disclosure.
- Single-file HTML/CSS/JS; DOM text (not canvas) for realistic font rendering; **v1 targets light-mode pages only** (dark/`prefers-color-scheme` variants deferred — YAGNI).

**Data flow:** user adjustments → live preview → profile → generated CSS.

**Edge cases:** dx/dy sign conventions; single-eye calibration only (note binocular limits); mobile touch targets ≥44px; dark pages in v1 show a "light mode only for now" notice rather than a bad profile.

**A11y of the tool:** large test letters; native range inputs with labels; step status announced; no color-only cues; safety prompt readable at 200% zoom.

**Testing:** unit tests for CSS generation (deterministic snapshot); manual calibration-flow timing test (<2 min); copy audit to guarantee no "corrects vision" claims and that the safety prompt is present.

**Risks:** users may expect optical correction — the wizard and export both carry the no-guarantee wording; generated CSS may hurt some fonts (small deltas + mandatory preview); effect may be imperceptible for some users (disclosed, not hidden).

---

## 4. Photopia — photophobia stimulus audit for developers

**What:** Developer tool that scores a page's blue-cyan energy (480–500nm band) and extreme-contrast hotspots, then suggests `<feColorMatrix>`/CSS fixes.

**Approaches considered**
1. Backend crawler — violates no-backend constraint. ❌
2. **Paste-HTML / sandboxed-iframe analyzer (recommended)** — static analysis, no network, no backend. ✅
3. Extension — later.

**Architecture (chosen)**
- `ingest.js` — URL (iframe best-effort) or pasted HTML; renders in a hidden sandboxed iframe (`srcdoc`) to read computed styles; does not execute remote JS.
- `analyzer.js` — per element: relative luminance; blue-cyan energy score (weighted R/G/B contribution approximating the 480–500nm band — documented heuristic based on ChromaCalm's research, not a clinical threshold); WCAG/APCA contrast; halation flag for extreme adjacent contrast.
- `report.js` — severity table + canvas heatmap + suggested fixes (feColorMatrix presets, color swaps, dimming values).

**Data flow:** input → sandbox render → style extraction → scoring → report + fixes.

**Edge cases:** external resources (strip or toggle); `url()` images (skip); dynamic JS (not executed — static only); large DOM (sample/cap); non-sRGB (treat as sRGB, note).

**A11y of the tool:** report passes contrast and keyboard; tables have captions; severity never color-only.

**Testing:** fixture HTML with known colors → expected scores (unit); report render smoke; zero-network test.

**Risks:** scoring misread as science — "heuristic" labeled throughout; iframe sandbox quirks.

---

## 5. AmslerWatch — local Amsler-grid journal

**What:** Zero-install web Amsler grid with calibration instructions, daily entries in `localStorage`, CSV export, and a plain trend view. For people with AMD/central-vision conditions who self-monitor metamorphopsia.

**Approaches considered**
1. **Single-file web app (recommended)** — canvas grid + local journal. ✅
2. PWA/offline service worker — later (YAGNI).
3. Extension — no.

**Architecture (chosen)**
- Grid renderer — SVG/canvas Amsler grid, configurable size, fixation dot, dark-mode inversion, calibration instructions.
- Journal — entries `{date, eye, result: none|distorted|changed, note}` in `localStorage`; trend list; CSV export; optional opt-in reminder via Notification API.
- Text-based fallback questionnaire for users who cannot see the grid (blind/low-vision).

**Data flow:** render grid → user views → record entry → journal update → trend/export.

**Edge cases:** localStorage quota (data is tiny); timezone-safe dates; both-eyes-per-day; clear-all with confirm; grid must have a text alternative and work at 200% zoom.

**A11y of the tool:** text fallback path; instructions readable; touch targets ≥44px; no autoplay.

**Testing:** journal storage/CSV unit tests; grid render smoke; copy audit for no-diagnosis wording.

**Risks:** over-reliance on self-monitoring — visible "record only; see your clinician" prompt; canvas needs text alternative.

---

## 6. ReadingLab — low-vision readability report + one-click fixes

**What:** Paste URL/text → report APCA contrast, font metrics, line length, halation risk, blue-light index → one-click fixes via generated userstyle or reader view. Reuses SoftContrast's APCA/OKLCH science.

**Approaches considered**
1. **Lab + report + userstyle export (recommended)** — testable, no on-page risk. ✅
2. Bookmarklet on-page fixer — later, after metrics are validated.
3. Browser extension — no.

**Architecture (chosen)**
- `ingest.js` — URL (iframe best-effort) or pasted HTML/text.
- `metrics.js` — APCA contrast pairs (reuse SoftContrast math), font-size/line-height/letter-spacing, line-length estimate, halation flag (white-on-black pairs), blue-light index.
- `fixes.js` — generates CSS: min font-size 16px, line-height ≥1.5, spacing, APCA-safe palette (SoftContrast presets), remove fixed widths.
- Report UI — issue table with severity + "Apply fixes" preview + userstyle export.

**Data flow:** ingest → extract styles → score → report → generate CSS → preview.

**Edge cases:** iframe CORS → paste mode; fixed-width layouts; `!important` overrides (userstyle needs specificity + `!important`); zoom interplay.

**A11y of the tool:** report readable at 200% zoom; keyboard; severity never color-only.

**Testing:** metric math unit tests against golden pairs; fixture page; CSS generation snapshot.

**Risks:** overlap with SoftContrast (share code, don't fork); auto-fixes can break layouts (preview before export).

---

## Decision log

| # | Decision | Alternatives | Chosen | Why |
| --- | --- | --- | --- | --- |
| D1 | GlareMap form | Bookmarklet-only / lab-only | Hybrid shared core | Testable + universal, one codebase |
| D2 | GlareMap heatmap | SVG / DOM patches | Native canvas + SVG mask | Canvas is fastest for grids, SVG for the softening mask |
| D3 | TemporalSafe form | Audit-only / extension | Live userscript | Acts directly; audit deferred (YAGNI) |
| D4 | TemporalSafe policy | Single toggle | Two profiles (Reduced / Photosensitive) | Matches `prefers-reduced-motion` while serving stricter need |
| D5 | Keratoscope rendering | SVG displacement / canvas | DOM-text wizard | Perceptual honesty; realistic font rendering |
| D6 | Photopia ingestion | Backend crawler / extension | Sandboxed iframe + paste | No backend; deterministic static analysis |
| D7 | AmslerWatch grid | Extension / PWA | Single-file canvas + text fallback | Zero-install; accessible to non-sighted users |
| D8 | ReadingLab fixes | Auto-apply on live sites | Report + preview + userstyle export | No layout breakage without consent |
| D9 | Code sharing | Monorepo modules | Copy/shared module only when 2nd tool needs it | YAGNI per AGENTS.md §2 |

## Next steps (gates before implementation)

1. ~~`multi-agent-brainstorming` peer review~~ — **done** (verdict REVISE; mandatory revisions folded into the ⭐ specs above).
2. **User selects build order.** For each chosen idea: create the project repo (or a branch under an existing repo pattern), a per-project `research/<name>-research.md` note, a project `IMPLEMENTATION_PLAN.md` with phases + acceptance criteria, and a `docs/TASKS.md` open row (priority, model, effort, skill) per AGENTS.md §1.
3. **Planning artifacts to produce before any code** (still planning-only until the user says build): repo layout, file map, test list, and the AGENTS.md §8 gate wiring.
4. Implementation (only after step 2–3 approval) uses toolkit skills: `javascript-pro` / `frontend-ui-engineering` (web tools), `better-accessibility` (tool a11y), `no-ai-slop` (copy), `surgical-patch` (fixes), with the AGENTS.md §8 test gate in the project repo.

---

# Peer Review — multi-agent-brainstorming (2026-09-10)

Scope: the three ⭐ specs (GlareMap, TemporalSafe, Keratoscope). Reviewers invoked sequentially; Primary Designer responded; Arbiter decided.

## Skeptic / Challenger — "assume it fails in production"

**GlareMap**
- S1. Computed styles ignore images, gradients, and videos — most real-world glare comes from those. The heatmap would mislead users on modern pages.
- S2. A full-DOM computed-style walk on news/SaaS pages can take seconds → INP blowout, violating the toolkit's sub-50ms standard.
- S3. "Glare" is subjective; there is no ground truth to validate the threshold against. Risk of shipping something that looks scientific but is unverifiable (the AGENTS.md §7 class).
- S4. Overlay softening may cover links/buttons and intercept clicks.
- S5. Strict CSP can block injected styles, silently breaking the bookmarklet on some sites.

**TemporalSafe**
- S1. Heuristics will false-positive (blinking caret, auto-advancing carousels are legitimate). Freezing legitimate UI will anger users.
- S2. JS-driven rAF animations can't be reliably stopped without breaking the app; sampling misses the worst offenders (canvas).
- S3. "Static frame" replacement for GIFs/videos can break layout or hide navigation.
- S4. Continuous observation on SPAs burns CPU/battery — the cure causes the sensory overload it prevents.
- S5. Cross-origin iframes (ads/embeds) are untouchable, yet they are a main source of flashing content — the tool may claim safety it cannot deliver.

**Keratoscope**
- S1. Core premise is unproven: on-screen text-shadow compensation may not reduce perceived diplopia at all. Placebo risk.
- S2. Low-vision users may not reliably match a fake "ghost"; wizard results may be noise.
- S3. Compensation CSS can blur text edges and increase strain; small deltas may do nothing.
- S4. Dark/light `prefers-color-scheme` variants double the surface for little v1 value.
- S5. Sudden new doubling is a red-flag symptom; the tool must not delay eye-care.

## Constraint Guardian

- C1 (GlareMap). Enforce a hard budget: max ~2,000 sampled elements, ~96×54 grid, idle scheduling, no network calls (verifiable), pure scanner core with injected DOM.
- C2 (TemporalSafe). Throttle: motion-triggered sampling, 500ms debounce, sleep when idle. No untrusted HTML injection (textContent/CSSOM only). Deterministic, clock-injected unit tests.
- C3 (Keratoscope). Local-only profile storage; CSS-only export (no scripts); deterministic snapshot-tested output; calibration < 2 minutes measured.

## User Advocate

- U1 (GlareMap). A red heatmap scares users; needs plain-language explanation and an instant off switch. Softening a button's background while the button stays bright reads as "broken" — auto-skip interactive elements.
- U2 (TemporalSafe). Default must be the gentler Reduced profile; Photosensitive is opt-in. Per-element "Show" undo and non-flashing badges required. 10-second first-run explainer.
- U3 (Keratoscope). The ghost-adjustment task is abstract — anchor it to real text ("line up the pale copy with the shadow you see on real words"). One default export (userstyle), others under Advanced. Prominent "new or sudden double vision? see an eye professional today" prompt at start.

## Primary Designer responses (accepted → revision)

- GlareMap: accept S1 (mark image/video regions "unmeasured", never claim full-page accuracy), S2 (hard budget), S4 (mask is `pointer-events: none`, skips interactive elements), U1 (off switch + plain language), C1. Accept S5 as a known fallback (detect and show "this site blocks injected styles"), S3 handled by labeling "relative brightness estimate, not a glare measurement".
- TemporalSafe: accept S1 (default Reduced, per-element undo, allowlist), S2 (label "best effort; cannot stop canvas/rAF loops safely"), S3 (never hide navigation; badge + reveal, no layout removal), S4 (C2 throttling), S5 (scope statement "does not affect cross-origin iframes"), U2.
- Keratoscope: accept S1 (copy: "presentation tweak that may reduce perceived doubling for some users — no guarantee"), S2 (concrete anchor + preview paragraph), S3 (small deltas, mandatory preview), S4 (drop `prefers-color-scheme` variants from v1 — YAGNI), S5 (prominent safety prompt), U3, C3.

## Arbiter dispositions

| Idea | Verdict | Mandatory revisions (accepted objections) | Rejected objections |
| --- | --- | --- | --- |
| GlareMap | **REVISE → approvable** | S1, S2, S4, U1, C1 + CSP fallback (S5) | none |
| TemporalSafe | **REVISE → approvable** | S1, S3, S4, S5, U2, C2 | S2 (acknowledged, scoped as best-effort — not fixable in browser) |
| Keratoscope | **REVISE → approvable** | S1, S2, S3, S4, S5, U3, C3 | none |

Final disposition: **REVISE** for all three ⭐ specs, with the mandatory revisions listed above incorporated into any implementation plan. No idea is REJECTED; none is APPROVED as-is.

## Review decision log

| # | Decision | Objections considered | Resolution |
| --- | --- | --- | --- |
| R1 | GlareMap overlay must never intercept input | S4 | mask `pointer-events: none`; skip interactive elements |
| R2 | GlareMap accuracy claims | S1, S3 | "relative brightness estimate"; image/video regions marked unmeasured |
| R3 | GlareMap perf budget | S2, C1 | ≤2,000 elements, ≤96×54 grid, idle scheduling |
| R4 | TemporalSafe default profile | S1, U2 | Reduced by default; Photosensitive opt-in; per-element undo |
| R5 | TemporalSafe scope honesty | S5, S2 | cross-origin iframes and rAF/canvas explicitly out of scope |
| R6 | TemporalSafe resource use | S4, C2 | motion-triggered sampling, 500ms debounce, idle sleep |
| R7 | Keratoscope efficacy wording | S1 | "may reduce perceived doubling for some users" — no guarantee |
| R8 | Keratoscope calibration UX | S2, U3 | concrete anchor wording, mandatory preview, <2 min |
| R9 | Keratoscope v1 scope | S4, C3 | single light mode; one default export (userstyle); safety prompt at start |

Revisions R1–R9 are now folded into the ⭐ specs above (2026-09-10). Implementation may begin only after the build-order and per-project planning gates in "Next steps" are completed and the user approves the build.

