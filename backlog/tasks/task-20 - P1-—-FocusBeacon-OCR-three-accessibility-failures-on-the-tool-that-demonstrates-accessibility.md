---
id: TASK-20
title: >-
  P1 — FocusBeacon OCR: three accessibility failures on the tool that
  demonstrates accessibility
status: To Do
assignee: []
created_date: '2026-11-25T11:47:33Z'
labels:
  - focusbeacon
dependencies: []
ordinal: 20000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — FocusBeacon OCR: three accessibility failures on the tool that demonstrates accessibility** — pro (medium) · `better-accessibility`. `index.html:245` — an anchor carries both `button` and `link` classes: `a.button` sets `color: var(--accent-text)` (#fff) but the later, equal-specificity `a.link { color: var(--accent) }` wins, so the text renders #0b5c9e on the `a.button` background `var(--accent)` (#0b5c9e) — text and background identical, underline accent-on-accent. That is a WCAG 1.4.3/1.4.11 failure on a page advertising AAA. `index.html:209` — blanket `outline: none` on `:focus-visible` makes the overlay the *only* focus indicator: if `focusbeacon.js` errors, is blocked, or fails to track an element (iframes/shadow DOM, targets focused before the first rAF draw) keyboard users get no focus indicator at all (2.4.7/2.4.11); the `border-color` change on inputs does not cover buttons and links. Only suppress the native outline once the overlay is confirmed drawn, and keep a fallback. `index.html:334` — the dialog declares `aria-modal="true"` with no focus trap and no Escape handler, so AT hides the background while Tab still walks into it, keyboard-only users have no standard dismiss path, and the page stays scrollable behind; use `<dialog>` + `showModal()` or add the handler, a trap, and `inert` on the siblings. Also `index.html:275` — the demo text input has only a placeholder, which is not an accessible name (3.3.2/4.1.2); `index.html:23` — `scroll-behavior: smooth` is unconditional, so anchor jumps animate even under `prefers-reduced-motion`, contradicting the advertised feature and risking 2.3.3, so wrap it in `no-preference`; `index.html:288` — the dialog copy says it opens without taking focus, but the open handler immediately calls `closeBtn.focus()`. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
