# Accessibility Tech Projects — Expanded Technical Specifications
## Low Vision · Photophobia · Sensory Utilities

> Synthesised from primary-source research across clinical ophthalmology, W3C specifications,
> browser internals, and OS accessibility APIs. The three web tools (ChromaCalm, SoftContrast,
> FocusBeacon) are OS-neutral and run without installation; `terminal-a11y` is a Python CLI
> package and requires installation (`pip install`).

---

## Table of Contents
1. [Core Science & Problem Space](#1-core-science--problem-space)
2. [Project 1 — ChromaCalm](#2-project-1--chromacalm)
3. [Project 2 — SoftContrast](#3-project-2--softcontrast)
4. [Project 3 — FocusBeacon](#4-project-3--focusbeacon)
5. [Project 4 — Terminal Accessibility Layer](#5-project-4--terminal-accessibility-layer)
6. [Distribution & Community Strategy](#6-distribution--community-strategy)

---

## 1. Core Science & Problem Space

### 1.1 The "High-Contrast" Myth

Most developers assume accessibility begins and ends with high-contrast dark mode (`#000000`
background / `#FFFFFF` text). Four clinical phenomena expose this as harmful:

| Phenomenon | Mechanism | Affected Populations |
|---|---|---|
| **Halation (Irradiation Effect)** | Intraocular light scatter (PSF) makes bright text bleed across dark backgrounds | Astigmatism, cataracts, post-LASIK HOAs |
| **Pupillary Mydriasis Trap** | Low total luminance dilates pupil to 5–7 mm; optical aberrations scale with r³–r⁶ of pupil radius | All dark-mode users, worst for astigmatism |
| **ipRGC Activation** | Melanopsin ganglion cells peak at 480 nm; LED blue-cyan spikes trigger trigeminal migraine pain | Migraine, photophobia, TBI/concussion |
| **Photostress Bleaching** | Pure white text on OLED black (400 cd/m²) causes 60–90 s recovery blindness vs 20–30 s normal | AMD, high myopia |

> **Primary Source:** Piepenbrock, Mayr & Buchner (2013/2014). Positive display polarity is
> advantageous for both younger and older adults. Ergonomics & Human Factors.
> DOI: 10.1177/0018720813515598

### 1.2 The Harvard 520 nm Discovery

**Study:** Noseda R, Bernstein CA, Burstein R et al. "Migraine photophobia originating in
cone-driven retinal pathways." Brain 139(7), 2016. DOI: 10.1093/brain/aww119

**Key Findings:**
- White, blue, amber, and red light exacerbated migraine pain in **~80% of patients**
- Narrow-band green at **520 ± 10 nm** exacerbated pain in only **~5%**
- At low-to-moderate intensities, 520 nm green **reduced headache intensity by 15–20%** in nearly 20% of patients
- The mechanism: 520 nm represents a physiological notch where S-cone + L-cone + melanopsin combined activation is minimised relative to perceived luminance

> Critical caveat: Broad-band "green" LEDs or monitor tints fail because any blue (<490 nm)
> or red (>580 nm) admixture restores nociceptive cone pathway activation.

### 1.3 FL-41 Spectral Filter

- **Origin:** Developed in Birmingham, UK (early 1990s) to counter fluorescent light glare
- **Spectrum:** Attenuates 480–520 nm (peak absorption 480–490 nm, blocking ~70–80% of blue-cyan LED spikes and the 436 nm mercury fluorescent spike)
- **Melanopsin Targeting:** The 480 nm notch directly suppresses melanopsin ipRGC peak (λ_max ≈ 480 nm)
- **Reported Efficacy:**
  - Migraine frequency: 6.2 → 1.6 attacks/month (Good et al., Headache 1991)
  - Blepharospasm: significant reduction in blink frequency and spasm force ([Blackburn et al., *Ophthalmology* 2009](https://pubmed.ncbi.nlm.nih.gov/19410958/))
  - Post-TBI photophobia: effective across >50% of TBI patients (Katz & Digre, Surv Ophthalmol 2016)
  - Green light: migraine pain reduced ~20% at low intensity and green was the least-exacerbating colour across intensities ([Noseda, Burstein et al., *Brain* 2016](https://pubmed.ncbi.nlm.nih.gov/27207542/))
- **Dark Adaptation Safety:** FL-41 maintains indoor VLT of 50–75%, avoiding the dark-adaptation rebound photophobia that sunglasses worn indoors cause

### 1.4 Amber Phosphor Photophysiology

Historical CRT P3/P134 phosphors (DEC VT220, IBM 3151, Wyse 50) emitted peak 585–590 nm amber.

| Effect | Blue 450 nm | Amber 590 nm | Ratio |
|---|---|---|---|
| Rayleigh scatter in ocular media | High | ~2.95× lower | (590/450)^4 |
| Melanopsin (ipRGC) activation | Peak (480 nm near) | Near-zero | — |
| Chromatic defocus | ~1.3 D in front of retina | On-fovea | — |
| Halation on dark BG | Severe | Minimal | — |

---

## 2. Project 1 — ChromaCalm
### Clinical FL-41 & 520 nm Green-Band Filter

**Target audiences:** Migraine, post-concussion syndrome, blepharospasm, photophobia,
Irlen Syndrome, Visual Snow Syndrome, post-refractive surgery (LASIK)

**Form factor:** Zero-install browser bookmarklet + single-file static HTML tool (GitHub Pages)

---

### 2.1 Technical Architecture

#### Why SVG feColorMatrix and Not a CSS Overlay div

A naive semi-transparent coloured div positioned over page content:
- Blends using sRGB alpha compositing, flattening perceptual contrast
- Makes already-dark text muddy and low-contrast
- Cannot implement spectral notch filtering — only additive colour blending

The feColorMatrix approach transforms every pixel through a 4×5 colour matrix.

> Note: ChromaCalm applies the matrix with `color-interpolation-filters="sRGB"`, not
> `linearRGB`. The preset matrices in §2.2 are the values that actually ship, so they are the
> authority for the current rendering — switching interpolation spaces would require
> re-deriving them, since the two change the result together.
> Source: W3C Filter Effects Module Level 1 §15
> https://www.w3.org/TR/filter-effects-1/#feColorMatrixElement

---

### 2.2 Spectral Preset Matrices

#### Preset 1 — Harvard 520 nm Green (Active Migraine)
Isolates the green subpixel channel, passing green through unweighted:
```
values="0 0 0 0 0   0 1 0 0 0   0 0 0 0 0   0 0 0 1 0"
```

#### Preset 2 — Clinical FL-41 Rose (Photophobia / Blepharospasm)
Scales the channels to 100% red, 70% green, 50% blue, with a +0.05 offset on green and blue:
```
values="1.0 0.0 0.0 0.0 0.0   0.0 0.7 0.0 0.0 0.05   0.0 0.0 0.5 0.0 0.05   0.0 0.0 0.0 1.0 0.0"
```

#### Preset 3 — Matte Paper / E-Ink (Low Stimulus Reading)
Scales red and green to 90%, blue to 80%, with a small cross-channel term and a +0.05 offset:
```
values="0.9 0.05 0.0 0.0 0.05   0.0 0.9 0.05 0.0 0.05   0.0 0.0 0.8 0.0 0.0   0.0 0.0 0.0 1.0 0.0"
```

#### Preset 4 — Sleep Preparation / Melatonin Mode [Creative Enhancement]
Passes only the red channel, zeroing green and blue:
```
values="1 0 0 0 0   0 0 0 0 0   0 0 0 0 0   0 0 0 1 0"
```

---

### 2.3 Cross-Browser SVG Injection: The Data URI Pattern

**Problem:** Safari and Firefox break `filter: url(#id)` when a `<base href>` tag is present.

**Solution:** Encode the entire SVG filter as an inline Data URI:
```javascript
const FILTER_URI = `url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg"><filter id="cc" color-interpolation-filters="sRGB"><feColorMatrix type="matrix" values="MATRIX_VALUES"/></filter></svg>#cc')`;
document.documentElement.style.filter = FILTER_URI;
```

**Bookmarklet constraints (2024–2026):**
- Firefox/Safari max bookmarklet URL: **64 KB**
- ChromaCalm minified bookmarklet: **< 2 KB**
- CSP restriction: all code must be self-contained inline JavaScript only

---

### 2.4 Operating Modes

| Mode | Description | API Used |
|---|---|---|
| **Green Light Bath** | Fullscreen calibrated 520 nm therapy lamp | Fullscreen API + Screen Wake Lock |
| **Anti-Halation Reader** | Sandboxed reading canvas with live filter toggling | SVG feColorMatrix |
| **Site-Wide Filter** | Applies to `document.documentElement` | Same |
| **Bookmarklet Exporter** | One-click drag-to-bookmarks-bar button | `javascript:` URI |
| **Sleep Preparation** | Red-only melatonin-safe mode | SVG feColorMatrix |
| **Userscript Fallback** | For enterprise sites with strict CSP | Tampermonkey GM_addStyle |

**Screen Wake Lock** (`navigator.wakeLock.request('screen')`): Prevents phone/tablet screen from
dimming during a migraine episode when used as a green light therapy lamp.
Supported: Chrome 84+, Edge 84+, Safari 16.4+ (all OS-neutral via browser).

---

### 2.5 Creative Enhancements

**Gradual Transition Engine:** Fades the matrix from identity to the target preset over
configurable 800–3000 ms using CSS `transition: filter`. Prevents startle responses in
photophobia patients during filter activation.

**Adaptive Intensity via Time-of-Day:** Uses `new Date().getHours()` to pre-select defaults:
- 07:00–12:00 → Matte Paper (morning comfort)
- 12:00–20:00 → FL-41 (afternoon LED exposure)
- 20:00–07:00 → Sleep Preparation (melatonin protection)

**Stimulus Journal (localStorage):** A lightweight log records which preset was active at
what time, allowing users to correlate filter use with migraine onset/relief — exportable
as CSV for neurologists. No server, no account, no data transmission.

**Audio Confirmation Tone:** When a filter activates, a single soft 440 Hz sine wave plays
for 120 ms via Web Audio API — confirms activation without requiring the user to look at UI
(useful mid-migraine when looking at a screen is painful).

---

## 3. Project 2 — SoftContrast
### Anti-Halation Reading Palette Generator

**Target audiences:** Low vision, high myopia, astigmatism, age-related macular degeneration (AMD),
Irlen Syndrome, extended screen-time reading professionals

**Form factor:** Micro web application + exportable Stylus userstyle / Tampermonkey userscript

---

### 3.1 Why APCA Instead of WCAG 2.1 Contrast

WCAG 2.1 SC 1.4.3 uses the formula `(L1 + 0.05) / (L2 + 0.05)` with a 4.5:1 minimum.

| Problem | WCAG 2.1 Behaviour | APCA Behaviour |
|---|---|---|
| **Polarity blindness** | White-on-black = black-on-white (both 21:1) | Separate curves for each polarity |
| **Low-luminance false passes** | Very dark grey pairs pass even when illegible | SoftToe black clamp catches these |
| **Font size decoupling** | Only 2 tiers (normal / large text) | Continuous spatial frequency coupling |

**APCA 0.0.98G-4g Algorithm:**
```
Y = 0.2126729*(R/255)^2.4 + 0.7151522*(G/255)^2.4 + 0.0721750*(B/255)^2.4
# SoftToe black clamp:
If Y <= 0.022: Y = Y + (0.022 - Y)^1.414
# Normal polarity (dark text on light BG):
Lc = (Y_bg^0.56 - Y_txt^0.57) * 1.14  (with -0.027 low-clamp offset)
# Reverse polarity (light text on dark BG):
Lc = (Y_bg^0.65 - Y_txt^0.62) * 1.14  (with +0.027 low-clamp offset)
```

> Sources: Somers, A. apca-w3: https://github.com/Myndex/apca-w3
> Roselli, A. (2024) APCA is not WCAG3: https://adrianroselli.com/2024/02/apca-is-not-wcag3.html

**SoftContrast targets APCA Lc ≥ 60** for body text — achieving legibility without the
halation-inducing extremes of Lc 95+ (pure white-on-black).

---

### 3.2 Curated Anti-Halation Presets

| Palette | Background | Text | APCA Lc | Character |
|---|---|---|---|---|
| **Midnight Ochre** | `#141416` (15 cd/m²) | `#D6D0C4` (90 cd/m²) | ~72 | Zero glow, warm charcoal |
| **Solar Flare Amber** | `#1A1200` | `#F5C842` | ~68 | Phosphor amber — migraine-safe |
| **Warm Slate** | `#1C1B1E` | `#E8D9C0` | ~70 | Neutral warm dark |
| **Sepia Paper** | `#F7F0E3` | `#2C2416` | ~78 | Positive polarity — AMD-optimised |
| **520 nm Reading** | `#0A120A` | `#7DCC7D` | ~55 | Green-band, Burstein-compliant |
| **FL-41 Night** | `#180B10` | `#D4A8B0` | ~58 | Rose-tinted reverse polarity |

---

### 3.3 OKLCH Palette Generation

SoftContrast generates extended palettes in **OKLCH** rather than HSL or HEX because:

- **Perceptually uniform lightness:** Stepping L by 0.1 guarantees equal perceived contrast regardless of hue
- **Hue stability:** No blue-to-purple Abney effect distortion (a known CIELAB flaw at 250°–300°)
- **Gamut-safe:** CSS Color 4 mandates OKLCH chroma reduction (not hue distortion) for sRGB mapping

**3-tier Token Architecture:**
```css
/* Primitive */
--sc-raw-charcoal: oklch(0.16 0.006 285);
/* Semantic */
--color-bg-canvas: var(--sc-raw-charcoal);
--color-text-body: oklch(0.82 0.012 80);
/* Component */
--article-bg: var(--color-bg-canvas);
```

---

### 3.4 Cross-Site Injection: Stylus UserCSS

**Eliminating Flash of Unstyled Content (FOUC):** Tampermonkey scripts set to
`@run-at document-start` inject CSS onto `document.documentElement` before any page HTML
renders, preventing the jarring white flash.

**CSP Bypass:** `GM_addElement(document.documentElement, 'style', { textContent: css })`
runs in the extension isolated world, bypassing even strict `style-src 'none'` CSPs.

**Safe Cross-Site Injection Rules:**
```css
html, body, main, article, p, li, h1, h2, h3, h4, h5, h6 {
  background-color: var(--color-bg-canvas) !important;
  color: var(--color-text-body) !important;
}
/* Preserve icon SVGs */
svg { fill: currentColor !important; }
/* Exempt images and video — never filter media */
img, video, canvas, [role="img"] {
  background: transparent !important;
  filter: none !important;
}
```

---

### 3.5 Export Formats

| Format | Use Case |
|---|---|
| **CSS Custom Properties** | Direct site theming |
| **DTCG JSON tokens** | Figma Variables, Tokens Studio, Style Dictionary |
| **Stylus `.user.css`** | Per-site browser override |
| **Tampermonkey `.user.js`** | CSP-protected sites, FOUC-free |
| **URL hash share link** | `#name=Midnight+Ochre&bg=141416&fg=d6d0c4` — zero server |
| **Tailwind v4 `@theme`** | Framework integration |

---

### 3.6 Creative Enhancements

**Visual Fatigue Estimator:** Before applying any palette, SoftContrast samples the page's
computed background and text colours via `getComputedStyle`, calculates APCA Lc, and displays
a "halation risk score" (Low / Medium / High) so users understand *why* a site is causing strain.

**Per-Domain Memory:** `localStorage` keyed to `window.location.hostname` stores the last-used
palette per site — preferences remembered without an account.

**Font Rendering Mode Toggle:**
- Subpixel antialiasing (`-webkit-font-smoothing: subpixel-antialiased`) — sharper on non-Retina LCD
- Grayscale antialiasing (`-webkit-font-smoothing: antialiased`) — reduced colour fringing for astigmatism

**Reading Ruler:** An optional horizontal highlight band follows the cursor Y-position in the
palette's accent colour at 15% opacity to support line tracking without a physical ruler.
(Tinted lenses/overlays for dyslexia are not supported by the 2009 AAP/AAO joint statement — [Learning Disabilities, Dyslexia and Vision](https://www.aao.org/education/clinical-statement/learning-disabilities-dyslexia-vision).)

---

## 4. Project 3 — FocusBeacon
### Dual-Ring Keyboard Focus & Cursor Radar

**Target audiences:** Tunnel vision (glaucoma, retinitis pigmentosa), motor/keyboard-only
navigators, screen magnifier users, low vision on 4K/ultrawide displays

**Form factor:** Vanilla JavaScript drop-in library, ~13 KB minified (zero dependencies, zero frameworks)

---

### 4.1 Clinical Motivation: Tunnel Vision

**Glaucoma:** Arcuate scotomas coalesce into dense peripheral loss, leaving only a tiny foveal
island (<10°). **Retinitis Pigmentosa (RP):** Contracting annular ring scotomas — "looking
through a cardboard tube."

**Critical screen interaction consequences:**
1. **Pre-attentive pop-out destroyed:** No peripheral vision means cursor/focus movements
   never trigger reflexive saccades — must scan manually
2. **Serial raster sweeping:** "Lighthouse technique" row-by-row scanning. Task time 3–5× longer
3. **Lost cursor on 4K:** A 24 px cursor subtends <0.35° on a 4K/55° FOV monitor. Locating
   it within a 5° functional field is associated with slower visual search in peripheral field loss
4. **Focus jumps cause disorientation:** Large DOM focus teleports (>800 px / 25°) take the
   focus ring entirely outside the patient's functional field of view

> Sources: Smith, Glen & Crabb (2012) BMC Ophthalmology 12:45. DOI: 10.1186/1471-2415-12-45
> Crabb et al. (2013) Ophthalmology 120(6):1120. DOI: 10.1016/j.ophtha.2012.11.043

---

### 4.2 Dual-Contour Focus Ring: Mathematical Guarantee

**The technique (W3C Technique C40):** White inner ring (1.5 px) + black outer ring (1.5 px).

In sRGB space, for any background luminance L_bg ∈ [0, 1]:
- Black ring contrast: `CR_black = 20·L_bg + 1`
- White ring contrast: `CR_white = 1.05 / (L_bg + 0.05)`

Worst-case background luminance: `L*_bg ≈ 0.1791`
**Minimum guaranteed contrast: ≥ 4.58:1 against ANY background colour**

This exceeds WCAG SC 2.4.13's 3:1 focus requirement by 52.7% and meets the 4.5:1 Level AA
text contrast threshold — universally valid without testing against specific backgrounds.

> Sources: W3C Technique C40: https://www.w3.org/WAI/WCAG22/Techniques/css/C40
> GOV.UK Design System Focus States: https://design-system.service.gov.uk/get-started/focus-states/

---

### 4.3 WCAG 2.2 Focus Appearance (SC 2.4.13)

Introduced in WCAG 2.2 (October 2023), Level AAA:
- **Size:** Focus indicator area ≥ 2 CSS px thick perimeter of the unfocused component
- **Contrast:** Minimum 3:1 between focused and unfocused states

FocusBeacon's 4.58:1 dual-contour guarantee exceeds this requirement.

---

### 4.4 CSS Implementation: outline vs box-shadow

| Property | Windows High Contrast | Multiple rings | Border-radius | Overflow clip |
|---|---|---|---|---|
| `outline` | Preserved | One ring only | Chrome 94+ | Clipped |
| `box-shadow` | **STRIPPED** | Multiple rings | Universal | Clipped |

**Critical:** `box-shadow` is stripped under `forced-colors: active` (Windows High Contrast Mode).
Always pair with a transparent outline:
```css
:focus-visible {
  outline: 2px solid transparent; /* preserved in forced-colors */
  box-shadow: 0 0 0 1.5px #fff, 0 0 0 3px #000; /* dual contour */
}
```

**Overflow clipping solution:** FocusBeacon uses a floating DOM overlay appended to
`document.body`, repositioned via `getBoundingClientRect()` — completely escaping
`overflow: hidden` and `clip-path` on parent containers.

---

### 4.5 focus-visible vs focus

```css
:focus:not(:focus-visible) { outline: none; }  /* no ring for mouse clicks */
:focus-visible { /* dual contour ring for keyboard navigation */ }
```

`:focus-visible` fires for keyboard navigation but NOT for mouse clicks on buttons/links.
Browser support: Chrome 86+, Firefox 85+, Safari 15.4+, Edge 86+ (>96.8% global coverage).

---

### 4.6 Cursor Radar (Double-Tap Ctrl)

**Trigger:** Two `Control` keydown events within 350 ms.

**Rendering — DOM overlay vs Canvas:**

| Approach | VRAM at 4K (DPR 2) | CPU load |
|---|---|---|
| Fullscreen canvas | ~132 MB | Continuous redraw loop |
| Pooled DOM div with translate3d | **< 10 KB** | Zero layout thrashing |

FocusBeacon uses the DOM approach with Web Animations API.

**prefers-reduced-motion degradation:**
- **Normal motion:** Expanding concentric ripple, scale 0.2→2.8 over 550 ms, cubic-bezier easing
- **Reduced motion:** Instant static high-contrast reticle at cursor position, fades after 750 ms

**Pointer Lock guard:** If `document.pointerLockElement !== null`, cursor radar is suspended
to prevent rendering at a disconnected absolute position.

**Stacking Context Isolation:**
```javascript
overlay.style.cssText = `position:fixed; z-index:2147483647; pointer-events:none; contain:strict;`;
if (overlay.showPopover) overlay.showPopover(); // Top Layer when supported
```

---

### 4.7 Creative Enhancements

**Focus Trail Mode:** Stores the last 5 focus element positions as fading breadcrumb halos
(opacity 60% → 20%) so users who lose orientation mid-navigation can visually retrace their path.
Enable with `data-focus-trail="5"` attribute on the script tag.

**Saccade Animation on Large Focus Jumps:** When a focus jump exceeds 300 px, a trailing line
briefly animates between the previous and new focus positions over 180 ms, giving tunnel vision
users a directional cue about *where* focus moved.

**Skip-Link Beacon:** FocusBeacon automatically detects `<a href="#main-content">` skip links
and enhances them with a persistent pulsing arrow indicator in the top-left viewport corner
when Tab is first pressed — making skip links discoverable to users who didn't know they existed.

**Developer Accessibility HUD:** In development mode (`data-focus-dev="true"`), a floating
panel shows:
- Current element's WCAG 2.2 focus contrast score
- Whether `:focus-visible` is active
- DOM path of focused element
- `tabindex` value

---

## 5. Project 4 — Terminal Accessibility Layer
### --screen-reader and --photophobia CLI Enhancement Flags

**Target audiences:** Visually impaired and light-sensitive software engineers, sysadmins,
data scientists, DevOps engineers

**Form factor:** Python library (pip installable, OS-neutral) + standalone shell wrapper

---

### 5.1 The Problem: Modern Terminals Break Screen Readers

| Platform | Bridge | Mechanism |
|---|---|---|
| Windows (NVDA/JAWS) | Windows Terminal UIA TextPattern | UIA_Text_TextChangedEventId |
| Linux (Orca) | AT-SPI2 over D-Bus | VTE AtkText / AtspiText |
| macOS (VoiceOver) | NSAccessibility in Terminal.app | Rapid rewrites drop speech |

**What breaks screen readers:**
1. `\r` + `\x1b[2K` spinner loops overwrite the same line 10–12×/second, flooding speech queues
2. ASCII progress bars read as "box drawings light horizontal" endlessly
3. Braille spinner frames read as meaningless dot-pattern glyphs
4. Colour-only status indicators invisible to blind users

---

### 5.2 ANSI Stripping

**Canonical stripping regex:**
```python
import re
ANSI_PATTERN = re.compile(
    r'\x1b(?:[@-Z\\-_]|\[[\x30-\x3f]*[\x20-\x2f]*[\x40-\x7e]|\].*?(?:\x07|\x1b\\))',
    re.DOTALL
)
def strip_ansi(text: str) -> str:
    return ANSI_PATTERN.sub('', text)
```

> Sources: ECMA-48 5th Edition: https://www.ecma-international.org/publications/standards/Ecma-048.htm
> chalk/ansi-regex: https://github.com/chalk/ansi-regex

---

### 5.3 Screen Reader Auto-Detection

```python
import sys, os, subprocess, ctypes

def detect_screen_reader() -> bool:
    if os.environ.get('SCREEN_READER') or os.environ.get('ACCESSIBILITY_ENABLED'):
        return True
    if os.environ.get('TERM') == 'dumb':
        return True
    if sys.platform == 'win32':
        val = ctypes.c_bool()
        ctypes.windll.user32.SystemParametersInfoW(0x0046, 0, ctypes.byref(val), 0)
        return bool(val.value)
    if sys.platform == 'darwin':
        r = subprocess.run(['defaults', 'read', 'com.apple.universalaccess',
                            'voiceOverOnOffKey'], capture_output=True, text=True)
        return r.stdout.strip() == '1'
    if sys.platform.startswith('linux'):
        r = subprocess.run(['gsettings', 'get', 'org.gnome.desktop.a11y.applications',
                            'screen-reader-enabled'], capture_output=True, text=True)
        return 'true' in r.stdout.lower()
    return False
```

---

### 5.4 --screen-reader Mode Specification

When active:
1. **Strip all ANSI codes** (colour, cursor, SGR)
2. **Replace spinners and progress bars** with milestone text updates (25%, 50%, 75%, Complete)
3. **Linearise output** — no `\r` back-overwriting; every status update on a new line
4. **Explicit status tokens** (WCAG 1.4.1):
   - `✓` → `[PASS]` | `✗` → `[FAIL]` | `⚠` → `[WARN]` | `ℹ` → `[INFO]`
5. **Audible completion bell** (`\a`) on long operation finish (WCAG 4.1.3)
6. **Structured section headers** for NVDA document mode navigation:
   ```
   === SECTION: Network Diagnostics ===
   ```

---

### 5.5 --photophobia Mode Specification

#### Amber Phosphor Palette

| Role | Hex | ANSI 256-colour | 16-colour fallback |
|---|---|---|---|
| Background | `#120E04` | 232 | Black `40` |
| Primary text (amber) | `#FFB000` | 214 | Yellow `33` |
| Bright (headings) | `#FFCC00` | 220 | Bright Yellow `93` |
| Dim (comments) | `#C47D00` | 172 | Dark Yellow |
| Error | `#FF6B35` | 202 | Red `31` |
| Success | `#7DCC7D` | 108 | Green `32` |

**Contrast verification:** `#FFB000` on `#120E04` = **8.4:1** (exceeds WCAG AA 4.5:1)

#### Cross-Platform TrueColor Detection

```python
def get_color_depth() -> int:
    if os.environ.get('NO_COLOR'):
        return 0   # NO_COLOR standard — always first
    if not sys.stdout.isatty():
        return 0
    if os.environ.get('COLORTERM') in ('truecolor', '24bit'):
        return 24
    if os.environ.get('WT_SESSION'):          # Windows Terminal
        return 24
    if os.environ.get('TERM_PROGRAM') == 'Apple_Terminal':
        return 256  # Apple Terminal.app: NO TrueColor support
    if os.environ.get('TERM', '').endswith('-256color'):
        return 256
    return 16
```

> IMPORTANT: Apple Terminal.app has never supported 24-bit TrueColor.
> Tools emitting `\x1b[38;2;R;G;Bm` codes corrupt output in Terminal.app.

**NO_COLOR Standard (no-color.org):** Presence of `NO_COLOR`, not its value, disables colour.
Even `NO_COLOR=0` must disable colour. Respect unconditionally.

---

### 5.6 WCAG for Command-Line Interfaces (WCAG2ICT 2024)

| SC | Requirement | CLI Implementation |
|---|---|---|
| 1.1.1 Non-text Content | Text alternative for non-text | Replace ASCII charts with tabular data |
| 1.3.2 Meaningful Sequence | No non-linear reading order | No `\r` overwriting |
| 1.4.1 Use of Color | Not colour alone for meaning | Explicit [PASS] / [FAIL] tokens |
| 1.4.3 Contrast | 4.5:1 minimum | Amber on charcoal = 8.4:1 |
| 2.2.2 Pause/Stop/Hide | User can disable animations | --screen-reader / --no-spinner flags |
| 4.1.3 Status Messages | Programmatically determinable | Structured headers + audible bell |

---

### 5.7 Creative Enhancements

**Plain Language Error Rewriter:** When `--screen-reader` is active and an exception is caught,
a local offline pattern-matching library (no remote API) translates technical errors:
- `ConnectionRefusedError: [Errno 111] Connection refused` →
  `[ERROR] Could not connect. The remote server is not responding. Check network and try again.`
- `PermissionError: [Errno 13] Permission denied` →
  `[ERROR] Access denied. You may need administrator privileges.`

**Sensory Budget Mode (`--sensory-budget`):** Tracks cumulative terminal output volume in a
session. When the budget (default 2000 lines) is exceeded, output is automatically throttled to
summary-only (`[N lines suppressed — use --verbose to show]`). Designed for users whose
photophobia or visual fatigue worsens with sustained terminal use.

**Braille-Optimised Layout (`--braille`):** When `BRAILLE_DISPLAY=1` is set, output is
reformatted to 40-character line width (standard Braille display width), abbreviations are
expanded, and all emoji are replaced with ASCII equivalents.

**Audio Progress (`--audio-progress`):** Uses platform-native audio (winsound on Windows,
afplay on macOS, aplay on Linux) to emit tones at 25%/50%/75%/100% milestones — providing
non-visual progress feedback for blind users or screen magnifier users zoomed far in.

---

## 6. Distribution & Community Strategy

### 6.1 GitHub Pages Deployment (Zero Cost)

| Project | Repo Structure | Distribution |
|---|---|---|
| ChromaCalm | `chromacalm.html` (single file, ~23KB) | GitHub Pages root (not yet enabled) |
| SoftContrast | `index.html` + `css/` + 7 JS modules | GitHub Pages root (not yet enabled) |
| FocusBeacon | `focusbeacon.min.js` (~13KB) + `index.html` | GitHub Pages (not yet enabled); npm/jsDelivr publishing pending |
| Terminal Layer | Python package (`terminal_a11y`) | GitHub only; PyPI publishing pending |

### 6.2 Problem-First Authentic Messaging

- *"I got tired of screen dimmers making text muddy — built a zero-install bookmarklet using Harvard's 520nm green-band research to kill screen glare during migraines"*
- *"Why high-contrast dark mode causes halation for astigmatism (and a free APCA-based palette generator that actually fixes it)"*
- *"The cursor just disappears on my 4K monitor — here's a 13KB script that adds a double-tap Ctrl radar ring to find it instantly"*
- *"I built CLI flags for screen reader users tired of spinners crashing NVDA"*

### 6.3 Subreddit Targeting

| Subreddit | Relevant Projects |
|---|---|
| `r/LowVision` | All four |
| `r/Blind` | FocusBeacon, Terminal Layer |
| `r/Migraine` | ChromaCalm, SoftContrast |
| `r/SensoryIssues` | ChromaCalm, SoftContrast |
| `r/web_accessibility` | FocusBeacon, SoftContrast |
| `r/programming` | Terminal Layer |
| `r/linux` | Terminal Layer |
| `r/glaucoma` | FocusBeacon |
| `r/TBI` | ChromaCalm |

### 6.4 The Sensory Stack — Cross-Project Integration

The four projects form a coherent layered toolkit:

```
ChromaCalm bookmarklet  (spectral filter — any browser, any OS)
    +
SoftContrast userstyle  (palette override — persistent per-domain)
    +
FocusBeacon script      (navigation beacon — keyboard users)
    +
Terminal Layer          (CLI tools — command-line users)
```

A single landing page presenting all four as a unified "Sensory Stack" creates a stronger
community foothold than four disconnected repos, and allows users to layer solutions as needed.

---

## Research Sources

All findings are fully documented and cited in the `/research/` directory:

| File | Content | Lines |
|---|---|---|
| `research/chromacalm-research.md` | 520nm science, FL-41, SVG matrices, bookmarklet | 560 |
| `research/softcontrast-research.md` | APCA math, OKLCH, halation, CSS injection | 672 |
| `research/focusbeacon-research.md` | WCAG 2.2, dual-contour proof, tunnel vision | 763 |
| `research/terminal-accessibility-research.md` | ANSI, screen readers, amber phosphor, detection | 871 |

---

*Document generated: 2026-09-09; updated 2026-09-10 | All projects open-source; the three web tools are OS-neutral and zero-install, `terminal-a11y` requires Python*
