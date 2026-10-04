# Surplife Smart LED Curtain (PWA)

Web Bluetooth controller for Surplife 20×20 BLE LED curtains, built from the protocol notes in
[controllights.md](controllights.md) ([upstream gist](https://gist.github.com/marcelrv/36f9d2c66a273289211cc5fa73b00018#file-surplife_protocol_description-md)).
Live: <https://rkuo2000.github.io/vibe-Surplife/>

Open `Surplife.html` over HTTPS (e.g. GitHub Pages) or `localhost` in a browser with Web Bluetooth
(Chrome/Edge on desktop or Android, Bluefy on iPhone/iPad), then press **Connect curtain**. Close the Surplife
mobile app first; the curtain accepts only one BLE controller at a time. **Try demo** previews the app without a curtain.

```sh
npm run serve                     # or: python3 -m http.server 8765 --bind 127.0.0.1
```

Then open <http://localhost:8765/Surplife.html>.

| Mode | What it does |
|---|---|
| **Color** | Color wheel and brightness; the color is shared with Text and Pic |
| **Effects** | Built-in effects 0–17 with speed |
| **Matrix** | Toggle individual pixels on the 20×20 grid, load and save `.map` files, mirror output |
| **Text** | Up to 3 lines / 500 characters, shown Static, Scrolling or Flashing in the exact selected color |
| **Pic** | Show a 20×20 map from `map/`, or a PNG/JPEG/WebP/GIF fitted or cropped to the curtain |
| **Rhythm** | Microphone or local audio file drives Flashing Text, Flashing Pic, Sound Waves, Sound Ring or Equalizer Bars |

*Connection & protocol settings* has status request, time sync, service UUID override, color calibration
(pin test and mapping), and a **Packet log** you can download when a connection fails.

## BLE

The picker lists all nearby BLE devices by default, because some curtains don't advertise their control service.
After you pick one, the app looks in services `F000` and `FFFF` for writable characteristic `FF01` and subscribes
to `FF02` status notifications. On connect it syncs the time (`0x10`) and sends the handshake (`0xEA`).

## Install and offline

Use **Install Surplife** (or the browser's install menu), and visit once online until it shows
**Available offline**. Installing doesn't add Web Bluetooth to a browser that lacks it, and Bluetooth and audio
only run while the app is open. See [PWA.md](PWA.md) for deployment, update and map-saving details.

Files: `Surplife.html` (app; `index.html` is an identical copy for the site root), `surplife-sw.js` +
`surplife.webmanifest` (offline install), `icons/`, `map/` (20×20 presets: `HEART_20x20.map`, `STAR_20x20.map`),
`controllights.md` (protocol notes), `PWA.md`. Keep `index.html` in sync with `Surplife.html`, and bump
`CACHE_NAME` in the service worker when you change cached assets.
