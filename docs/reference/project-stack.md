## Project Identity & Tech Stack

- **Project Name**: Vision Apps — `Local Workspace (GitHub Pages)`.
- **Niche**: accessibility utilities for low vision, photophobia, migraines, and sensory sensitivities.
- **Tech Stack**: Vanilla JavaScript, HTML, SVG, and CSS (zero-install web tools and browser extensions); static GitHub Pages deployment.
- **No heavy third-party UI libraries** (e.g. Radix, Shadcn, Material UI) unless already present in the dependency manifest.
- **Language & Conventions**: Strict JavaScript/TypeScript; standard CSS/SVG; empathetic and clinical documentation tone.

## Directory Layout & Module Invariants

- **Module Invariants**: 100% client-side execution (no backend dependencies); no heavy UI frameworks; strict adherence to lightweight single-file architectures.

- **Directory Tree**: 
  ```text
  chromacalm/ - Clinical FL-41 & 520nm filter web tool
  softcontrast/ - Anti-halation reading palette generator
  focusbeacon/ - Dual-ring keyboard focus & cursor radar library
  terminal-a11y/ - CLI accessibility wrappers and patches
  docs/ - Documentation and task registers
  ```

## Platform-Specific Invariants

- **Static HTML export; must respect prefers-reduced-motion; screen-reader compatibility requires stripping ASCII/ANSI**

## Whitelisted Network Endpoints & CORS

- **None (must remain entirely client-side and offline capable)**
