# Vision Apps

An umbrella repository for Mark Kirby's accessibility toolkit — four small, dependency-light projects that reduce visual strain for low-vision, photophobic, and astigmatic readers.

## Projects

| Project | Description | Repo |
| --- | --- | --- |
| **ChromaCalm** | Zero-install spectral notch filtering web tool for photophobia, migraine, and screen halation. | [markkirby125/chromacalm](https://github.com/markkirby125/chromacalm) |
| **SoftContrast** | Anti-halation reading palette generator using APCA and OKLCH. | [markkirby125/softcontrast](https://github.com/markkirby125/softcontrast) |
| **terminal-a11y** | Terminal accessibility enhancement layer (screen-reader, photophobia, sensory-budget, braille, and audio-progress modes). | [markkirby125/terminal-a11y](https://github.com/markkirby125/terminal-a11y) |
| **FocusBeacon** | Ultra-lightweight vanilla JS focus-ring accessibility engine with a high-contrast dual-contour focus ring and cursor radar. | [markkirby125/focusbeacon](https://github.com/markkirby125/focusbeacon) |

No GitHub Pages site is published for any of the four projects yet: their `pages.yml`
deploy workflows fail at the "Setup Pages" step, so the `markkirby125.github.io/*` URLs
return 404. Each project runs from a clone of its own repository in the meantime.

## Layout

Each project lives in its own repository (linked above). This repo holds the shared governance and research:

- `docs/` — task register (`TASKS.md`) and reference specs.
- `research/` — per-project research notes.
- `AGENTS.md` — agent governance.
- `ACCESSIBILITY_PROJECTS_EXPANDED.md` and `ACCESSIBILITY_PROJECT_IDEAS.md` — project background and ideas.
