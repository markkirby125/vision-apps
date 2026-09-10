## Project Identity & Tech Stack

- **Project Name**: Vision Apps — umbrella repository for Mark Kirby's accessibility toolkit.
- **Niche**: accessibility utilities for low vision, photophobia, migraines, and sensory sensitivities.
- **Tech Stack**: Vanilla JavaScript, HTML, SVG, and CSS for the three web tools (ChromaCalm, SoftContrast, FocusBeacon); Python for `terminal-a11y`; static GitHub Pages deployment.
- **No heavy third-party UI libraries** (e.g. Radix, Shadcn, Material UI) unless already present in the dependency manifest.
- **Language & Conventions**: Strict JavaScript/TypeScript for the web tools, strictly-typed Python for `terminal-a11y`; standard CSS/SVG; empathetic and clear documentation tone.

## Directory Layout & Module Invariants

- **Module Invariants**: the three web tools are 100% client-side with no backend dependencies; `terminal-a11y` is a server-side Python CLI package. No heavy UI frameworks; the web tools favour lightweight single-file or few-module architectures.

- **Repositories** (each project lives in its own repo; this repo holds the shared docs):
  ```text
  chromacalm      github.com/markkirby125/chromacalm      FL-41 & 520nm spectral filter web tool
  softcontrast    github.com/markkirby125/softcontrast    anti-halation reading palette generator
  terminal-a11y   github.com/markkirby125/terminal-a11y   CLI accessibility wrappers and patches
  focusbeacon     github.com/markkirby125/focusbeacon     dual-ring keyboard focus & cursor radar library

  docs/           task register + reference specs (this repo)
  research/       per-project research notes (this repo)
  ```

## Platform-Specific Invariants

- **Static HTML export; must respect prefers-reduced-motion; screen-reader compatibility requires stripping ASCII/ANSI**

## Whitelisted Network Endpoints & CORS

- **None (must remain entirely client-side and offline capable)**
