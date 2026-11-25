---
id: TASK-16
title: >-
  P1 — TemporalSafe OCR: the demo page must not trigger the symptoms it claims
  to reduce
status: To Do
assignee: []
created_date: '2026-11-25T11:47:25Z'
labels:
  - temporalsafe
dependencies: []
ordinal: 16000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
**P1 — TemporalSafe OCR: the demo page must not trigger the symptoms it claims to reduce** — pro (low) · `better-accessibility`. `index.html:59` — the page auto-plays a looping video and runs a scrolling marquee by default, but `prefers-reduced-motion` only disables `.blink-demo`. Photosensitive, migraine and vestibular visitors get flashing and motion content before — and unless — they click Run, which is the exact harm the tool exists to prevent. Drop `autoplay` (keep `controls` + a `poster`), or call `video.play()` only when `matchMedia('(prefers-reduced-motion: reduce)')` is false, and extend the reduced-motion rule to the marquee fixture; gating the whole fixture section behind an explicit opt-in button is safer. `index.html:60` — the only video fixture streams from `interactive-examples.mdn.mozilla.net`, which MDN has retired and which is commonly blocked, so the core demo silently breaks when it fails to load; it also leaks the visitor's IP to a third party, which is off-brand for a privacy/comfort tool. Bundle a tiny local clip. `index.html:78` — if the static `import` fails (404, `file://` CORS, syntax error) the module script aborts and both buttons silently do nothing with no live-region feedback; import dynamically inside the handler and report failures through `#status`. Support: `code-review-and-quality`.
<!-- SECTION:DESCRIPTION:END -->
