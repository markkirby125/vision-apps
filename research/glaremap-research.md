# GlareMap — Per-Project Research Note

> Status: build-order gate **passed** (2026-09-10). GlareMap is #1 in the build sequence and has a full planning package at `planning/glaremap/IMPLEMENTATION_PLAN.md`. Planning only — no code until the user approves the build.
> Parent: `research/browser-vision-app-ideas.md` (shortlist #1, ⭐) · `research/browser-vision-app-ideas-design.md` (spec §1, decisions D1–D2, revisions R1–R3).

## Problem

Global dimmers darken the whole page and flatten contrast everywhere. Photophobic and migraine users need the *brightest regions* softened while readable content stays crisp. No tool in the surveyed landscape does spatial, per-region glare targeting.

## Mechanism (grounded)

- Walk at most ~2,000 visible elements (sampled when the DOM is larger); read computed background/foreground colors.
- Convert sRGB → WCAG relative luminance per element; aggregate into a ~96×54 viewport grid.
- Render the heatmap on native `<canvas>`; soften only cells above a user threshold with an SVG overlay (dark translucent blurred patches).
- The mask is `pointer-events: none` and **skips interactive elements** (links, buttons, inputs, form fields).
- Distribution: shared core powers both a lab page (URL iframe best-effort + paste-HTML) and a bookmarklet.

The math is browser-native and deterministic: sRGB channel decode plus WCAG `L = 0.2126R + 0.7152G + 0.0722B`. "Glare" is an **estimate from CSS colors, not a photometric measurement** — the UI must label it as such (AGENTS.md §7).

## Landscape grounding

Crowded (avoid duplicating): global dimmers/warm filters — [Dimly](https://chromewebstore.google.com/detail/dimly-%E2%80%94-screen-dimmer-for/elkdfophogmfbiffkgjpomjajihklnmk), [Telsia](https://addons.mozilla.org/en-US/firefox/addon/telsia-eye-strain-reduction/), [Smart Brightness](https://socket.dev/chrome/package/ccagdjdbalpeoegjnnenjfocjgkiihbe), [Brightness Lowerer Plus](https://addons.mozilla.org/en-US/firefox/addon/brightness-lowerer-plus/); all-in-one toolkits (Assistive24, Helperbird, TD Accessibility Adapter, AccessiFlow, Incluser, open-nagish); dark modes preserving images ([Notte](https://addons.mozilla.org/hu/firefox/addon/notte-dark-mode/)).

Gap: **spatial** glare targeting — per-region softening instead of global or per-site dimming.

## Honesty constraints (locked)

- Label: "relative brightness estimate, not a glare measurement."
- Image/video/gradient regions are marked **"unmeasured"** and never scored as glare.
- Comfort aid, not a medical device / not a diagnosis (visible note, AGENTS.md §7).

## Decisions & revisions folded

| # | Decision | Outcome |
| --- | --- | --- |
| D1 | Form: bookmarklet-only / lab-only / hybrid | Hybrid shared core + lab page + bookmarklet |
| D2 | Heatmap rendering: SVG / DOM patches | Native `<canvas>` heatmap + SVG softening mask |
| R1 | Overlay must never intercept input | `pointer-events: none`; skip interactive elements |
| R2 | Accuracy claims | "relative brightness estimate"; unmeasured regions disclosed |
| R3 | Performance budget | ≤2,000 elements, ≤96×54 grid, `requestIdleCallback`, no network calls |
| S5 | Strict CSP blocks injected styles | Detect and show "this site blocks injected styles" — never fail silently |

## Open questions

- None blocking. Fixture-page smoke test list (3 real sites incl. one strict-CSP site) to be picked at implementation time.
