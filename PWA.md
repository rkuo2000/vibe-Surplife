Surplife is an installable Progressive Web App with a cached offline controller.

Serve this directory over HTTPS, or use localhost for development:

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Open http://localhost:8000/Surplife.html. Use **Install Surplife** when the browser offers it, or the browser's install app menu. Visit online once and wait for **Available offline** before using it without a network connection. Reconnect the curtain after relaunching the app.

Deploy `Surplife.html`, `surplife.webmanifest`, `surplife-sw.js`, `icons/`, `map/`, and `controllights.md` together. Relative URLs support hosting in a subdirectory. Opening the HTML as a local file does not enable installation or offline caching. For a phone accessing a development computer over the LAN, use HTTPS; plain HTTP on a LAN address is not localhost.

Installation does not add Bluetooth support to a browser. The controller still requires Web Bluetooth. Browser microphone mode requires microphone permission; audio analysis stays local. Bluetooth and audio run while the app is open, not as background service worker tasks.

For releases, bump `CACHE_NAME` in `surplife-sw.js` whenever a cached asset changes. The new worker waits until existing app windows/tabs close before activating, so updates do not reload an active curtain session. The controller HTML also refreshes from the network when opened online. Cache cleanup is limited to Surplife caches.

BLE connection: the picker shows all nearby BLE devices by default, including curtains that do not advertise their control service. Select your curtain; the controller checks the F000 and FFFF services for a writable FF01 characteristic and subscribes to FF02 status notifications when available. An optional full service UUID override supports other service layouts. Uncheck **Show all BLE devices** to filter by the configured service candidates. Close the Surplife mobile app or other BLE controllers before connecting.

If connection fails, expand **Packet log** for the discovery stage, available characteristics, and error. Browser support depends on the browser and operating system; installing the PWA does not bypass this requirement.

In Matrix and Pic, use **Choose File** to select a 20 × 20 `.map` or `.json` file from this project’s `map/` directory. Matrix opens maps for editing; Pic displays them. Matrix’s **Save map** asks for the local `map` folder on the first save in browsers with folder access and reuses that folder until reloading. Other browsers download saved maps; move those files into `map/` manually. Static hosting cannot write files back to the server.
