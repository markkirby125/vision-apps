# Vision Apps

**Four small tools for readers whose eyes hurt from screens — low vision, photophobia, astigmatism.**

An umbrella repository for Paul Kirby's accessibility toolkit — four small, dependency-light projects that reduce visual strain for low-vision, photophobic, and astigmatic readers.

*Updated: 2026-09-10*

**Start here:** open [ChromaCalm](https://markkirby125.github.io/chromacalm/) in your browser — no install, no account.

## Projects

| Project | Description | Repo | Live |
| --- | --- | --- | --- |
| **ChromaCalm** | Zero-install spectral notch filtering web tool for photophobia, migraine, and screen halation. | [markkirby125/chromacalm](https://github.com/markkirby125/chromacalm) | [chromacalm](https://markkirby125.github.io/chromacalm/) |
| **SoftContrast** | Anti-halation reading palette generator using APCA and OKLCH. | [markkirby125/softcontrast](https://github.com/markkirby125/softcontrast) | [softcontrast](https://markkirby125.github.io/softcontrast/) |
| **terminal-a11y** | Terminal accessibility enhancement layer (screen-reader, photophobia, sensory-budget, braille, and audio-progress modes). | [markkirby125/terminal-a11y](https://github.com/markkirby125/terminal-a11y) | — |
| **FocusBeacon** | Ultra-lightweight vanilla JS focus-ring accessibility engine with a high-contrast dual-contour focus ring and cursor radar. | [markkirby125/focusbeacon](https://github.com/markkirby125/focusbeacon) | [focusbeacon](https://markkirby125.github.io/focusbeacon/) |

The three web tools are served from GitHub Pages. `terminal-a11y` is a CLI package with no
hosted demo; install it from its repository.

## In planning / in build

Three more browser tools are through buildable-spec design and peer review, and have full
planning packages (phases + acceptance criteria + model/effort assignments).

| Project | Description | Status | Plan |
| --- | --- | --- | --- |
| **GlareMap** | Spatial glare heatmap + targeted softening (hybrid lab + bookmarklet). | **Build approved 2026-09-11 — in progress** in [`markkirby125/glaremap`](https://github.com/markkirby125/glaremap). | `planning/glaremap/IMPLEMENTATION_PLAN.md` |
| **TemporalSafe** | Page-level flicker/flash reducer (userscript + demo page). | Planning only. | `planning/temporalsafe/IMPLEMENTATION_PLAN.md` |
| **Keratoscope** | Astigmatism ghosting calibration lab (userstyle export). | Planning only. | `planning/keratoscope/IMPLEMENTATION_PLAN.md` |

Build order: GlareMap → TemporalSafe → Keratoscope. Task register: `docs/TASKS.md`.

## Layout

Each project lives in its own repository (linked above). This repo holds the shared governance and research:

- `docs/` — task register (`TASKS.md`) and reference specs.
- `research/` — per-project research notes.
- `AGENTS.md` — agent governance.
- `ACCESSIBILITY_PROJECTS_EXPANDED.md` and `ACCESSIBILITY_PROJECT_IDEAS.md` — project background and ideas.
