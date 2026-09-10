# Accessibility Tech Projects: Low Vision, Photophobia & Sensory Utilities

A collection of lightweight, high-utility open-source software concepts designed for users with low vision, light sensitivity (photophobia), migraines, and visual fatigue.

---

## 1. Core Science & Problem Space

### The "High-Contrast" Myth
Most developers assume vision accessibility begins and ends with high-contrast dark mode (`#000000` background with `#FFFFFF` text). In practice:
* **Halation Effect:** In users with astigmatism, cataracts, or corneal irregularities, pure white text on pitch black bleeds and glows, causing blurred double-vision and severe eye strain.
* **Retinal Ganglion Cell Stimulation (ipRGCs):** Photophobia is not caused by "brightness" alone; it is triggered by specific wavelengths (480nm–500nm blue-cyan spikes in LED backlights).
* **Narrow-Band Green Light (Harvard Research):** Research led by Dr. Rami Burstein at Harvard Medical School proved that **narrow-band green light (~520nm)** is the only visual spectrum wavelength that does *not* exacerbate migraine pain or photophobia.
* **FL-41 Rose Tint:** The clinical standard prescribed by neuro-ophthalmologists for blepharospasm, concussion recovery, and severe light sensitivity.

---

## 2. Project Proposals

### Project 1: ChromaCalm — Clinical FL-41 & 520nm Green-Band Filter
* **Target Audience:** Migraine sufferers, post-concussion syndrome, blepharospasm, extreme photophobia.
* **Form Factor:** Zero-install browser bookmarklet + single-file static web tool (GitHub Pages).
* **Technical Architecture:**
  * Uses an SVG `<feColorMatrix>` filter overlay instead of a naive CSS opacity div (which flattens contrast and makes text muddy).
  * Spectral clamping presets:
    1. **Harvard Green (520nm):** Calming, narrow-band tint for active migraine episodes.
    2. **Clinical FL-41:** Rose-copper spectral notch filter blocking 480nm–500nm LED spikes.
    3. **Matte Paper / E-Ink:** Low-stimulus monochromatic gray with peak luminance clamped to 120 cd/m².
* **Quick Win:** 100% client-side HTML/SVG/JS in a single file. Instant one-click bookmarklet.

---

### Project 2: SoftContrast — The Anti-Halation Reading Palette Generator
* **Target Audience:** Low vision, high myopia, astigmatism, age-related macular degeneration.
* **Form Factor:** Micro web application + Stylus / Tampermonkey userstyles.
* **Technical Architecture:**
  * Calculates contrast using the modern **APCA (Accessible Perceptual Contrast Algorithm)** instead of outdated WCAG 2.1 formulas.
  * Curated, halation-proof presets:
    * *Midnight Ochre:* Warm deep charcoal (`#141416`) with muted oatmeal text (`#d6d0c4`) — zero glow.
    * *Solar Flare Amber:* Warm amber phosphor on slate background for high-contrast, low-glare night reading.
  * Exports clean CSS custom properties (`--color-bg`, `--color-text`), bookmarklets, or reader-mode style overrides.

---

### Project 3: FocusBeacon — Dual-Ring Keyboard Focus & Cursor Radar
* **Target Audience:** Tunnel vision (glaucoma, retinitis pigmentosa), motor/keyboard-only navigators, severe low vision.
* **Form Factor:** 2KB vanilla JavaScript drop-in library or lightweight browser extension.
* **Technical Architecture:**
  * **Dual-Contour Focus Outline:** 3px thick alternating ring (1.5px white inside, 1.5px black outside) that mathematically guarantees high visibility against any background color or background image.
  * **Cursor Radar Hotkey:** Double-tapping `Ctrl` triggers an expanding, high-contrast ripple animation around the mouse cursor to instantly locate it on 4K/high-DPI monitors.
  * **Motion Safety:** Respects `prefers-reduced-motion` with static, high-contrast halos.

---

### Project 4: Terminal Accessibility Patch (Existing Repo Integration)
* **Target Audience:** Visually impaired and light-sensitive software engineers and sysadmins.
* **Form Factor:** Enhancement to `residential-network-diagnostics` (or standalone CLI wrapper).
* **Technical Architecture:**
  * `--screen-reader` (`--sr`): Strips out spinning progress indicators, ASCII charts, and ANSI escape codes so tools like NVDA, Orca, and VoiceOver read clean sequential output.
  * `--photophobia` (`--soft`): Swaps blinding neon terminal colors for muted amber and soft cream.

---

## 3. Distribution & Subreddit Reactivation (`r/AccessibilityTech`)

### Launch Strategy
1. **Host on GitHub Pages:** Zero hosting cost, high trust, open source.
2. **Authentic Problem-First Angles:**
   * *"I got tired of screen dimmers making text muddy, so I built a zero-install bookmarklet using Harvard's 520nm green-band research to kill screen glare during migraines."*
   * *"Why high-contrast black & white dark mode causes halation for astigmatism—and a free tool to fix it."*
3. **Crosspost Targets:** `r/LowVision`, `r/Blind`, `r/Migraine`, `r/SensoryIssues`, `r/web_accessibility`.
