# Voice Tour Backend Handoff

This repo contains a minimal “backend” page that mounts the ElevenLabs Conversational AI widget, plays an intro voice-over, and runs a looping seasonal soundscape bed.

> Project nickname: **Dijk van een Wijf — Voice Tour**

---

## Files

- `index_complete_functionality.html` — compact, minimal UI with all functionality wired (consent gate, language choice, audio graph, intro sequence, widget mount, Android hint).
- `index_demo_pre_figma.html` — same logic with a more polished temporary UI (wordmark, styled buttons, nicer consent modal, minor extras).

Both versions load the ElevenLabs widget **only after user consent** and support EN / NL / FR agents via data attributes on the language buttons.

---

## Quick start

This is a static build; any static host will work (Netlify, Vercel, S3, GitHub Pages). During development you can use a tiny HTTP server to avoid autoplay/cors quirks:

```bash
# Python 3
python -m http.server 8080

# or Node (if you have it)
npx http-server -p 8080
```

Open `http://localhost:8080/index_demo_pre_figma.html` (or the `index_complete_functionality.html` if you prefer the minimal version).

> **Note:** Ship the audio files alongside the HTML (see “assets”).

---

## External dependency

We use the ElevenLabs Conversational AI **web component**. It’s inserted dynamically **after consent**:

```html
<script src="https://unpkg.com/@elevenlabs/convai-widget-embed" async></script>
```

Mount point (created in JS):
```html
<elevenlabs-convai agent-id="AGENT_ID_HERE"></elevenlabs-convai>
```

---

## How it works (flow)

1. **Consent overlay (blocking)**
   - Full white overlay appears on page load.
   - “Yes” hides the overlay, unlocks scroll, and injects the ElevenLabs embed script.
   - “No” navigates back; in the demo page we fall back to a neutral URL if there’s no history.

2. **Language selection**
   - Buttons have `data-agent` (ElevenLabs agent id) and `data-lang` (`EN`, `NL`, `FR`). 
   - On click we:
     - Initialize a shared **Web Audio** graph (see next section).
     - Start the **seasonal background soundscape** based on current month.
     - Play **Intro, Part 1** (silent or minimal instructions VO per language).
     - (Non‑Android) pre‑warm mic permission with `getUserMedia({ audio: true })` to reduce iOS stutter later.
     - Show a **Skip Intro** button.

3. **Widget mount + Intro Part 2**
   - When Part 1 ends:
     - We briefly **duck** background and intro levels to mask route changes (avoids iOS “stutter” as WebRTC initializes).
     - Inject `<elevenlabs-convai>` with the chosen `agent-id`.
     - On Android: gently lift AI gain and show a one‑time hint about disabling Bluetooth “Phone calls” profile to avoid SCO/HFP robotic audio.
     - Start **Intro, Part 2** and do a timed dip/return on the background bed for a cinematic feel.

4. **Ongoing**
   - Background loops.
   - The widget runs inside an iframe. (The demo includes a light cosmetic `enhanceWidget()` that adds a blur/alpha to the pane and attempts to auto‑click a “Begin” button if present — harmless no‑op if the selectors change.)

---

## Audio architecture

- Two `<audio>` elements on the page:
  - `bgAudio` → seasonal bed (looping).
  - `introAudio` → both Part 1 and Part 2 voice‑overs.
- A single `AudioContext` with two `GainNode`s:
  - `bgGainNode` (bed mix level).
  - `aiGainNode` (intro/widget mix level).
- Helper functions:
  - `rampGain()` / `rampGainSlow()` for smooth fades.
  - `duckAll()` / `unduckAll()` to mask route changes.
- Platform tweaks:
  - `AI_VOLUME` is slightly higher on Android.
  - On Android, post‑mount we add a small extra lift to `aiGainNode` and optionally show a hint banner.

---

## Media assets

Expected relative file names (place in the same directory as the HTML or adjust the paths in the code):

```
# Intro, Part 1 (language specific; intentionally minimal)
DveW_Intro_EN_2 (silent intro).mp3
DveW_Intro_NL_2 (silent intro).mp3
DveW_Intro_FR_2 (silent intro).mp3

# Intro, Part 2 (language specific; the main intro VO)
Dijk van een Wijf - Introtext EN_1.2.mp3
Dijk van een Wijf - Introtext NL_1.2.mp3
Dijk van een Wijf - Introtext FR_1.2.mp3

# Seasonal beds (loop)
Soundscapes - Lente - Dijk van een Wijf V2.mp3
Soundscapes - Zommer - Dijk van een Wijf V2.mp3
Soundscapes - Herfst - Dijk van een Wijf V2.mp3
Soundscapes - Winter - Dijk van een Wijf V2.mp3
```

The current month picks the bed automatically: **Mar–May → Spring**, **Jun–Aug → Summer**, **Sep–Nov → Autumn**, otherwise **Winter**.

---

## Browser/platform notes

- iOS Safari requires user interaction before audio playback; we initialize the `AudioContext` and start audio only **after a button click**.
- **Bluetooth headsets on Android** can switch to the HFP/SCO telephony profile when the mic opens (robotic, narrowband). The hint suggests disabling the “Phone calls” profile or Bluetooth for cleaner audio.
- Autoplay with sound generally needs user gesture; do not remove the language click step.

---

## Minimal integration example

If you want to embed the widget elsewhere, at its simplest after consent you can do:

```html
<!-- After consent -->
<script src="https://unpkg.com/@elevenlabs/convai-widget-embed" async></script>

<div id="widgetContainer"></div>

<script>
  const container = document.getElementById('widgetContainer');
  // Replace with your agent id
  const agentId = 'agent_XXXXXXXXXXXXXXX';
  container.innerHTML = `<elevenlabs-convai agent-id="${agentId}"></elevenlabs-convai>`;
</script>
```

You’d lose the intro/bed choreography, but the component will function.

---


## Website installation


Install the dependencies:

```bash
npm install
```
<br/>

### Scripts

The following scripts are defined in package.json:

<br/>

**Build** – generate the CSS once:
```bash
npm run build
```
This compiles ./src/css/input.css into ./src/css/output.css

<br/>

**Dev** – development mode with file watching:
```bash
npm run dev
```
This keeps watching your files and recompiles on every change.




