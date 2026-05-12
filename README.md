# 🎹 MidiControls

**Browser-based MIDI controller editor — no install, no backend, no build step.**  
Connect your USB MIDI hardware directly in Chrome or Edge, edit device presets visually,
send SysEx, monitor incoming messages, and back up everything to JSON.

---

## Screenshots

> _Add your own screenshot to `assets/screenshot.png` and it will appear here._

![MidiControls Editor](assets/screenshot.png)

---

## Quickstart

```bash
# Clone or download the project
git clone https://github.com/YOUR_USERNAME/midicontrols.git
cd midicontrols

# Start a local server (required — fetch() blocks on file://)
npx serve .
# or
python3 -m http.server 8080
# or
php -S localhost:8080
```

Open **http://localhost:8080** in Chrome or Edge.  
Plug in your MIDI device, select MIDI In/Out ports in the top bar — done.

> **Why a server?** Device templates are loaded via `fetch()` from `devices/*/device.json`.
> Browsers block `fetch()` on `file://` URLs for security reasons.
> Any static HTTP server works — even `npx serve .` in one terminal.

---

## Features

| Feature | Details |
|---|---|
| 🎛 **Visual Editor** | Pad / knob / fader / button grid with live CC value display |
| 📡 **MIDI Monitor** | Real-time log with Note / CC / PC / SysEx filters and pause |
| ⚡ **SysEx** | Send raw SysEx hex, raw CC/Note/PC, per-device quick commands |
| 💾 **Backup** | Export all presets to JSON, restore per-device or full dump |
| 🗂 **Device Manager** | Add, edit, remove, import and export device templates |
| 🎓 **MIDI Learn** | Click LEARN on any pad — press hardware key to auto-map |
| 🖱 **Knob Drag** | Drag knobs up/down to send live CC to MIDI Out |
| 🔌 **Auto-connect** | Port state changes detected automatically |
| 🔁 **MIDI Thru** | Optional echo of MIDI In to MIDI Out |
| 🎼 **BPM Detection** | Reads MIDI clock messages and displays live BPM |
| 📋 **CC Map Export** | Copy full preset mapping as plain text to clipboard |
| 📦 **Drag & Drop** | Drop any `device.json` or backup `.json` anywhere on the page |
| ⌨ **Keyboard Shortcuts** | `Ctrl+S` backup · `Ctrl+M` monitor · `Ctrl+E` editor · `Esc` close |
| 📱 **PWA Ready** | Inline service worker for offline caching |

---

## Browser Support

| Browser | Status | Notes |
|---|---|---|
| Chrome 80+ | ✅ Full support | Recommended |
| Edge 80+ | ✅ Full support | |
| Firefox | ⚠️ Partial | Enable `dom.webmidi.enabled` + `dom.webmidi.sysex.enabled` in `about:config` |
| Safari | ❌ Not supported | No WebMIDI API |
| Opera / Brave | ✅ Usually works | Chromium-based |

SysEx access triggers a browser permission prompt on first use — click **Allow**.

---

## Project Structure

```
MidiControls/
│
├── index.html ← App shell (HTML only, no inline JS/CSS)
├── README.md
├── manifest.json ← PWA manifest
│
├── assets/
│ ├── css/
│ │ └── midicontrols.v1.css ← All styles
│ └── js/
│ └── midicontrols.v2.js ← All application logic
│
└── devices/
├── akai_lpd8_v1/
│ └── device.json
├── akai_midimix/
│ └── device.json
├── novation_launchcontrol_mk1/
│ └── device.json
├── novation_nocturn/
│ └── device.json
├── novation_remote_zero_sl_mk1/
│ └── device.json
└── novation_remote25_sl_compact_mk1/
└── device.json
```

All device configs live in `devices/` as plain JSON — no compilation needed.  
The JS and CSS are versioned by filename (`v1`, `v2`) for cache-busting on updates.

---

## Included Devices

| Device | Manufacturer | Pads | Knobs | Faders | Buttons | SysEx |
|---|---|---|---|---|---|---|
| LPD8 v1 | Akai Professional | 8 | 8 | — | — | ✅ Read/Write |
| MIDImix | Akai Professional | — | 24 | 9 | 17 | — |
| Launch Control mk1 | Novation | 8 | — | — | 4 | ✅ LED control |
| Nocturn | Novation | — | 9 | — | 8 | ✅ Automap |
| Remote Zero SL mk1 | Novation | — | 8 | 8 | 14 | ✅ LCD write |
| Remote 25 SL Compact mk1 | Novation | 8 | 8 | — | 6 | ✅ LCD write |

---

## Adding a New Device

### 1 — Create the folder and JSON

```bash
mkdir devices/my_controller_v1
touch devices/my_controller_v1/device.json
```

### 2 — Write the device template

```json
{
  "id": "my_controller_v1",
  "name": "My Controller",
  "manufacturer": "ACME",
  "icon": "🎹",
  "color": "#6c63ff",
  "midiName": ["My Controller", "ACME MIDI"],
  "sysex": false,
  "presets": 1,
  "description": "8 pads, 8 knobs",
  "controls": {
    "pads": [
      { "id": "pad1", "label": "Pad 1", "note": 36, "cc": 1, "pc": 0, "mode": "Momentary" }
    ],
    "knobs": [
      { "id": "k1", "label": "K1", "cc": 1, "lo": 0, "hi": 127 }
    ],
    "faders": [
      { "id": "f1", "label": "Fader 1", "cc": 7 }
    ],
    "buttons": [
      { "id": "btn1", "label": "Mute", "note": 0, "cc": 0, "color": "amber" }
    ]
  },
  "defaultPresets": [
    { "name": "Preset 1", "channel": 0 }
  ],
  "quickSysEx": [
    { "label": "Reset", "bytes": "F0 00 00 00 F7" }
  ]
}
```

### 3 — Register it in `midicontrols.v2.js`

```js
const DEVICE_MANIFEST = [
  'akai_lpd8_v1',
  'akai_midimix',
  // ...
  'my_controller_v1'   // ← add this line
];
```

### 4 — Reload the page

The new device appears in the sidebar automatically.

> **Tip:** You can also skip steps 1–3 entirely and use the **Device Manager → ➕ Add Device**
> form in the UI, then export the result as `device.json` to persist it in the filesystem.

---

## Control Object Reference

### Pad
```json
{ "id": "pad1", "label": "Pad 1", "note": 36, "cc": 1, "pc": 0, "mode": "Momentary" }
```
- `note` — MIDI note number sent on pad hit (0–127)
- `cc` — CC number sent in CC mode
- `pc` — Program Change number
- `mode` — `"Momentary"` (note on + note off) or `"Toggle"` (latching)

### Knob / Encoder
```json
{ "id": "k1", "label": "K1", "cc": 1, "lo": 0, "hi": 127 }
```
- `lo` / `hi` — value range (useful for scaled encoders or inverted controls)

### Fader
```json
{ "id": "f1", "label": "Ch 1", "cc": 19 }
```

### Button / LED
```json
{ "id": "btn1", "label": "Mute 1", "note": 1, "cc": 1, "color": "amber" }
```
- `color` — LED colour: `"red"` · `"green"` · `"amber"`

### Preset
```json
{ "name": "Preset 1 — Default", "channel": 0 }
```
- `channel` — MIDI channel index (0–15, displayed as 1–16 in UI)
- Presets can override individual control values by adding `pads[]`, `knobs[]`, `faders[]`, `buttons[]` arrays

### Quick SysEx Command
```json
{ "label": "Reset Device", "bytes": "F0 7E 7F 09 01 F7" }
```
- Appears as a one-click button in the SysEx panel when the device is selected
- `bytes` — space-separated hex string, must start with `F0` and end with `F7`

---

## Device-Specific Notes

### Akai LPD8 v1
- 4 hardware presets, editable over SysEx
- **Read Preset** → `F0 47 7F 75 61 00 01 [preset 01–04] F7`
- **Write Preset** → `F0 47 7F 75 62 00 00 3F [preset] [ch] [pad×8×4 bytes] [knob×8×3 bytes] F7`
- Pad bytes: `[note] [pc] [cc] [mode: 0=Momentary 1=Toggle]`
- Knob bytes: `[cc] [lo] [hi]`
- Use **Read Device** and **Write Device** buttons in the Editor — no manual SysEx needed

### Akai MIDImix
- Fixed CC map — no SysEx editing supported by the hardware
- Channel 1 fixed; knob rows CC 16–30 (ch 1–4) and CC 46–60 (ch 5–8)
- Faders: CC 19 23 27 31 49 53 57 61 · Master: CC 62
- Buttons send Note On/Off on channel 1

### Novation Launch Control mk1
- 16 templates (8 user + 8 factory), bi-colour LEDs (red / green / amber / yellow)
- **Change template** → `F0 00 20 29 02 0A 77 [template 00–0F] F7`
- **Set LED** → `F0 00 20 29 02 0A 78 [template] [led index] [colour value] F7`
- Colour values: off=`0C` · red low=`0D` · red full=`0F` · green low=`1C` · green full=`3C` · amber full=`3F`
- Use quick-command buttons in the SysEx panel for common LED operations

### Novation Nocturn
- 8 endless encoders (relative CC) + speed dial + 8 backlit buttons
- Supports Novation Automap protocol over SysEx
- **Automap On** → `F0 00 20 29 40 5C F7`
- **Automap Off** → `F0 00 20 29 40 5D F7`

### Novation Remote SL series (Zero SL mk1 / Remote 25 SL Compact)
- Supports up to 40 templates in Automap and direct MIDI modes
- **LCD line write** → `F0 00 20 29 03 03 04 [line: 00/01] [ASCII bytes...] 00 F7`
- LCD accepts up to 72 ASCII characters per line
- Transport buttons: Stop=115 · Play=118 · Record=119 · Rewind=116 · Forward=117 · Loop=113

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + S` | Save backup snapshot of the currently selected device |
| `Ctrl + M` | Switch to MIDI Monitor panel |
| `Ctrl + E` | Switch to Editor panel |
| `Escape` | Close modal dialog / cancel active MIDI Learn |

---

## Data & Storage

All user edits (preset overrides, added devices, backups) are stored in **localStorage** under these keys:

| Key | Contents |
|---|---|
| `mc_presets` | User-edited preset data, keyed by device ID |
| `mc_backups` | Backup snapshots array |
| `mc_settings` | UI and MIDI settings |
| `mc_user_devices` | Devices added via UI (not in `devices/` folder) |

Base device configs are always re-fetched from `devices/*/device.json` on load — updating a JSON file is immediately reflected after a page reload without losing user preset edits.

Use **Backup → Export All** to download a full `.json` snapshot of everything. Drag it back onto the page or use **Backup → Restore from File** to recover.

---

## Deploy

### Local development
```bash
npx serve .          # Node — recommended
python3 -m http.server 8080
php -S localhost:8080
```

### Static hosting (zero config)
The entire project is a static folder — drop it anywhere:

- **Netlify** — drag the project folder onto [app.netlify.com/drop](https://app.netlify.com/drop)
- **Vercel** — `vercel --prod` in the project root
- **GitHub Pages** — push to a repo, enable Pages on `main` branch root
- **Plesk / cPanel** — upload via File Manager or FTP, no server config needed

### Versioning CSS / JS

When you update the CSS or JS, rename the file and update the reference in `index.html`:

```html
<!-- bump v1 → v2 to bust browser cache -->
<link rel="stylesheet" href="assets/css/midicontrols.v2.css">
<script src="assets/js/midicontrols.v3.js"></script>
```

The service worker cache key (`mc-v1` in `midicontrols.v2.js`) should be bumped at the same time:
```js
const CACHE = 'mc-v2'; // ← increment on each deploy
```

---

## Contributing

1. Fork the repo
2. Add your device JSON to `devices/YOUR_DEVICE_ID/device.json`
3. Test in Chrome with a real MIDI device or a virtual port (e.g. [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html) on Windows, IAC Driver on macOS)
4. Open a pull request — device JSONs only need the fields that apply to your hardware

---

## License

MIT — use freely, attribution appreciated.

---

## Credits

- [WebMIDI API](https://developer.mozilla.org/en-US/docs/Web/API/Web_MIDI_API) — W3C specification
- Device specs sourced from official manuals (Akai, Novation)
- No frameworks · No build tools · No tracking