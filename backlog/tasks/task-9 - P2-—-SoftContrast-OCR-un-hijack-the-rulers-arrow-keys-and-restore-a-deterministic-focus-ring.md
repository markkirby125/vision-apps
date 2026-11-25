---
id: TASK-9
title: >-
  P2 — SoftContrast OCR: un-hijack the ruler's arrow keys and restore a
  deterministic focus ring
status: To Do
assignee: []
created_date: '2026-11-25T11:47:00Z'
labels:
  - softcontrast
dependencies: []
ordinal: 9000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P2 — SoftContrast OCR: un-hijack the ruler's arrow keys and restore a deterministic focus ring** — pro (medium) · `better-accessibility`. `js/features.js:82` — while the ruler is enabled every `ArrowUp`/`ArrowDown` on the page is `preventDefault()`-ed. `isEditable()` only covers form controls and `contentEditable`, so arrows used by ARIA widgets (listbox, slider, radiogroup, custom menus) and arrow combinations with Alt/Ctrl/Meta are swallowed, conflicting with the WCAG 2.1 AA goal. Also, `parseFloat(ruler.style.top) || 0` puts the ruler at the viewport top instead of the setter's `window.innerHeight / 2` default when `style.top` is unset. `css/style.css:273` — the global `:focus-visible` outline uses `currentColor`, so its contrast against `#f0f0f0` is whatever the element's text colour happens to be; in a palette generator any future light control colour makes the focus ring invisible. Use a verified dedicated focus colour or a double ring (outline + `box-shadow`). Also drop `width` from the progress-fill transition lists (`css/style.css:140` — pseudo-element width driven by the `value` attribute does not animate outside Firefox) and add `animation-iteration-count` / `scroll-behavior` to the reduced-motion override (`css/style.css:292`). Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
