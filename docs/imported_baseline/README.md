# Single-File Stereo Audio Visualizers (Canvas + Web Audio)

This folder is a proof-of-work argument: **HTML + CSS + JavaScript** is enough to build *real software*—interactive, real-time, audio-reactive, visually rich—while staying **fully portable** as a **single offline HTML file**.

No Node. No bundlers. No imported libraries. No cloud.  
Just the browser runtime that billions of devices already ship with.

Two implementations live here as **Example 1** and **Example 2**. They share the same core architecture; Example 2 adds richer interaction, preset sharing, debug tooling, and extra robustness.

---

## Why this exists

Modern dev culture often implies you must “earn the right to build” by adopting a framework stack and a dependency ecosystem. That’s gatekeeping disguised as “best practices.”

These examples are a counterexample.

They demonstrate that you can:
- ship something meaningful as a single file,
- understand the entire system end-to-end,
- run it offline,
- share it like a postcard,
- and still use powerful platform primitives (Canvas, Web Audio, Fullscreen, Clipboard, Touch, etc.).

If you can read and edit a text file, you can build.

---

## What it does

A stereo audio stream (microphone or audio file) is analyzed for **low-frequency energy** (“bass-ish” bins). That energy is smoothed and used to drive a generative visual:

- A set of **orbs** orbit around the center.
- Each orb draws a **particle field** (pseudo-3D sphere/spiral mapping).
- **Stereo matters**: left/right energy is mixed differently per orb based on its orbit angle.
- A lightweight “**BPM-ish**” peak detector produces a tempo-like signal used to scale particle density.

When no audio is active, an **idle/demo oscillator** keeps the scene alive.

---

## Quick start

### Offline (audio file mode)
1. Open **Example 1** or **Example 2** by double-clicking the HTML file.
2. Click **Use Audio File** and pick any audio file.

This mode works offline because it uses a local `<audio>` element.

### Microphone mode (requires secure context)
Browsers block microphone access on `file://`.

Run a local server in this folder and open the page via `http://localhost`:

- Python:
  - `python -m http.server 8000`
- Node (optional convenience, not required):
  - `npx serve`

Then open:
- `http://localhost:8000/<example>.html`

---

## Controls

### On-screen sliders (both examples)
- **Orbs**: number of orbiting emitters.
- **Particles min / max**: clamps particle density (which is derived from BPM-ish detection).
- **Trail**: persistence (0 = hard clear; higher = longer trails).
- **Orbit radius**: base orbit size.
- **Orbit energy**: how much bass inflates orbit radius.

### Buttons (both examples)
- **Stop Audio**: stops mic/file mode and returns to idle demo.
- **Show Overlay**: re-open onboarding/permission panel.

### Example 2 extras
- **Keyboard shortcuts**
  - `SPACE` pause visuals (audio can keep running)
  - `F` fullscreen
  - `R` reset to defaults
  - `D` debug / FPS display
  - `S` share settings (creates a URL with preset in the hash, copies when possible)
  - `?` help overlay
- **Touch gestures**
  - swipe up/down: adjust trail
  - swipe left/right: adjust orbit radius
  - pinch: change orb count
- **Preset sharing**
  - settings can be exported into the URL hash and reloaded on startup

---

## How it works (architecture)

### 1) Parameter discipline
- **DEFAULTS** holds immutable baseline settings.
- **PARAMS** is a deep copy used at runtime.
- UI updates mutate PARAMS.
- Example 2 supports **bi-directional** sync (PARAMS → UI) for presets and gestures.

### 2) Canvas sizing
- Canvas is sized to the `<main>` container.
- DPR scaling is applied so drawing uses **CSS pixels** while the backing store stays crisp.

### 3) Audio analysis (stereo)
The audio graph is intentionally minimal:

- Source feeds a **master analyser**
- Source feeds a **channel splitter**
- Splitter feeds **left/right analysers**
- File mode optionally routes audio to speakers via gain → destination

“Bass-ish energy” is computed by averaging a configurable range of low FFT bins.

### 4) Feature signals
- `leftEnergy`, `rightEnergy`, and `centerEnergy` are smoothed (exponential smoothing).
- A simple peak detector produces a “BPM-ish” estimate:
  - sliding window average
  - peak threshold multiplier
  - refractory time
  - EMA smoothing

### 5) Visual synthesis
Each frame:
- fade the canvas with a translucent fill (trails)
- switch to additive blending (“lighter”)
- place orbs on an energy-inflated orbit
- for each orb, compute energy from stereo mix by angle + a small shared center component
- render particle field using a parametric mapping and depth shaping

---

## Design constraints (intentional)
- **Single-file, no dependencies**
- **Readable and modifiable**
- **Tunable without spelunking**
- **Offline-capable**
- **Works even with no audio input**

---

## Known limitations / notes
- Microphone mode requires `https://` or `http://localhost`; `file://` will not work.
- The “BPM-ish” detector is a visual heuristic, not a musical BPM analyzer.
- Trail fade uses a black translucent fill; if background color is changed away from black, trail decay may still trend black.
- Performance scales with `orbs × particles`. Keep particle caps conservative on mobile.

---

## Version 2.2 idea (one concrete next step)
**Add a built-in “Recorder” mode that exports a short loop as a video (WebM) or GIF-like clip (WebM recommended).**

Why this is a great v2.2 step:
- stays zero-dependency and single-file
- turns the toy into a shareable artifact generator
- teaches an additional core web primitive (MediaRecorder + canvas capture)

---

## Four evolution paths (where this can go)

1. **Better audio features (still lightweight)**
   - add a mid/high band energy split
   - detect onsets more reliably (energy derivative + adaptive threshold)
   - separate “kick-like” vs “snare-like” events for different visual triggers

2. **Preset system as a first-class concept**
   - named presets in a small in-page list
   - import/export presets via JSON file
   - “randomize within safe bounds” button for generative exploration

3. **Visual engine modularity**
   - multiple “scenes” (orbit spirals, radial blooms, lattice fields, etc.)
   - hot-swap scene modules via a dropdown
   - shared audio/features layer feeding different renderers

4. **Accessibility and resilience**
   - reduced-motion mode (auto-detect `prefers-reduced-motion`)
   - explicit “low-power mode” (caps particles/orbs if FPS drops)
   - improved mobile UX (gesture hints + larger controls + safe defaults)

---

## Attribution / license

© Christopher “Kisuul” Lohman.  
If the parent repository defines a license, this folder is expected to inherit it unless overridden here.
