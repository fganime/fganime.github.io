# Nightstand

A static, installable bedside clock for iPad, hosted on GitHub Pages.

## Features

- Live local time and date
- Optional 24-hour clock and seconds
- Automatic warm-red night theme (9 PM–6 AM), plus manual override
- Software dimmer overlay
- Screen Wake Lock where supported
- PWA manifest and service worker for home-screen installation and basic offline use
- Settings saved locally in the browser

## Install on iPad

1. Open the published `/nightstand/` URL in Safari.
2. Tap Share, then **Add to Home Screen**.
3. Launch Nightstand from the new Home Screen icon.
4. Open the gear menu and enable **Keep display awake** if supported.
5. For a bedside setup, use Guided Access to keep the iPad in the dashboard.

The dimmer is a black overlay, not a hardware brightness control. Screen Wake Lock depends on iPadOS/Safari support and may be released when the app is backgrounded or the device needs to conserve power.

No analytics, tracking, accounts, or third-party services are used. Preferences are stored locally on the device.