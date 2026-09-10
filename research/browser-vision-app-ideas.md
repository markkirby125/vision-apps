# Browser-Based Vision Accessibility App Ideas — Research Notes

> Scope: new, 100% browser-based apps for the Vision Apps toolkit (beyond ChromaCalm, SoftContrast, FocusBeacon, terminal-a11y). All ideas must be zero-install or drop-in, lightweight, and free of heavy libraries (per AGENTS.md §2). Research date: 2026-09-10.

## Understanding summary (brainstorming lock, condensed)

- **What:** A shortlist of new browser-based vision accessibility app ideas with problem, mechanism, feasibility, and differentiation.
- **Why:** Extend the toolkit with new, non-duplicative tools for low-vision, photophobia, astigmatism, migraine, and related visual stress.
- **Who for:** End users (low-vision readers, photophobic/migraine users) and, for one idea, developers auditing their own sites.
- **Constraints:** 100% browser; no backend; no heavy libraries; GitHub Pages friendly; must not make medical/clinical claims (AGENTS.md §7).
- **Non-goals:** No extension-store-only distribution (web tool + bookmarklet/userscript first), no clinical diagnosis tools, no AI backends.

## Assumptions

1. "Browser-based" means a static web tool, bookmarklet, or userscript — not a Chrome Web Store extension (extension can come later).
2. Differentiation matters more than raw novelty: dimmers, dark modes, magnifiers, reading rulers, and font controls are already crowded.
3. Ideas are pre-design research; none are validated or approved for implementation.

## Decision log

| Decision | Alternatives | Chosen | Why |
| --- | --- | --- | --- |
| Skill stack | `apify-market-research` MCP, web-only, brainstorming | `brainstorming` + `web_search` + `better-accessibility` | User-approved; web_search gives live grounding without MCP dependency |
| Idea selection lens | Broad a11y, AI tools | Vision-strain niche (photophobia, halation, flicker, astigmatism) | Matches toolkit's existing research and audience |
| Medical-claim policy | Describe benefits | Frame as comfort aids, never treatment | AGENTS.md §7 anti-hallucination |
| Distribution | Extension-first | Web tool + bookmarklet/userscript first | Matches existing ChromaCalm/SoftContrast pattern |

## Competitor landscape (grounded, Sept 2026)

Crowded categories — avoid building another of these:

- **Global dimmers / warm filters:** [Dimly](https://chromewebstore.google.com/detail/dimly-%E2%80%94-screen-dimmer-for/elkdfophogmfbiffkgjpomjajihklnmk), [Telsia](https://addons.mozilla.org/en-US/firefox/addon/telsia-eye-strain-reduction/), [Smart Brightness](https://socket.dev/chrome/package/ccagdjdbalpeoegjnnenjfocjgkiihbe), [Brightness Lowerer Plus](https://addons.mozilla.org/en-US/firefox/addon/brightness-lowerer-plus/).
- **All-in-one accessibility toolkits:** [Assistive24](https://chromewebstore.google.com/detail/assistive24-assistive-tec/dnddealkeofnkhecelfakjjokmfjibih), [Helperbird](https://www.helperbird.com/blog/best-free-tools-for-low-vision-browsing/), [TD Accessibility Adapter](https://stories.td.com/us/en/article/td-accessibility-adapter), [AccessiFlow](https://socket.dev/chrome/package/ddfcfcideeildccknhhbnbkklniaaepo), [Incluser](https://addons.mozilla.org/vi/firefox/addon/incluser/), [open-nagish](https://www.npmjs.com/package/open-nagish).
- **Dark modes that preserve images:** [Notte](https://addons.mozilla.org/hu/firefox/addon/notte-dark-mode/) (open source, low-vision focused).
- **Magnifiers:** [a11y-magnifier](https://www.npmjs.com/package/a11y-magnifier) (~11 KB widget).
- **Reading rulers / focus lines / bionic reading:** Ability extension, [FocusFlow](https://addons.mozilla.org/fy-NL/firefox/addon/focusflow-accessibilitytoolkit/).
- **Low-vision & color-blindness simulators for developers:** [lowvision.support](https://awesome.ecosyste.ms/projects/github.com%2Fericwbailey%2Flowvision.support), [A11yCanvas](https://devpost.com/software/a11ycanvas).
- **Astigmatism-specific:** [KeratoVision](https://www.producthunt.com/products/github-288) (editor theme + extension: contrast reduction, ghosting correction, edge enhancement).
- **YouTube-specific flicker guard:** [CogniShield](https://socket.dev/chrome/package/kmengpakcdmpcliojljfepdokneehmdn).
- **WCAG auto-patching:** [Open Tweak](https://platform.fossunited.org/hack/fosshack26/p/ca48qdo47n) (proposed).

Gap the toolkit can target: **spatial** (per-region glare, not global dimming), **temporal** (page-level flicker/flash reduction, not just one video site), **astigmatism ghosting calibration**, and **photophobia-oriented site auditing** for developers.

## Shortlist

### 1. GlareMap — spatial glare heatmap + targeted softening ⭐ recommended

- **Problem:** Global dimmers darken the whole page, flattening contrast everywhere (the exact failure ChromaCalm's research describes). Photophobic users need the *brightest regions* softened while readable content stays crisp.
- **Mechanism:** Parse computed background/foreground colors, estimate relative luminance per region, build a heatmap, then overlay an SVG mask that softens only hotspots. Runs client-side from a bookmarklet or a paste-a-URL lab page.
- **Feasibility:** High. CSS color parsing + WCAG relative-luminance math + SVG overlay are all browser-native; no library needed.
- **Differentiation:** Nobody in the landscape above does *spatial* glare targeting; all existing tools dim globally or per-site.
- **Honesty note:** "Glare" is estimated from CSS colors, not a photometric measurement — must be labeled as a heuristic.

### 2. TemporalSafe — page-level flicker & flash reducer ⭐ recommended

- **Problem:** Photosensitive and migraine users are hurt by CSS animations, blinking cursors, marquees, auto-playing video, and GIFs. Existing tools (CogniShield) cover YouTube only.
- **Mechanism:** A bookmarklet/userscript scans the page via MutationObserver + computed styles, detects rapid blinking/flashing elements, and replaces them with static frames or pauses them. Honors `prefers-reduced-motion` and offers a stricter "photosensitive" profile.
- **Feasibility:** Medium-high. Detection heuristics (animation durations, blink timing, frame-rate estimation) are the hard part; pausing is easy.
- **Differentiation:** General page-level temporal safety, not per-site.
- **Honesty note:** It reduces *page* motion, not display PWM (browsers cannot measure display backlight flicker) — scope must say so.

### 3. Keratoscope — astigmatism ghosting calibration lab ⭐ recommended

- **Problem:** Astigmatism/keratoconus users see doubled/ghosted text. KeratoVision targets editors; a calibration-driven web tool that exports a personal compensation profile is still open.
- **Mechanism:** Calibration wizard (user adjusts ghost offset/opacity/direction on a test chart), then generates compensating CSS (directional edge enhancement, text-shadow cancellation, font-weight/contrast tweaks) exported as a userstyle or bookmarklet.
- **Feasibility:** Medium. Compensation is perceptual, not optical deconvolution; an iterative adjustment UI is the product.
- **Differentiation:** Calibration + exportable profile, versus KeratoVision's fixed settings.
- **Honesty note:** Never claim it "corrects" vision or ghosting; it adjusts presentation to reduce perceived doubling.

### 4. Photopia — photophobia stimulus audit for developers

- **Problem:** Developers can't see their site the way a photophobic user does, and existing simulators cover disorders (blur, color blindness) but not wavelength-driven ipRGC stimulation.
- **Mechanism:** A web lab that loads a URL (via iframe where CORS allows, otherwise pasted HTML), scores blue-cyan energy (480–500nm) and extreme-contrast hotspots using a documented heuristic weighting, then outputs a report + suggested `<feColorMatrix>`/CSS fixes.
- **Feasibility:** Medium. Color-space math is straightforward; the scoring model must be labeled heuristic and validated later.
- **Differentiation:** First developer audit tool focused on photophobia rather than WCAG contrast.
- **Honesty note:** Heuristic only; not a medical or photometric instrument.

### 5. AmslerWatch — home visual-grid journal (AMD/metamorphopsia)

- **Problem:** People with macular conditions are told to monitor an Amsler grid daily, but paper grids and general apps don't log results locally or show change over time.
- **Mechanism:** A zero-install web Amsler grid with calibration instructions, daily entry stored in `localStorage`, CSV export, and a plain trend view. No accounts, no cloud.
- **Feasibility:** High. Canvas/SVG grid + localStorage is trivial.
- **Differentiation:** Privacy-first local logging; zero-install; no diagnosis.
- **Honesty note:** Must state it does not diagnose or treat; it only records what the user reports seeing. (Check medical-device guidance before any stronger positioning.)

### 6. ReadingLab — low-vision readability report + one-click fixes

- **Problem:** Low-vision readers hit pages with tiny text, bad line-height, low APCA contrast, and halation-prone color pairs. Audits exist for WCAG; an end-user-facing "fix my page now" report is rarer.
- **Mechanism:** Paste a URL or text; report APCA contrast, font metrics, line length, halation risk, and blue-light index; apply one-click fixes via a generated userstyle or reader view.
- **Feasibility:** Medium (overlaps SoftContrast's palette science — could reuse its APCA/OKLCH logic).
- **Differentiation:** End-user focused; pairs naturally with SoftContrast exports.
- **Honesty note:** Same heuristic caveats as #4.

## Skills used

| Skill | Role |
| --- | --- |
| `brainstorming` | Structured idea-to-shortlist process, understanding lock, decision log |
| `better-accessibility` | Domain guardrails (WCAG, motion safety, native-first, no false claims) |
| `web_search` | Live grounding of the 2026 competitor landscape |

## Next steps

1. User picks 1–2 ideas to develop.
2. For the chosen idea(s), run a full `brainstorming` design pass (architecture, edge cases, testing) before implementation.
3. Add chosen ideas to `docs/TASKS.md` as open rows and create per-project `research/` notes.
