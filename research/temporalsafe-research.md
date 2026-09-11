# TemporalSafe — Per-Project Research Note

> Status: build-order gate **passed** (2026-09-10). TemporalSafe is #2 in the build sequence and has a full planning package at `planning/temporalsafe/IMPLEMENTATION_PLAN.md`. Planning only — no code until the user approves the build.
> Parent: `research/browser-vision-app-ideas.md` (shortlist #2, ⭐) · `research/browser-vision-app-ideas-design.md` (spec §2, decisions D3–D4, revisions R4–R6).

## Problem

Photosensitive, migraine, and vestibular users are hurt by CSS animations, blinking cursors, marquees, auto-playing video, and GIFs. Existing tools cover single sites (YouTube-only CogniShield); nothing reduces page-level temporal risk generally.

## Mechanism (grounded)

- Bookmarklet/userscript observes the page via `MutationObserver` + computed styles and classifies risky motion:
  - CSS animations/transitions below a duration floor with high iteration counts;
  - `<marquee>` / `<blink>` and JS-driven style toggling (sampled);
  - `<video autoplay>`.
- Reducer strategies are per-element and reversible: `animation-play-state: paused`, dim cover, pause/mute video — always with a "Paused" badge, per-element "Show" undo, and a per-site allowlist. **Never** silently deletes content or removes layout.
- Two profiles: `Reduced` (default, honors `prefers-reduced-motion`) and `Photosensitive` (opt-in, also freezes fast blinking text/cursors and pauses autoplay).
- Sampling is **motion-triggered and throttled**: 500ms debounce, sleeps when idle.

Grounding anchors: WCAG 2.3.1 (three flashes / general flash) and `prefers-reduced-motion` are the standards references; the tool reduces *page content* motion only.

## Landscape grounding

- [CogniShield](https://socket.dev/chrome/package/kmengpakcdmpcliojljfepdokneehmdn) — YouTube-specific flicker guard (gap: page-level, not per-site).
- Reading rulers / focus tools (Ability, FocusFlow) — spatial, not temporal.
- Global dimmers — irrelevant to motion.

Gap: **general page-level temporal safety** with a gentle default and per-element undo.

## Honesty constraints (locked)

- Scope statement in the panel: **"does not affect cross-origin iframes; canvas/rAF-loop animation is best-effort."**
- Reduces *page* motion, not display PWM — browsers cannot measure backlight flicker.
- Comfort aid, not a medical device / not a diagnosis (visible note, AGENTS.md §7).

## Decisions & revisions folded

| # | Decision | Outcome |
| --- | --- | --- |
| D3 | Form: audit-only / extension | Live userscript (acts directly) |
| D4 | Policy: single toggle | Two profiles (Reduced default / Photosensitive opt-in) |
| R4 | False positives | Reduced default; per-element undo; per-site allowlist |
| R5 | Scope honesty | Cross-origin iframes + canvas/rAF out of scope, stated in UI |
| R6 | Resource use | Motion-triggered sampling, 500ms debounce, idle sleep |
| C2 | Determinism | Injected-clock unit tests for rule classification |

## Open questions

- None blocking. Real-site smoke list (one SPA + one news site) to be picked at implementation time.
