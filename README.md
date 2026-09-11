# Vision Apps

**Six small tools for readers whose eyes hurt from screens — low vision, photophobia, astigmatism.**

An umbrella repository for Paul Kirby's accessibility toolkit — six small, dependency-light projects that reduce visual strain for low-vision, photophobic, and astigmatic readers.

*Updated: 2026-09-10*

**Start here:** open [ChromaCalm](https://markkirby125.github.io/chromacalm/) in your browser — no install, no account.

## Projects

| Project | Description | Repo | Live |
| --- | --- | --- | --- |
| **ChromaCalm** | Zero-install spectral notch filtering web tool for photophobia, migraine, and screen halation. | [markkirby125/chromacalm](https://github.com/markkirby125/chromacalm) | [chromacalm](https://markkirby125.github.io/chromacalm/) |
| **SoftContrast** | Anti-halation reading palette generator using APCA and OKLCH. | [markkirby125/softcontrast](https://github.com/markkirby125/softcontrast) | [softcontrast](https://markkirby125.github.io/softcontrast/) |
| **terminal-a11y** | Terminal accessibility enhancement layer (screen-reader, photophobia, sensory-budget, braille, and audio-progress modes). | [markkirby125/terminal-a11y](https://github.com/markkirby125/terminal-a11y) | — |
| **FocusBeacon** | Ultra-lightweight vanilla JS focus-ring accessibility engine with a high-contrast dual-contour focus ring and cursor radar. | [markkirby125/focusbeacon](https://github.com/markkirby125/focusbeacon) | [focusbeacon](https://markkirby125.github.io/focusbeacon/) |
| **GlareMap** | Spatial glare heatmap + targeted softening for photophobia (lab page + bookmarklet). | [markkirby125/glaremap](https://github.com/markkirby125/glaremap) | [glaremap](https://markkirby125.github.io/glaremap/) |
| **TemporalSafe** | Page-level flicker/flash reducer for photosensitive and migraine users (userscript + demo). | [markkirby125/temporalsafe](https://github.com/markkirby125/temporalsafe) | [temporalsafe](https://markkirby125.github.io/temporalsafe/) |

The five web tools are served from GitHub Pages. `terminal-a11y` is a CLI package with no
hosted demo; install it from its repository.

## In planning

One more browser tool is through buildable-spec design and peer review, and has a full
planning package (phases + acceptance criteria + model/effort assignments).

| Project | Description | Status | Plan |
| --- | --- | --- | --- |
| **Keratoscope** | Astigmatism ghosting calibration lab (userstyle export). | Planning only. | `planning/keratoscope/IMPLEMENTATION_PLAN.md` |

Next in build order: Keratoscope (GlareMap and TemporalSafe are live). Task register: `docs/TASKS.md`.

## Layout

Each project lives in its own repository (linked above). This repo holds the shared governance and research:

- `docs/` — task register (`TASKS.md`) and reference specs.
- `research/` — per-project research notes.
- `AGENTS.md` — agent governance.
- `ACCESSIBILITY_PROJECTS_EXPANDED.md` and `ACCESSIBILITY_PROJECT_IDEAS.md` — project background and ideas.
