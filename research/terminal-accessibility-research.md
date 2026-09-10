# Terminal Accessibility & Sensory Enhancement Layer (Project 4)
## Deep Technical Research: Specifications, Ergonomics, Screen Reader Compatibility, and Implementation Architecture

> **Document Status:** Authoritative Technical Research & Architectural Specification  
> **Date:** September 2026  
> **Scope:** Design and implementation of `--screen-reader` (`--sr`) and `--photophobia` (`--soft`) flags for command-line utilities.  
> **Target Systems:** Cross-platform (Linux, macOS, Windows; xterm, VTE, Apple Terminal, Windows Terminal, ConHost).

---

## Executive Summary

Command-line interfaces (CLIs) and terminal emulators are frequently mischaracterized as inherently accessible because their underlying substrate is textual. In practice, modern CLI developer tooling is heavily visual: tools employ ANSI/VT escape sequences, 24-bit TrueColor styling, dynamic cursor positioning (`\r`, `CSI 2K`), animated braille spinners, Unicode box-drawing tables, and visual progress bars. 

For **blind and low-vision developers using screen readers** (NVDA, Orca, VoiceOver), these dynamic visual tricks cause severe accessibility failures: speech synthesizers stutter or crash from rapid cursor repositioning, braille progress indicators are read aloud as meaningless character streams, and critical state information communicated solely via ANSI color codes is lost.

For **developers with photophobia, migraines, post-concussion syndrome, or astigmatic halation**, standard terminal aesthetics are equally punishing. Default high-contrast dark mode (pure `#FFFFFF` text on pitch `#000000`) induces severe optical halation in irregular corneas, while modern 24-bit themes emit high-energy blue-cyan light (450nm–480nm) that aggressively stimulates intrinsically photosensitive Retinal Ganglion Cells (ipRGCs), triggering trigeminal pain pathways.

This research report provides the technical foundation for **Project 4**: a terminal accessibility enhancement layer providing:
1. `--screen-reader` (`--sr`): Linearizes output, strips progress bars/spinners, converts interactive prompts to numbered lists, adds audible bell signals (`\a`), and ensures semantic textual labelling.
2. `--photophobia` (`--soft`): Maps neon terminal colors to a photophysiologically calibrated amber phosphor palette (~590nm peak) and soft oatmeal tones, eliminating halation and suppressing ipRGC activation.

---

## 1. ANSI Escape Code Specifications & Stripping Mechanics

Terminal control sequences are governed by formal international standards, notably **ECMA-48** and **ISO/IEC 6429**. Understanding their byte structure is necessary to reliably intercept, re-theme, or strip them without corrupting standard text.

### 1.1 Standards Overview
* **ECMA-48 (5th Edition, June 1991):** *"Control Functions for Coded Character Sets"* ([Ecma International ECMA-48 Spec](https://www.ecma-international.org/publications-and-standards/standards/ecma-48/)).
* **ISO/IEC 6429:** International standard identical to ECMA-48.
* **ANSI X3.64:** American National Standard adopted in 1979, later withdrawn in favor of ISO/IEC 6429.
* **ITU-T Recommendation T.416 / ISO/IEC 8613-6:** Open Document Architecture (ODA) specification governing 24-bit TrueColor SGR extensions ([ITU-T Rec. T.416](https://www.itu.int/rec/T-REC-T.416-199303-I/en)).

### 1.2 Control Sequence Structure (ECMA-48 § 5.4)
Control sequences begin with the **Control Sequence Introducer (CSI)**:
* **7-bit representation:** `ESC [` (`0x1B 0x5B` or `\033[` / `\x1b[`).
* **8-bit representation:** Single byte `0x9B` (C1 control code). Modern UTF-8 environments almost universally use the 2-byte 7-bit form because `0x9B` is an invalid continuation byte in UTF-8.

A standard CSI sequence follows the grammar:
$$\text{CSI} \quad [P \dots P] \quad [I \dots I] \quad F$$
* **Parameters ($P$):** Zero or more ASCII digits (`0x30`–`0x39`) separated by semicolons (`0x3B`).
* **Intermediates ($I$):** Zero or more bytes in range `0x20`–`0x2F`.
* **Final Byte ($F$):** Exactly one terminating command byte in range `0x40`–`0x7E`.

```
           +------- ESC (0x1B)
           | +----- [   (0x5B) : Control Sequence Introducer (CSI)
           | |
           v v
Sequence: \x1b [ 3 8 ; 2 ; 2 5 5 ; 1 7 6 ; 0 m
                 ^-------------------------^ ^
                              |              |
                      Parameter Bytes     Final Byte (SGR: Select Graphic Rendition)
```

### 1.3 Key ANSI Escape Codes by Category

#### Select Graphic Rendition (SGR) — Styling and Color (ECMA-48 § 8.3.117)
The final character is `m`.

| Code (`CSI n m`) | Action | Accessibility / Sensory Impact |
| :--- | :--- | :--- |
| `0` | Reset all attributes to default | Resets any active color or style. |
| `1` | Bold / Increased intensity | Increases font weight or brightness. |
| `2` | Faint / Decreased intensity | Reduces luminance contrast (hazardous for low vision). |
| `3` | Italicized | Screen readers ignore; low-vision readers find it harder to parse. |
| `4` | Singly underlined | Used for links or emphasis. |
| `5` | Slowly blinking (< 150 bpm) | **Severe cognitive/seizure violation** (WCAG 2.3.1). Must be stripped. |
| `6` | Rapidly blinking ($\ge$ 150 bpm) | **Critical seizure hazard**. Must be stripped unconditionally. |
| `7` | Reverse video (negative image) | Swaps foreground and background. |
| `8` | Concealed (hidden) | Invisible text; screen readers may still read it or buffer it. |
| `9` | Crossed-out (strikethrough) | Visual cancellation; screen readers rarely announce deletion. |
| `30`–`37` | Standard foreground (Black, Red, Green, Yellow, Blue, Magenta, Cyan, White) | Basic 3-bit color palette. Red/Green causes deuteranopia failure. |
| `38;5;n` | 256-color extended foreground | 8-bit palette ($0 \le n \le 255$). |
| `38;2;r;g;b` | 24-bit TrueColor foreground | Direct RGB specification ($0 \le r,g,b \le 255$). |
| `39` | Default foreground color | Terminal default. |
| `40`–`47` | Standard background color | 3-bit background palette. |
| `48;5;n` | 256-color extended background | 8-bit background palette. |
| `48;2;r;g;b` | 24-bit TrueColor background | Direct RGB background. |
| `49` | Default background color | Terminal default background. |
| `90`–`97` | High-intensity / Bright foreground | AIXterm extension (Bright Black to Bright White). |
| `100`–`107` | High-intensity / Bright background | AIXterm extension bright backgrounds. |

#### Cursor Positioning & Screen Clearing (ECMA-48 § 8.3)
| Sequence | Name | Description | Screen Reader Impact |
| :--- | :--- | :--- | :--- |
| `CSI n A` | CUU (Cursor Up) | Moves cursor up $n$ rows | Overwrites previously read buffer lines. |
| `CSI n B` | CUD (Cursor Down) | Moves cursor down $n$ rows | Jumps visual layout. |
| `CSI n C` | CUF (Cursor Forward) | Moves cursor right $n$ columns | Creates horizontal spacing gaps. |
| `CSI n D` | CUB (Cursor Backward) | Moves cursor left $n$ columns | Backspaces across buffer. |
| `CSI n G` | CHA (Cursor Character Absolute) | Moves cursor to column $n$ | Resets horizontal position. |
| `CSI n ; m H` | CUP (Cursor Position) | Sets cursor to row $n$, col $m$ | Random access buffer writing; destroys stream linearity. |
| `CSI n J` | ED (Erase in Page) | $0$: cursor to end; $1$: start to cursor; $2$: entire screen; $3$: clear scrollback | $n=2$ wipes visible text; $n=3$ destroys user's review buffer. |
| `CSI n K` | EL (Erase in Line) | $0$: cursor to end; $1$: start to cursor; $2$: entire line | Erases current line content before redrawing. |

#### Control Characters (C0 Set)
* **Carriage Return (`\r` / `0x0D`):** Moves the cursor to column 0 on the current line without advancing downward. This is the primary mechanism used by progress bars and spinners to rewrite lines.
* **Line Feed (`\n` / `0x0A`):** Moves the cursor down one line.
* **Bell (`\a` / `0x07`):** Generates an audible or visual alert. Critical positive signal for screen reader users.

### 1.4 Mechanics of CLI Spinners and Progress Bars
CLI libraries (such as `ora`, `cli-progress`, `tqdm`, `indicatif`, or `rich.progress`) achieve visual animation via a tight update loop:
1. Move cursor to column 0: output `\r`.
2. Erase the existing line: emit `\x1b[2K` (EL 2).
3. Print updated glyphs: e.g., braille frame `⠋` or bar `[=======>    ] 58%`.
4. Sleep for 80–100ms.
5. Repeat.

**Screen Reader Consequence:** Screen readers listening for terminal text updates receive 10 to 12 `TextChange` events per second on the exact same row. The screen reader either:
* Spews rapid-fire audio: *"Braille pattern dots 1 4, braille pattern dots 1 2 4, bracket equals equals equals..."*
* Completely chokes its speech queue, dropping actual diagnostic messages.
* Stalls system responsiveness while speech synthesis attempts to catch up.

### 1.5 Stripping ANSI Codes: Regular Expressions & Utilities
To eliminate ANSI codes reliably, a parser must cover CSI sequences, Operating System Commands (OSC), and C1 control characters.

#### Standard ANSI Regex Pattern (Chalk / strip-ansi)
The canonical regex used by the JavaScript and Python ecosystems (derived from [chalk/ansi-regex](https://github.com/chalk/ansi-regex)):

```regex
[\u001B\u009B][\[\]()#;?]*(?:(?:(?:(?:;[-a-zA-Z\d\/#&.:=?%@~_]+)*|[a-zA-Z\d]+(?:;[-a-zA-Z\d\/#&.:=?%@~_]+)*)?\u0007)|(?:(?:\d{1,4}(?:;\d{0,4})*)?[\dA-PRZcf-ntqry=><~]))
```

#### Production Python Regex Stripper
```python
import re

# Comprehensive ANSI escape sequence pattern
ANSI_ESCAPE_PATTERN = re.compile(
    r"""
    \x1B                     # ESC byte
    (?:                      # 7-bit C1 Fe sequence
        [@-Z\\-_]            # 2-character escape sequence
    |                        # or
        \[                   # CSI [
        [0-?]*               # Parameter bytes (0x30-0x3F: digits, semicolons, etc.)
        [ -/]*               # Intermediate bytes (0x20-0x2F)
        [@-~]                # Final byte (0x40-0x7E)
    |                        # or
        \]                   # OSC ]
        .*?                  # Operating System Command payload
        (?:\x07|\x1B\\)      # BEL (0x07) or ST (ESC \) terminator
    )
    """,
    re.VERBOSE
)

def strip_ansi(text: str) -> str:
    """Strip all ANSI escape codes, OSC sequences, and CSI commands."""
    return ANSI_ESCAPE_PATTERN.sub('', text)
```

#### Canonical ANSI Filtering Tools
* **Ansifilter (by André Simon):** C++ command-line utility and library hosted on [GitLab: saalen/ansifilter](https://gitlab.com/saalen/ansifilter). Handles ANSI stripping, HTML/RTF conversion, and ISO 6429 parsing.
* **col -b:** Standard POSIX utility (`col`) with `-b` flag filters out backspaces and carriage returns, retaining only the last written characters on each column.

---

## 2. Screen Reader Terminal Compatibility & Failure Modes

Screen readers do not inspect video memory; they rely on OS accessibility APIs and terminal emulator accessibility bridges.

```
+------------------+         +-----------------------+         +---------------------+
| CLI Application  | stdout  |   Terminal Emulator   | UIA /   |    Screen Reader    |
| (python/node.js) | ------> | (Windows Terminal,    | AT-SPI  | (NVDA, Orca,        |
|                  |         |  GNOME Terminal, etc) | ------> |  VoiceOver, JAWS)   |
+------------------+         +-----------------------+         +---------------------+
```

### 2.1 Screen Reader Architecture Across Platforms

#### 1. Windows: NVDA & JAWS via Windows Terminal / ConHost
* **Bridge:** Microsoft UI Automation (UIA) `TextPattern` and `UiaRaiseAutomationEvent` ([Microsoft UIA Console Architecture](https://learn.microsoft.com/en-us/windows/console/accessibility)).
* **Mechanism:** Windows Terminal exposes its buffer as a UIA text document. When new output arrives, Windows Terminal dispatches `UIA_Text_TextChangedEventId`.
* **NVDA Behavior:** NVDA monitors buffer changes. However, when an application uses `\r` to overwrite the current line, Windows Terminal modifies the existing text range. NVDA's review cursor either loses track of the current insertion point or re-announces the entire modified line every time the spinner updates.
* **Ecosystem Solution:** Power users install the [Terminal Access for NVDA add-on](https://github.com/mwhapples/terminal-access), which provides specialized terminal review keys (`NVDA + '`), tabular data parsing, and bookmarking.

#### 2. Linux: Orca via VTE / AT-SPI2
* **Bridge:** Assistive Technology Service Provider Interface (AT-SPI2) over D-Bus (`org.a11y.Bus`) ([GNOME AT-SPI2 Documentation](https://gitlab.gnome.org/GNOME/at-spi2-core)).
* **Mechanism:** Terminals based on the VTE library (GNOME Terminal, XFCE Terminal, Tilix) implement the `AtkText` / `AtspiText` interface. When characters change, VTE emits `text-changed::insert` and `text-changed::delete` signals.
* **Orca Behavior:** Orca attempts to narrate live terminal output via "flat review". Spinners and progress bars trigger an avalanche of D-Bus signals, causing significant input lag and speech synthesizer stuttering.
* **Console Alternative:** For pure TTYs without X11/Wayland, blind Linux users rely on **Speakup** (an in-kernel screen reader) or **BRLTTY** for refreshable braille displays ([BRLTTY Documentation](https://brltty.app/)).

#### 3. macOS: VoiceOver via Terminal.app / iTerm2
* **Bridge:** Apple NSAccessibility protocol (`AXUIElement`, `AXTextMarkerRange`).
* **Mechanism:** Terminal.app exposes a visual grid of terminal lines.
* **VoiceOver Behavior:** VoiceOver struggles heavily with cursor-addressing CLI tools. When `\r` rewrites occur, VoiceOver often announces individual changed character fragments or drops notifications entirely. In interactive editors (like Vim) or nested tools (like tmux), VoiceOver users frequently report "lost cursor" errors ([AppleVis Terminal Discussion](https://www.applevis.com/)).
* **Specialized Alternative:** Blind developers on macOS frequently use **TDSR (Tyler's Design Screen Reader)** ([tspivey/tdsr on GitHub](https://github.com/tspivey/tdsr)), an open-source Python daemon that intercepts terminal PTY output directly and feeds clean audio to `speech-dispatcher` or macOS speech synthesis.

### 2.2 The Four Terminal Accessibility Failure Modes

1. **The Spinner Speech Storm:**
   Animated glyphs (e.g. `⠋ ⠙ ⠹ ⠸ ⠼ ⠴ ⠦ ⠧ ⠇ ⠏`) output every 80ms. Screen readers verbalize Unicode characters literally: *"Braille pattern dots 1 2, Braille pattern dots 2 3..."* This floods the speech buffer, completely silencing actual logs.
2. **The Progress Bar Buffer Thrashing:**
   Progress bars relying on `\r` generate dozens of line re-reads per second. Screen readers repeatedly announce percentage changes or read ASCII blocks (`[=====>  ] 32%`), making linear review impossible.
3. **The Silent Semantic Loss (Color-Only Status):**
   A test runner prints green text for pass and red text for fail without text tokens. Screen readers read only the test name, giving the user zero indication of whether the test passed or failed.
4. **The ASCII Graph Audio Sludge:**
   Terminal charts (sparklines, box plots, table borders like `┌───┬───┐`) are read character-by-character: *"Box drawings light down and right, box drawings light horizontal..."* This renders diagnostic output completely unintelligible.

---

## 3. Existing Solutions & Ecosystem Precedents

Several major open-source tools and specifications have evolved to address terminal accessibility and color control.

### 3.1 `accessible-pygments` (Quansight Labs)
* **Repository:** [Quansight-Labs/accessible-pygments on GitHub](https://github.com/Quansight-Labs/accessible-pygments)  
* **PyPI:** `pip install accessible-pygments`  
* **Architecture:** Provides syntax-highlighting themes for Pygments designed to comply strictly with **WCAG 2.1 AA/AAA contrast ratios** (minimum 4.5:1 for normal text, 7:1 for AAA) and optimized for color-blindness (deuteranopia, protanopia, tritanopia).
* **Key Presets:** `a11y-light`, `a11y-dark`, `a11y-high-contrast-light`, `a11y-high-contrast-dark`.
* **Adoption:** Adopted by the Jupyter project and PyData Sphinx Theme for accessible code documentation.

### 3.2 The NO_COLOR Standard (no-color.org)
* **Origin:** Authored in 2017 by Lucas Werkmeister ([no-color.org](https://no-color.org/)).
* **Specification:**
  > *"Command-line software which accepts the `NO_COLOR` environment variable should, when the variable is present and not empty (regardless of its value), prevent the addition of ANSI color."*
* **Critical Implementation Detail:**
  * If `NO_COLOR` is present and contains *any* character (even `NO_COLOR=0` or `NO_COLOR=false`), color **must be disabled**.
  * Color is enabled *only* if `NO_COLOR` is completely unset or set to an empty string (`NO_COLOR=""`).
* **Adoption:** Supported by `ripgrep`, `curl`, `cmake`, `pip`, `deno`, `rustc`, `pytest`, `gh`, and hundreds of others.

### 3.3 Python `rich` Accessibility Architecture (Textualize)
* **Author:** Will McGugan ([Rich Documentation](https://rich.readthedocs.io/)).
* **Configuration for Accessibility:**
  ```python
  from rich.console import Console
  from rich.progress import Progress

  # Accessible Console Configuration
  console = Console(
      no_color=True,          # Disables ANSI color codes (respects NO_COLOR automatically)
      force_terminal=False,   # Prevents terminal control sequence injection
      color_system=None,      # Completely disables 3-bit, 8-bit, and 24-bit color
      highlight=False,        # Disables auto-regex syntax highlighting of numbers/paths
      legacy_windows=False    # Avoids old Win32 Console API bugs
  )

  # Disabling animated progress bars for screen readers
  progress = Progress(disable=True, console=console)
  ```
* **Textual Framework Direction:** The Textual TUI engine internally structures UI as a DOM tree and has a browser-based deployment roadmap specifically to leverage web accessibility APIs (HTML/ARIA) where native terminal emulators fail ([Textual Accessibility Roadmap](https://textual.textualize.io/)).

### 3.4 GitHub CLI (`gh a11y`)
* **Reference:** [GitHub Engineering Blog: Building a more accessible GitHub CLI (May 2025)](https://github.blog/engineering/accessibility/building-a-more-accessible-github-cli/)
* **Architecture:** GitHub integrated the `charmbracelet/huh` prompting library to revamp command-line prompts for screen reader compatibility.
* **Mechanisms:**
  * Running `gh a11y` exposes accessibility settings.
  * `gh config set accessible_prompter enabled` replaces visual arrow-key selection menus with simple numbered prompts (e.g., `1) Submit, 2) Cancel: Enter number:`), eliminating dynamic redraws.
  * Restricts palettes to 4-bit ANSI colors with guaranteed contrast against black/white backgrounds.

### 3.5 Anthropic Claude Code CLI (`--ax-screen-reader`)
* **Reference:** [Claude Code Accessibility Documentation](https://docs.anthropic.com/en/docs/claude-code) (Introduced v2.1.181).
* **Flags & Variables:**
  * Flag: `claude --ax-screen-reader`
  * Environment variable: `CLAUDE_AX_SCREEN_READER=1`
  * Settings file: `~/.claude/settings.json` (`"axScreenReader": true`)
* **Behaviors Activated in Screen Reader Mode:**
  1. **Linearized Transcripts:** Disables all in-place line overwrites (`\r`). All conversation turns and tool calls are appended sequentially.
  2. **Semantic Role Tagging:** Messages are prefixed with explicit textual labels: `you:`, `claude:`, `tool:`, `Permission Required:`.
  3. **Numbered Choice Prompts:** Converts arrow-key navigation menus into accessible numbered lists typed by number.
  4. **Audible Completion Chimes:** Rings the terminal bell (`\a` / ASCII `0x07`) when long-running tool calls finish or when user input is required.

---

## 4. TTY Detection vs. Screen Reader Auto-Detection

### 4.1 TTY vs. Piped Stream Detection

Command-line tools must automatically determine whether standard output is connected to an interactive display or redirected into a pipe, file, or subshell.

#### Python Implementation
```python
import os
import sys

def is_interactive_tty() -> bool:
    """Check if stdout is connected to an active, interactive terminal."""
    return hasattr(sys.stdout, "isatty") and sys.stdout.isatty()

def get_stream_capabilities():
    return {
        "stdin_isatty": sys.stdin.isatty() if hasattr(sys.stdin, "isatty") else False,
        "stdout_isatty": sys.stdout.isatty() if hasattr(sys.stdout, "isatty") else False,
        "stderr_isatty": sys.stderr.isatty() if hasattr(sys.stderr, "isatty") else False,
    }
```

#### Node.js Implementation
```javascript
const tty = require('node:tty');

function isInteractiveTTY() {
    // process.stdout.isTTY is boolean true if TTY, undefined if piped or redirected
    return Boolean(process.stdout && process.stdout.isTTY);
}
```

### 4.2 The Screen Reader Auto-Detection Dilemma

**In Web Browsers:** The W3C and browser vendors deliberately prohibit web applications from querying whether a screen reader is active, protecting users from invasive fingerprinting, tracking, and discriminatory degradation ([W3C Accessibility Privacy Guidelines](https://www.w3.org/TR/privacy-principles/)).

**In Local CLI Tools:** Because CLI tools run with local user privileges directly on the host OS, detecting accessibility configurations is technically feasible. However, no single cross-platform API exists.

#### Environment Variables (Highest Reliability & Universal Standard)
The most robust approach is checking established environment variables in priority order:
1. `SCREEN_READER=1` or `SCREEN_READER=true` (Informal community convention)
2. `CLAUDE_AX_SCREEN_READER=1`
3. `ACCESSIBILITY_ENABLED=1`
4. `TERM=dumb` (Legacy signal indicating terminal cannot handle escape sequences or cursor movement)

#### Platform-Specific OS API Detection Methods

```python
import os
import sys
import subprocess

def detect_screen_reader() -> bool:
    """
    Cross-platform heuristic for detecting active screen reader environments.
    Checks environment variables first, then queries OS accessibility subsystems.
    """
    # 1. Check explicit environment overrides
    env_vars = ["SCREEN_READER", "CLAUDE_AX_SCREEN_READER", "ACCESSIBILITY_ENABLED"]
    for var in env_vars:
        val = os.environ.get(var, "").strip().lower()
        if val in ("1", "true", "yes", "on"):
            return True

    if os.environ.get("TERM") == "dumb":
        return True

    # 2. Windows: SystemParametersInfoW SPI_GETSCREENREADER (0x0046)
    if sys.platform == "win32":
        try:
            import ctypes
            from ctypes import wintypes
            SPI_GETSCREENREADER = 0x0046
            is_running = wintypes.BOOL()
            result = ctypes.windll.user32.SystemParametersInfoW(
                SPI_GETSCREENREADER, 0, ctypes.byref(is_running), 0
            )
            if result and bool(is_running.value):
                return True
        except Exception:
            pass

    # 3. macOS: Check VoiceOver preference state via defaults
    elif sys.platform == "darwin":
        try:
            cmd = ["defaults", "read", "com.apple.universalaccess", "voiceOverOnOffKey"]
            output = subprocess.check_output(cmd, stderr=subprocess.DEVNULL).decode().strip()
            if output == "1":
                return True
        except Exception:
            pass

    # 4. Linux: Query GNOME Desktop Accessibility GSettings & Orca Process
    elif sys.platform.startswith("linux"):
        try:
            cmd = ["gsettings", "get", "org.gnome.desktop.a11y.applications", "screen-reader-enabled"]
            output = subprocess.check_output(cmd, stderr=subprocess.DEVNULL).decode().strip()
            if output == "true":
                return True
        except Exception:
            pass

    return False
```

---

## 5. WCAG & International Standards Applied to CLIs

While the **Web Content Accessibility Guidelines (WCAG 2.1/2.2)** were written primarily for web documents, their underlying principles apply directly to non-web software under international procurement standards.

### 5.1 Authoritative Non-Web Standards
* **W3C WCAG2ICT:** *"Guidance on Applying WCAG 2 to Non-Web Information and Communications Technologies"* ([W3C WCAG2ICT Group Note](https://www.w3.org/TR/wcag2ict/)). Provides specific guidance on interpreting WCAG success criteria for command-line interfaces and terminal environments.
* **US Section 508 Chapter 5 (Software):** Mandates that all federal agency software (including administrative CLIs) satisfy accessibility criteria ([U.S. Access Board Section 508 Standards](https://www.access-board.gov/ict/#502-interoperability-with-assistive-technology)).
* **EN 301 549 (Clause 11 — Software):** European standard mandating accessibility requirements for public sector software procurement.

### 5.2 Mapping WCAG Success Criteria to Command-Line Interfaces

| WCAG SC | Criterion Name | Level | Terminal / CLI Requirement | Anti-Pattern to Avoid |
| :--- | :--- | :--- | :--- | :--- |
| **1.1.1** | Non-text Content | A | All non-text content (ASCII charts, braille spinners, Unicode icons) must have an equivalent text label or alternative. | Printing `[==>  ]` without numeric `45%` or printing raw box art. |
| **1.3.1** | Info and Relationships | A | Output structure must be programmatically determinable. Use clear headers, key-value delimiters, and machine-readable output formats (`--json`). | Relying solely on spatial indentation or multi-column alignment without delimiters. |
| **1.3.2** | Meaningful Sequence | A | Output must follow a predictable, linear reading order. | Using `\r` and cursor-up sequences (`CSI n A`) to overwrite previously printed lines. |
| **1.4.1** | Use of Color | A | Color cannot be the sole vehicle for communicating status, errors, or required actions. | Outputting a red dot without the word `[FAILED]` or a green dot without `[PASSED]`. |
| **1.4.3** | Contrast (Minimum) | AA | Text and terminal backgrounds must maintain at least **4.5:1** contrast (3:1 for large text). | Dim gray text (`#666666`) on black (`#000000`) = 2.8:1 contrast (Failure). |
| **1.4.6** | Contrast (Enhanced) | AAA | Text and terminal backgrounds maintain at least **7:1** contrast. | Low-luminance color pairs. |
| **2.1.1** | Keyboard | A | All CLI prompts, pagination, and menus must be operable via standard keyboard keystrokes without requiring mouse clicks. | Requiring mouse selection in terminal multiplexers or terminals with mouse tracking. |
| **2.2.2** | Pause, Stop, Hide | A | Any moving, blinking, or auto-updating information must provide a mechanism to pause, stop, or disable it. | Unstoppable animated spinners or flashing progress bars. |
| **2.3.1** | Three Flashes or Below | A | Terminal output must never flash or blink text faster than 3 times per second. | Emitting SGR `5` (slow blink) or SGR `6` (rapid blink). |
| **3.3.2** | Labels or Instructions | A | User input prompts must provide clear instructions and defaults (e.g., `[y/N]`, `(1-5)`). | A blinking bare cursor with no prompt label. |
| **4.1.3** | Status Messages | AA | Completion states and long-running job finishes must be determinable via explicit text lines and audible signals (`\a`). | A CLI command finishing silently with no concluding summary line. |

---

## 6. Amber Monochrome Emulation & Photophysiology

The `--photophobia` (`--soft`) flag replaces harsh neon syntax highlighting with a calibrated monochrome amber phosphor scheme.

```
Visible Spectrum & Photophobia Sensitivity:
400nm             480nm-500nm (ipRGC Peak Spike)       580nm-590nm (P3/P134 Amber)      700nm
 [-----------------------|===================================|----------------------------]
 Blue/Cyan: High Rayleigh Scatter,                   Amber: Focuses on Fovea,
 ipRGC Stimulus, Migraine Pain                       Zero ipRGC Stimulus, Anti-Halation
```

### 6.1 Historical Amber CRT Hardware Specifications
During the 1980s, commercial terminal manufacturers transitioned from white (P4 phosphor) and green (P31 phosphor, ~525nm) to amber monochrome monitors (DEC VT220, IBM 3178/3151, Wyse 50/60) based on ergonomic studies conducted in Germany (DIN 66234 standard for display workplaces).

* **Phosphor Chemistry:** Standardized as **P3** (Zinc Sulfide:Cadmium Sulfide, ZnS:CdS:Ag) and later **P134** ([Phosphor Technology CRT Data](https://www.phosphor-technology.com/)).
* **Dominant Emission Wavelength:** Narrow band centered between **585 nm and 590 nm** (yellow-orange amber spectrum).
* **Decay Time:** Medium-long persistence (~20–40ms), which eliminated perceived 50Hz/60Hz cathode tube flicker.

### 6.2 Photophysiological & Ophthalmological Mechanisms

#### 1. Intraocular Rayleigh Scattering & Veiling Glare
Light traversing the human ocular media (cornea, crystalline lens, vitreous body) scatters inversely proportional to the fourth power of the wavelength:
$$I_{\text{scatter}} \propto \frac{1}{\lambda^4}$$

Comparing standard blue terminal accent text ($\lambda = 450\,\text{nm}$) against amber phosphor text ($\lambda = 590\,\text{nm}$):
$$\frac{I_{\text{blue}}}{I_{\text{amber}}} = \left(\frac{590}{450}\right)^4 = (1.311)^4 \approx 2.95$$
**Result:** Blue light scatters **295% more intensely** inside the eyeball than 590nm amber light. This intraocular scattering causes "veiling glare" and perceived blue haze, forcing the ciliary muscles to constantly micro-adjust focus.

#### 2. Chromatic Aberration & The "Blue Myopia" Effect
The human eye has an uncorrected chromatic dispersion of approximately 2.0 diopters across the visible spectrum. Short blue wavelengths refract more sharply than long wavelengths, focusing in front of the retina. To bring blue text into focus, the eye must accommodate excessively. Amber light (~590nm) focuses naturally directly onto the foveal receptor layer with minimum accommodation effort.

#### 3. Suppression of ipRGC Activation & Migraine Pathways
Intrinsically photosensitive Retinal Ganglion Cells (ipRGCs) contain the photopigment **melanopsin**, whose spectral sensitivity peaks sharply at **480 nm** ([Berson et al., Science 2002](https://www.science.org/doi/10.1126/science.1067262)). 
* Clinical research by Dr. Rami Burstein at Harvard Medical School ([Brain 2016](https://academic.oup.com/brain/article/139/7/1971/2468798)) demonstrated that blue light directly exacerbates migraine photophobia via retinal-thalamic projections.
* Light at **585–590 nm** generates negligible melanopic flux, preventing the activation of ipRGC-mediated trigeminal pain pathways.

#### 4. Halation Elimination for Astigmatism & Low Vision
In users with astigmatism, keratoconus, or cataracts, high-contrast white text (`#FFFFFF`) on pitch black (`#000000`) creates optical **halation**—a bleeding aura or double-contour fringe caused by irregular corneal refraction. Muting the foreground to warm amber (`#FFB000`) and softening the background to deep slate/charcoal (`#120E04`) preserves crisp perceptual contrast (APCA $L_c > 75$) while completely eliminating halation bleed.

### 6.3 Digital Amber Color Palette Specifications

For `--photophobia` mode, the enhancement layer defines three tiers of color mapping depending on terminal capabilities:

#### 1. 24-Bit TrueColor Palette (Direct Hex)
* **Primary Foreground (P3 Phosphor):** `#FFB000` (RGB: `255, 176, 0`) — Peak 588nm emulation.
* **Bright / Highlight Foreground:** `#FFCC00` (RGB: `255, 204, 0`) — Emphasis / headers.
* **Muted / Secondary Foreground:** `#C47D00` (RGB: `196, 125, 0`) — Timestamps / secondary data.
* **Background (Anti-Halation Charcoal):** `#120E04` (RGB: `18, 14, 4`) — Deep warm black, avoids 0-luminance optical shock.
* **Alternative "Oatmeal Soft Contrast":** `#D6D0C4` on `#141416` (Low stimulus paper-reading tone).

#### 2. 256-Color Palette Mapping (For macOS Terminal.app)
* **Primary Amber:** Color index `214` (`#FFAF00`)
* **Bright Amber:** Color index `220` (`#FFD700`)
* **Dim / Muted Amber:** Color index `172` (`#D78700`) or `130` (`#AF5F00`)
* **Background Slate:** Color index `234` (`#1C1C1C`) or `232` (`#080808`)

#### 3. 16-Color ANSI Mapping
* **Primary Amber:** Standard ANSI Yellow (`33`)
* **Bright Amber:** High-intensity Bright Yellow (`93`)
* **Background:** Standard Default Background (`49`)

---

## 7. Cross-Platform Terminal Color Detection

Determining whether a terminal supports 16 colors, 256 colors, or 24-bit TrueColor requires navigating a historical patchwork of platform implementations.

### 7.1 Platform Peculiarities

* **Windows (ConHost vs. Windows Terminal):**
  * *Legacy ConHost (Windows 7/8):* Supported only 16 console colors via `SetConsoleTextAttribute`.
  * *Windows 10 (Build 10586+):* Added native ANSI/VT processing. Applications must call `SetConsoleMode` with `ENABLE_VIRTUAL_TERMINAL_PROCESSING` (`0x0004`).
  * *Modern Windows Terminal:* Sets `WT_SESSION` in environment; natively supports full 24-bit TrueColor.
* **macOS Terminal.app:**
  * Sets `TERM_PROGRAM="Apple_Terminal"`.
  * **Critical Limitation:** Apple's native Terminal.app **only supports up to 256 colors**. It does *not* support 24-bit TrueColor. Emitting `\x1b[38;2;r;g;bm` in Terminal.app produces either stripped output, color corruption, or ignored escapes.
  * Modern alternatives (iTerm2, Kitty, Alacritty, WezTerm, Ghostty) support TrueColor and set `COLORTERM="truecolor"`.
* **Linux (xterm, VTE, Console):**
  * VTE-based terminals (GNOME Terminal, Tilix) set `COLORTERM="truecolor"` and fully support 24-bit color.
  * Linux Virtual Terminals (`/dev/tty1`–`/dev/tty6`) support only standard 16-color ANSI (`TERM="linux"`).

### 7.2 Portable Color Depth Resolution Algorithm

Following the conventions established by `supports-color` ([chalk/supports-color](https://github.com/chalk/supports-color)), `termcolor`, and `rich`:

```
                    +------------------------------------+
                    |  Check CLI Flags: --no-color, etc. |
                    +------------------------------------+
                                      |
                    +------------------------------------+
                    |  Is NO_COLOR set & non-empty?      | ---- Yes ---> [ Level 0: None ]
                    +------------------------------------+
                                      | No
                    +------------------------------------+
                    |  Is stdout a TTY (isatty())?       | ---- No ----> [ Level 0: None ]
                    +------------------------------------+
                                      | Yes
                    +------------------------------------+
                    |  Is FORCE_COLOR set?               | ---- Yes ---> [ Level 1, 2, or 3 ]
                    +------------------------------------+
                                      | No
                    +------------------------------------+
                    |  COLORTERM == "truecolor" | "24bit"| ---- Yes ---> [ Level 3: TrueColor ]
                    +------------------------------------+
                                      | No
                    +------------------------------------+
                    |  TERM_PROGRAM == "Apple_Terminal"? | ---- Yes ---> [ Level 2: 256-Color ]
                    +------------------------------------+
                                      | No
                    +------------------------------------+
                    |  TERM contains "256color"?         | ---- Yes ---> [ Level 2: 256-Color ]
                    +------------------------------------+
                                      | No
                    +------------------------------------+
                    |  TERM matches basic vt100/xterm    | ---- Yes ---> [ Level 1: 16-Color ]
                    +------------------------------------+
                                      | Otherwise
                                      v
                             [ Level 0: No Color ]
```

#### Production Python Color Capability Resolver
```python
import os
import sys

class ColorLevel:
    NONE = 0        # Plain text
    ANSI_16 = 1     # 3-bit / 4-bit standard ANSI
    EXTENDED_256 = 2# 8-bit palette
    TRUECOLOR = 3   # 24-bit direct RGB

def detect_color_support() -> int:
    """Portably detect terminal color capabilities."""
    # 1. Respect explicit NO_COLOR standard
    no_color = os.environ.get("NO_COLOR")
    if no_color is not None and len(no_color) > 0:
        return ColorLevel.NONE

    # 2. Check FORCE_COLOR override
    force_color = os.environ.get("FORCE_COLOR")
    if force_color is not None:
        if force_color in ("0", "false", "none"):
            return ColorLevel.NONE
        if force_color in ("1", "true"):
            return ColorLevel.ANSI_16
        if force_color == "2":
            return ColorLevel.EXTENDED_256
        if force_color == "3":
            return ColorLevel.TRUECOLOR

    # 3. Stream verification: must be a TTY unless explicitly forced
    if not (hasattr(sys.stdout, "isatty") and sys.stdout.isatty()):
        return ColorLevel.NONE

    # 4. Check for dumb terminals
    term = os.environ.get("TERM", "").lower()
    if term == "dumb":
        return ColorLevel.NONE

    # 5. Windows platform handling
    if sys.platform == "win32":
        # Windows Terminal natively supports TrueColor
        if "WT_SESSION" in os.environ:
            return ColorLevel.TRUECOLOR
        # Enable Virtual Terminal Processing on Windows 10+
        try:
            import ctypes
            kernel32 = ctypes.windll.kernel32
            STD_OUTPUT_HANDLE = -11
            ENABLE_VIRTUAL_TERMINAL_PROCESSING = 0x0004
            handle = kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
            mode = ctypes.c_ulong()
            if kernel32.GetConsoleMode(handle, ctypes.byref(mode)):
                if kernel32.SetConsoleMode(handle, mode.value | ENABLE_VIRTUAL_TERMINAL_PROCESSING):
                    return ColorLevel.TRUECOLOR
        except Exception:
            pass
        return ColorLevel.ANSI_16

    # 6. Check COLORTERM environment variable for TrueColor
    colorterm = os.environ.get("COLORTERM", "").lower()
    if colorterm in ("truecolor", "24bit"):
        return ColorLevel.TRUECOLOR

    # 7. macOS Terminal.app check (strictly 256 colors maximum)
    if os.environ.get("TERM_PROGRAM") == "Apple_Terminal":
        return ColorLevel.EXTENDED_256

    # 8. Check TERM string heuristics
    if "256color" in term:
        return ColorLevel.EXTENDED_256
    if any(prefix in term for prefix in ("xterm", "vt100", "screen", "ansi", "linux")):
        return ColorLevel.ANSI_16

    return ColorLevel.NONE
```

---

## 8. Environmental Color Standards: NO_COLOR, COLORTERM & CLICOLOR

A rigorous CLI tool must properly negotiate conflicting environment variables.

### 8.1 Standards Summary

| Variable | Governing Authority | Canonical Values | Semantics |
| :--- | :--- | :--- | :--- |
| `NO_COLOR` | Community ([no-color.org](https://no-color.org/)) | Any non-empty string | Disables all ANSI color output. |
| `COLORTERM` | Terminal Emulator De Facto | `truecolor`, `24bit` | Signals that the terminal understands direct 24-bit SGR sequences. |
| `FORCE_COLOR` | JS/Chalk Ecosystem | `0`, `1`, `2`, `3` | Overrides TTY checks to force specific color depths. |
| `CLICOLOR` | BSD / Apple ([bixense.com](https://bixense.com/clicolors/)) | `0` or `1` | `0`: disable color. `1`: enable color if stdout is a TTY. |
| `CLICOLOR_FORCE`| BSD / Apple ([bixense.com](https://bixense.com/clicolors/)) | Non-zero | Forces color output even if stdout is piped. |

### 8.2 Precedence Order for Project 4
When evaluating whether to output color and which palette to use, follow this strict precedence hierarchy:
1. **Explicit CLI Flags (Absolute Authority):**
   * `--no-color` or `--screen-reader`: Force `ColorLevel.NONE`.
   * `--photophobia`: Force amber monochrome palette.
   * `--color`: Force color enabled.
2. **`NO_COLOR` Variable:** If set and non-empty, immediately force `ColorLevel.NONE`.
3. **`CLICOLOR_FORCE` / `FORCE_COLOR`:** If set, override TTY checks.
4. **`CLICOLOR=0`:** If set, disable color.
5. **TTY Verification (`sys.stdout.isatty()`):** If false, disable color.
6. **Capability Negotiation (`COLORTERM`, `TERM_PROGRAM`, `TERM`):** Resolve to `ANSI_16`, `EXTENDED_256`, or `TRUECOLOR`.

---

## 9. Existing CLI Accessibility Wrappers & Tooling

While testing web accessibility is mature (via `axe-core` and `pa11y`), accessibility tooling built specifically *for* CLI binaries is emerging.

### 9.1 Developer & Testing Tools
* **`term-a11y` & `term-a11y-lint` (npm: `zaydea805/term-a11y`):**
  * Open-source JavaScript library providing accessible drop-in replacements for `ora`, `cli-progress`, and `cli-table3`.
  * Detects non-interactive and assistive environments; automatically renders linear text instead of animated frames.
  * Includes a static linter (`term-a11y-lint`) that scans codebases for CLI accessibility anti-patterns (e.g., raw cursor movement codes or inaccessible progress bars).
* **`ansifilter` ([GitLab: saalen/ansifilter](https://gitlab.com/saalen/ansifilter)):**
  * Production utility by André Simon for stripping ANSI escape sequences from standard input streams or converting them to plain text, HTML, or LaTeX.
* **`strip-ansi` (npm) & `strip-ansi-escapes` (Rust crate):**
  * De facto ecosystem libraries for programmatic regex-based ANSI removal.
* **`Guidepup` ([guidepup.dev](https://www.guidepup.dev/)):**
  * Automated testing library for driving real screen readers (VoiceOver on macOS, NVDA on Windows) programmatically in test suites and terminal environments.
* **`axe DevTools CLI` & `Pa11y`:**
  * While command-line tools themselves, their target of evaluation is web documents (HTML/DOM), not CLI terminal streams.

---

## 10. Proposed Architecture for Project 4 (CLI Accessibility Layer)

Based on these research findings, the proposed terminal accessibility layer should be implemented as an intercepting stream filter or wrapper class.

```
+-------------------------------------------------------------------------+
|                        CLI Core Logic & Diagnostics                     |
+-------------------------------------------------------------------------+
                                    |
                    emit(event, text, status, progress)
                                    v
+-------------------------------------------------------------------------+
|                 Accessibility & Ergonomic Transform Layer               |
|                                                                         |
|  Mode: --screen-reader                Mode: --photophobia               |
|  - Strip ANSI & cursor escapes        - Intercept RGB / 16-color codes  |
|  - Suppress \r & spinners             - Map to Amber (590nm: #FFB000)   |
|  - Linearize progress (at 25% jumps)  - Soften BG to Charcoal (#120E04)|
|  - Add [PASS]/[FAIL] tokens           - Suppress high-energy blue spikes|
|  - Ring terminal bell (\a) on done    - Ensure contrast ratio > 7:1     |
+-------------------------------------------------------------------------+
                                    |
                             Filtered Stream
                                    v
                          sys.stdout (Terminal)
```

### 10.1 Complete Python Reference Implementation (`cli_a11y.py`)

```python
"""
cli_a11y.py - Terminal Accessibility & Photophobia Enhancement Engine
Project 4: Low-Vision, Screen Reader, and Sensory Ergonomics Layer
"""

import os
import sys
import re
from typing import Optional

# ANSI Escape Sequence Regex (ECMA-48 / ISO 6429 parser)
ANSI_STRIP_REGEX = re.compile(
    r"""
    \x1B
    (?:
        [@-Z\\-_]
    |
        \[ [0-?]* [ -/]* [@-~]
    |
        \] .*? (?:\x07|\x1B\\)
    )
    """,
    re.VERBOSE
)

class TerminalAccessibilityEngine:
    def __init__(
        self,
        screen_reader: bool = False,
        photophobia: bool = False,
        no_color: bool = False
    ):
        # Auto-detect if flags were not explicitly provided
        self.screen_reader = screen_reader or self._detect_screen_reader()
        self.photophobia = photophobia
        self.no_color = no_color or self._detect_no_color()
        self.color_level = self._detect_color_level()

        # Amber Palette Definitions (~590nm P3/P134 Phosphor Emulation)
        self.AMBER_PRIMARY_TRUECOLOR = "\x1b[38;2;255;176;0m"    # #FFB000 (Peak 588nm)
        self.AMBER_BRIGHT_TRUECOLOR  = "\x1b[38;2;255;204;0m"    # #FFCC00 (Headers/Emphasis)
        self.AMBER_MUTED_TRUECOLOR   = "\x1b[38;2;196;125;0m"    # #C47D00 (Secondary/Dim)
        self.AMBER_256               = "\x1b[38;5;214m"          # xterm index 214 (#FFAF00)
        self.AMBER_ANSI              = "\x1b[33m"                # ANSI Yellow
        self.RESET                   = "\x1b[0m"

        # Screen Reader State Tracking
        self._last_progress_pct = -1

    def _detect_no_color(self) -> bool:
        val = os.environ.get("NO_COLOR")
        return val is not None and len(val) > 0

    def _detect_screen_reader(self) -> bool:
        env_vars = ["SCREEN_READER", "CLAUDE_AX_SCREEN_READER", "ACCESSIBILITY_ENABLED"]
        if any(os.environ.get(v, "").lower() in ("1", "true", "yes") for v in env_vars):
            return True
        if os.environ.get("TERM") == "dumb":
            return True
        return False

    def _detect_color_level(self) -> int:
        if self.no_color or self.screen_reader:
            return 0
        if not (hasattr(sys.stdout, "isatty") and sys.stdout.isatty()):
            return 0
        if os.environ.get("COLORTERM") in ("truecolor", "24bit"):
            return 3
        if os.environ.get("TERM_PROGRAM") == "Apple_Terminal":
            return 2
        term = os.environ.get("TERM", "").lower()
        if "256color" in term:
            return 2
        return 1

    def style_text(self, text: str, role: str = "normal") -> str:
        """Apply photophobia amber styling or strip color entirely."""
        if self.screen_reader or self.no_color or self.color_level == 0:
            return ANSI_STRIP_REGEX.sub("", text)

        if self.photophobia:
            clean_text = ANSI_STRIP_REGEX.sub("", text)
            if self.color_level == 3:
                prefix = self.AMBER_BRIGHT_TRUECOLOR if role == "bold" else (
                    self.AMBER_MUTED_TRUECOLOR if role == "dim" else self.AMBER_PRIMARY_TRUECOLOR
                )
            elif self.color_level == 2:
                prefix = self.AMBER_256
            else:
                prefix = self.AMBER_ANSI
            return f"{prefix}{clean_text}{self.RESET}"

        return text

    def print_line(self, text: str, role: str = "normal") -> None:
        """Emit a distinct line of output with proper semantic treatment."""
        formatted = self.style_text(text, role=role)
        sys.stdout.write(formatted + "\n")
        sys.stdout.flush()

    def print_status(self, label: str, message: str, is_error: bool = False) -> None:
        """
        Output status messages ensuring WCAG 1.4.1 compliance:
        Never rely solely on color; explicitly embed [SUCCESS], [FAILURE], or [INFO].
        """
        tag = "[FAILURE]" if is_error else f"[{label.upper()}]"
        role = "bold" if is_error else "normal"
        line = f"{tag} {message}"
        self.print_line(line, role=role)

    def print_progress(self, current: int, total: int, prefix: str = "Progress") -> None:
        """
        Accessible progress indicator.
        - Under visual mode: Can output in-place bar or overwrite.
        - Under --screen-reader: Suppresses \\r and prints linear checkpoints only at 25% jumps.
        """
        pct = int((current / total) * 100) if total > 0 else 0

        if self.screen_reader:
            # Emit only at major milestones to prevent speech queue flooding
            milestones = (0, 25, 50, 75, 100)
            if pct in milestones and pct != self._last_progress_pct:
                self._last_progress_pct = pct
                sys.stdout.write(f"{prefix}: {pct}% complete.\n")
                sys.stdout.flush()
        else:
            # Standard visual overwrite using \r
            bar_len = 20
            filled = int(bar_len * (current / total)) if total > 0 else 0
            bar = "=" * filled + " " * (bar_len - filled)
            line = f"\r{prefix}: [{bar}] {pct}%"
            sys.stdout.write(self.style_text(line))
            if current >= total:
                sys.stdout.write("\n")
            sys.stdout.flush()

    def notify_complete(self, message: str = "Operation complete.") -> None:
        """Signals completion. Rings audible terminal bell (\\a) in screen reader mode."""
        if self.screen_reader:
            # Emit audible BEL (WCAG 4.1.3 status announcement)
            sys.stdout.write("\a")
        self.print_status("DONE", message)
```

---

## 11. Primary Source Citations & References

1. **Ecma International:** *Standard ECMA-48: Control Functions for Coded Character Sets*, 5th Edition (June 1991).  
   URL: https://www.ecma-international.org/publications-and-standards/standards/ecma-48/
2. **International Telecommunication Union (ITU):** *ITU-T Recommendation T.416 (1993) | ISO/IEC 8613-6: Open Document Architecture (ODA) and Interchange Format - Character Content Architectures*.  
   URL: https://www.itu.int/rec/T-REC-T.416-199303-I/en
3. **World Wide Web Consortium (W3C):** *Guidance on Applying WCAG 2 to Non-Web Information and Communications Technologies (WCAG2ICT)*, W3C Working Group Note.  
   URL: https://www.w3.org/TR/wcag2ict/
4. **United States Access Board:** *Revised Section 508 Standards and Section 255 Guidelines for Information and Communication Technology (ICT)*, Chapter 5: Software.  
   URL: https://www.access-board.gov/ict/#502-interoperability-with-assistive-technology
5. **Burstein, R., Noseda, R., & Fulton, A. B. (2016):** *Migraine photophobia originating in cone-driven retinal pathways*, *Brain*, Volume 139, Issue 7, Pages 1971–1986.  
   URL: https://academic.oup.com/brain/article/139/7/1971/2468798
6. **Berson, D. M., Dunn, F. A., & Takao, M. (2002):** *Phototransduction by Retinal Ganglion Cells That Set the Circadian Clock*, *Science*, 295(5557), 1070–1073.  
   URL: https://www.science.org/doi/10.1126/science.1067262
7. **Werkmeister, L. (2017):** *NO_COLOR: An informal standard for disabling ANSI color in command-line tools*.  
   URL: https://no-color.org/
8. **Schubert, C. (bixense.com):** *CLICOLOR & CLICOLOR_FORCE Standard Specification*.  
   URL: https://bixense.com/clicolors/
9. **Hecht, R., & Feller, A. (GitHub, May 2, 2025):** *Building a more accessible GitHub CLI*.  
   URL: https://github.blog/engineering/accessibility/building-a-more-accessible-github-cli/
10. **Anthropic PBC:** *Claude Code CLI Documentation — Accessibility and Screen Reader Mode (`--ax-screen-reader`)*.  
    URL: https://docs.anthropic.com/en/docs/claude-code
11. **Quansight Labs:** *accessible-pygments: WCAG 2.1 Compliant Syntax Highlighting Themes for Pygments*.  
    URL: https://github.com/Quansight-Labs/accessible-pygments
12. **McGugan, W. (Textualize):** *Rich: Python library for rich text and beautiful formatting in the terminal*.  
    URL: https://github.com/Textualize/rich
13. **Microsoft Corporation:** *Windows Terminal & Console Accessibility Architecture (UI Automation Provider)*.  
    URL: https://learn.microsoft.com/en-us/windows/console/accessibility
14. **Simon, A.:** *Ansifilter: ANSI Terminal Escape Code Parser and Converter*, hosted on GitLab.  
    URL: https://gitlab.com/saalen/ansifilter
15. **Spivey, T.:** *TDSR: Terminal-based Screen Reader for macOS and Linux*.  
    URL: https://github.com/tspivey/tdsr
16. **Zayde, A.:** *term-a11y & term-a11y-lint: Accessible Terminal UI Components for Node.js*.  
    URL: https://github.com/zaydea805/term-a11y
17. **Sindre Sorhus et al.:** *supports-color: Detect whether a terminal supports color (npm package)*.  
    URL: https://github.com/chalk/supports-color
18. **Sindre Sorhus et al.:** *ansi-regex: Regular expression for matching ANSI escape codes*.  
    URL: https://github.com/chalk/ansi-regex
