# 🎹 MidiControls

**Browser-based MIDI controller editor — no install, no backend, no build step.**

Edit device presets visually, send and receive SysEx, monitor MIDI traffic in real time,
and back up everything to JSON. Runs entirely in the browser via the WebMIDI API.

---

## Quickstart

```bash
git clone https://github.com/YOUR_USERNAME/midicontrols.git
cd midicontrols
npx serve .
```

Open **http://localhost:3000** in Chrome or Edge.  
Plug in your USB MIDI device → select ports in the top bar → start editing.

> **Why a local server?**  
> Device templates load via `fetch()` from `devices/*/device.json`.  
> Browsers block `fetch()` on `file://` URLs. Any static HTTP server works fine.

### Alternative servers

```bash
python3 -m http.server 8080   # http://localhost:8080
php -S localhost:8080          # http://localhost:8080
npx http-server .              # http://localhost:8080
```

---

## Browser Support

| Browser | Status | Notes |
|---|---|---|
| Chrome 80+ | ✅ Full | Recommended |
| Edge 80+ | ✅ Full | |
| Brave / Opera | ✅ Full | Chromium-based |
| Firefox | ⚠️ Partial | Enable `dom.webmidi.enabled` + `dom.webmidi.sysex.enabled` in `about:config` |
| Safari | ❌ None | No WebMIDI API |

SysEx access triggers a browser permission prompt on first use — click **Allow**.

---

## Features

### Editor
- Visual pad / knob / fader / button grid per device
- Live CC value display — knobs and faders update in real time from incoming MIDI
- **MIDI Learn** — click LEARN on any pad, press a hardware key → auto-maps the note
- **Knob drag** — drag any knob up/down to send live CC to MIDI Out
- **Button LEDs** — click any button in the UI to trigger Note On/Off via MIDI Out
- Per-preset MIDI channel selector
- Pad toggle between Momentary and Toggle mode
- **Read Device** — request a SysEx preset dump from hardware (LPD8 supported)
- **Write Device** — send current preset to hardware over SysEx (LPD8 supported)

### MIDI Monitor
- Real-time message log — Note On/Off, CC, PC, SysEx
- Filter by message type
- Pause / resume without losing buffered messages
- Hex value display toggle
- Up to 500 messages buffered, newest on top

### SysEx Panel
- Send raw SysEx as space-separated hex bytes
- Built-in hex parser with byte count and validation
- Send raw CC / Note On / Note Off / Program Change directly
- Per-device **Quick Commands** — one-click SysEx buttons for common operations

### Backup & Restore
- Save named snapshots per device at any time (`Ctrl+S`)
- Restore any snapshot with one click
- Export full backup as a single `.json` file
- Restore from file or drag & drop anywhere on the page
- Export individual preset as `.json`
- Copy full CC map to clipboard as plain text

### Device Manager
- Add new devices via UI form — generates the control structure automatically
- Import / export device templates as `.json`
- Remove user-added devices
- Built-in devices are protected (cannot be accidentally deleted)

### Settings
- MIDI Thru — echo MIDI In to MIDI Out
- Highlight controls on incoming message
- Auto-refresh port list on device connect/disconnect
- Show note names (C4, D#3…) or raw numbers
- Show CC values as hex
- Compact layout mode

---

## Project Structure

```
MidiControls/
│
├── index.html ← App shell — HTML only
├── README.md
├── manifest.json ← PWA manifest
├── gen-icons.mjs ← One-time icon generator
│
├── assets/
│ ├── css/
│ │ └── midicontrols.v1.css ← All styles
│ ├── js/
│ │ └── midicontrols.v2.js ← All application logic
│ ├── icon-192.png
│ └── icon-512.png
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

JS and CSS filenames are versioned (`v1`, `v2`) for manual cache busting — bump the
filename and update the reference in `index.html` when deploying changes.

---

## Included Devices

| Device | Manufacturer | Pads | Knobs | Faders | Buttons | SysEx |
|---|---|:---:|:---:|:---:|:---:|:---:|
| LPD8 v1 | Akai Professional | 8 | 8 | — | — | ✅ Read + Write |
| MIDImix | Akai Professional | — | 24 | 9 | 17 | — |
| Launch Control mk1 | Novation | 8 | — | — | 4 | ✅ LED control |
| Nocturn | Novation | — | 9 | — | 8 | ✅ Automap |
| Remote Zero SL mk1 | Novation | — | 8 | 8 | 14 | ✅ LCD write |
| Remote 25 SL Compact mk1 | Novation | 8 | 8 | — | 6 | ✅ LCD write |

---

## Adding a New Device

### Option A — UI (no files)

1. Open **Devices → ➕ Add Device**
2. Fill in name, manufacturer, control counts
3. The device is saved to `localStorage` immediately
4. Export it via **Device Manager → 📤 Export** to save it as a permanent `device.json`

### Option B — JSON file (recommended for sharing)

**1. Create the folder**
```bash
mkdir -p devices/my_controller_v1
```

**2. Write `devices/my_controller_v1/device.json`**
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
  "description": "8 pads · 8 knobs",
  "controls": {
    "pads": [
      { "id": "pad1", "label": "Pad 1", "note": 36, "cc": 1, "pc": 0, "mode": "Momentary" }
    ],
    "knobs": [
      { "id": "k1", "label": "K1", "cc": 1, "lo": 0, "hi": 127 }
    ],
    "faders": [],
    "buttons": []
  },
  "defaultPresets": [
    { "name": "Preset 1", "channel": 0 }
  ],
  "quickSysEx": []
}
```

**3. Register it in `assets/js/midicontrols.v2.js`**
```js
const DEVICE_MANIFEST = [
  'akai_lpd8_v1',
  'akai_midimix',
  'novation_launchcontrol_mk1',
  'novation_nocturn',
  'novation_remote_zero_sl_mk1',
  'novation_remote25_sl_compact_mk1',
  'my_controller_v1'    // ← add this line
];
```

**4. Reload** — the device appears in the sidebar.

---

## Control Reference

### Pad
```json
{ "id": "pad1", "label": "Pad 1", "note": 36, "cc": 1, "pc": 0, "mode": "Momentary" }
```

| Field | Type | Description |
|---|---|---|
| `note` | 0–127 | MIDI note sent on hit |
| `cc` | 0–127 | CC number (CC mode) |
| `pc` | 0–127 | Program Change number |
| `mode` | string | `"Momentary"` or `"Toggle"` |

### Knob / Encoder
```json
{ "id": "k1", "label": "K1", "cc": 1, "lo": 0, "hi": 127 }
```

| Field | Type | Description |
|---|---|---|
| `cc` | 0–127 | CC number |
| `lo` | 0–127 | Minimum value |
| `hi` | 0–127 | Maximum value |

### Fader
```json
{ "id": "f1", "label": "Ch 1", "cc": 19 }
```

### Button / LED
```json
{ "id": "btn1", "label": "Mute 1", "note": 1, "cc": 1, "color": "amber" }
```

| Field | Type | Description |
|---|---|---|
| `note` | 0–127 | Note sent on click |
| `cc` | 0–127 | CC number |
| `color` | string | `"red"` · `"green"` · `"amber"` |

### Preset override
Presets can override any control value by including the matching array:
```json
{
  "name": "Ableton Live",
  "channel": 7,
  "pads":  [{ "note": 88, "cc": 30, "pc": 24, "mode": "Momentary" }],
  "knobs": [{ "cc": 50, "lo": 0, "hi": 127 }]
}
```
Only the fields you include are overridden — everything else falls back to the base `controls` definition.

### Quick SysEx command
```json
{ "label": "Reset", "bytes": "F0 7E 7F 09 01 F7" }
```
Must start with `F0` and end with `F7`. Appears as a one-click button in the SysEx panel.

---

## Device Notes

### Akai LPD8 v1
4 hardware presets, fully editable over SysEx.

| Operation | SysEx |
|---|---|
| Read preset N | `F0 47 7F 75 61 00 01 0N F7` |
| Write preset N | `F0 47 7F 75 62 00 00 3F 0N [ch] [pad×8×4B] [knob×8×3B] F7` |

Pad bytes: `[note] [pc] [cc] [mode: 00=Momentary 01=Toggle]`  
Knob bytes: `[cc] [lo] [hi]`  
Use the **Read Device** and **Write Device** buttons — no manual SysEx needed.

### Akai MIDImix
Fixed CC map, no SysEx. Channel 1 only.

| Controls | CC numbers |
|---|---|
| Knobs ch 1–4 | 16–18, 20–22, 24–26, 28–30 |
| Knobs ch 5–8 | 46–48, 50–52, 54–56, 58–60 |
| Faders ch 1–8 | 19, 23, 27, 31, 49, 53, 57, 61 |
| Master fader | 62 |
| Mute buttons | Note 1–8 |
| Rec Arm buttons | Note 10–17 |

### Novation Launch Control mk1
16 templates (8 user + 8 factory), bi-colour LEDs.

| Operation | SysEx |
|---|---|
| Change template | `F0 00 20 29 02 0A 77 [00–0F] F7` |
| Set LED colour | `F0 00 20 29 02 0A 78 [template] [led] [colour] F7` |

LED colour values: `0C` off · `0D` red low · `0F` red full · `1C` green low · `3C` green full · `3F` amber full

### Novation Nocturn
8 endless encoders (relative CC) + speed dial + 8 backlit buttons.

| Operation | SysEx |
|---|---|
| Automap On | `F0 00 20 29 40 5C F7` |
| Automap Off | `F0 00 20 29 40 5D F7` |

### Novation Remote SL (Zero SL mk1 / Remote 25 SL Compact)
Up to 40 templates, Automap + direct MIDI modes.

| Operation | SysEx |
|---|---|
| Init Automap | `F0 00 20 29 03 03 12 01 F7` |
| Write LCD line 1 | `F0 00 20 29 03 03 04 00 [ASCII…] 00 F7` |
| Write LCD line 2 | `F0 00 20 29 03 03 04 01 [ASCII…] 00 F7` |

Transport note numbers: Stop=115 · Play=118 · Record=119 · Rewind=116 · Forward=117 · Loop=113

---

## Data & Storage

Everything is stored in `localStorage`. No data is sent anywhere.

| Key | Contents |
|---|---|
| `mc_presets` | User-edited preset data, keyed by device ID |
| `mc_backups` | Backup snapshots array |
| `mc_settings` | UI and MIDI behaviour settings |
| `mc_user_devices` | Devices added via the UI (not from `devices/` folder) |

Base device configs are always re-fetched from `devices/*/device.json` on load.
Updating a JSON file is reflected immediately after reload — user preset edits are preserved.

**Always export a full backup before clearing localStorage or switching browsers.**

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + S` | Save backup of the current device |
| `Ctrl + M` | Switch to MIDI Monitor |
| `Ctrl + E` | Switch to Editor |
| `Escape` | Close modal / cancel MIDI Learn |

---

## Deploy

### Netlify
Drag the project folder onto [app.netlify.com/drop](https://app.netlify.com/drop) — live in 30 seconds.

### Vercel
```bash
npx vercel --prod
```

### GitHub Pages
Push to a repo → Settings → Pages → Source: `main` branch `/root` → Save.

### Plesk / shared hosting
Upload the entire folder via File Manager or FTP. No server-side config needed.

### Cache busting on updates
When you update CSS or JS, rename the file and update `index.html`:
```html
<link rel="stylesheet" href="assets/css/midicontrols.v2.css">
<script src="assets/js/midicontrols.v3.js"></script>
```
Also bump the service worker cache key in the JS file:
```js
const CACHE = 'mc-v2'; // increment on each deploy
```

---

## Troubleshooting

**Devices show 404 in the console**  
→ The `devices/` subfolders don't exist yet. Create them and place each `device.json` inside:
```bash
mkdir -p devices/akai_lpd8_v1
# ... repeat for each device
```

**`icon-192.png` 404 in console**  
→ Run the browser console snippet from the setup docs to generate and download both PNG icons,
then move them to `assets/`.

**WebMIDI permission denied**  
→ Chrome requires HTTPS or `localhost`. Make sure you're not opening via `file://`.
If running on a remote server without SSL, use a reverse proxy with a certificate.

**No MIDI ports appear in the dropdowns**  
→ Check the device is connected and powered before opening the app.
On macOS, check **Audio MIDI Setup → MIDI Studio**. On Windows, check **Device Manager**.
Try unplugging and replugging — the port list auto-refreshes.

**Firefox — no MIDI access**  
→ Open `about:config`, search `webmidi`, set both `dom.webmidi.enabled`
and `dom.webmidi.sysex.enabled` to `true`, then reload.

---

## Contributing

1. Fork the repo
2. Add your device to `devices/YOUR_DEVICE_ID/device.json`
3. Test with a real MIDI device or a virtual port:
   - **macOS** — IAC Driver in Audio MIDI Setup
   - **Windows** — [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html)
   - **Linux** — `modprobe snd-virmidi` or JACK
4. Open a pull request — device JSONs only, no changes to core JS needed

---

## License

MIT — use freely, attribution appreciated.

---

*WebMIDI API · Pure HTML/CSS/JS · No frameworks · No tracking · No server required*