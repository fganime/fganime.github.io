# Changelog

All notable changes to Nightstand are documented here. Dates use the project-local release date; commit hashes refer to the repository history.

## 2026-11-08 — Beta reliability and sensor integration

- Restored continuous second-hand animation using `requestAnimationFrame`; the previous timer removal had left the hand static.
- Added an explicit weather enable/refresh action and persisted the user's weather opt-in locally.
- Reworked the Open-Meteo request to use `URL` and `URLSearchParams`, validate device coordinates, and report which stage fails.
- Fixed the service worker so cross-origin requests (including Open-Meteo) bypass the app cache. Failed third-party API requests are no longer replaced with the cached Nightstand HTML shell.
- Updated the indoor Homebridge parser to accept the proxy's actual response shape: a top-level `sensors` array, sensor `name`, and nested `values` fields.
- Bumped the service-worker cache through v13 so installed clients can receive the updated app shell.
- Added documentation for current architecture, setup, data handling, and beta limitations.

### Current beta behavior

- Analog clock, date, optional sweeping second hand, software dimmer, and best-effort Screen Wake Lock.
- Outdoor current conditions from Open-Meteo, using device geolocation.
- Indoor readings from a fixed, local-only Homebridge proxy endpoint.

### Known limitations

- The indoor proxy address is hard-coded for one home network; there is no general-purpose connection setup.
- A trusted, correctly named HTTPS certificate is required for the local proxy.
- Live weather and indoor readings require their respective network connections.
- This remains a personal-use PWA beta, not a native App Store release.
