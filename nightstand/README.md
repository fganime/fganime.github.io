# Nightstand

Nightstand is an experimental, installable bedside clock for iPad, currently hosted as a static Progressive Web App (PWA) on GitHub Pages.

- **Live beta:** https://fganime.github.io/nightstand/
- **Repository:** https://github.com/fganime/fganime.github.io
- **Status:** Personal-use beta; not an App Store app.

## Current features

- Minimal analog clock with a deep-red display on a true-black background
- Date display and 24 evenly spaced dial marks, with longer quarter-hour marks
- Continuously sweeping second hand, with an option to hide it
- Screen dimmer overlay
- Screen Wake Lock request where supported by the browser and iPadOS
- Outdoor current conditions from Open-Meteo, using the device's location after the user enables weather
- Indoor temperature, humidity, and thermostat status from a local Homebridge proxy
- Local browser storage for settings and weather opt-in
- PWA manifest and service worker for Home Screen installation and caching of the app shell

## Install on iPad

1. Open https://fganime.github.io/nightstand/ in Safari.
2. Choose **Share → Add to Home Screen**.
3. Enable **Open as Web App** if that option is presented.
4. Launch Nightstand from its Home Screen icon.
5. Open Settings to enable the second hand, display wake lock, dimmer, and outdoor weather as desired.

The dimmer is a software black overlay; it does not change the iPad's hardware brightness. Screen Wake Lock is best-effort: iPadOS may release it when the app is backgrounded or the device needs to conserve power. Live weather and indoor readings require network access.

## Data sources and architecture

### Outdoor weather

- Uses browser geolocation after the user taps **Enable / refresh weather**.
- Requests current temperature, relative humidity, and weather code from the Open-Meteo forecast API.
- Displays Fahrenheit and a short condition label.
- Refreshes approximately every 30 minutes while weather is enabled.
- The app stores a local weather-enabled preference so it can refresh on later launches. Location permission remains controlled by iPadOS/Safari.

Open-Meteo is a third-party service. Weather will not work if location permission is denied, the service is unavailable, or network access is blocked.

### Indoor Homebridge readings

The current beta polls this fixed local endpoint every three minutes:

`https://192.168.6.125:8591/sensors`

The endpoint is specific to the developer's home network and is not configurable in the current UI. The Pi-side proxy is responsible for authenticating to Homebridge; the Nightstand page makes an unauthenticated, read-only GET request and does not store Homebridge credentials or tokens. The app does not send Homebridge control/write requests and does not use Tailscale as a fallback.

The parser expects a JSON object with a top-level `sensors` array. Each sensor has a `name` and a `values` object. The expected sensor names and fields are:

| Sensor name | Required values |
|---|---|
| `Nursery Temperature` | `CurrentTemperature` |
| `Nursery Humidity` | `CurrentRelativeHumidity` |
| `Living Room Thermostat` | `CurrentTemperature`, `TargetTemperature`, `TargetHeatingCoolingState`, `CurrentHeatingCoolingState`; `CurrentRelativeHumidity` is displayed when present |

Temperatures from Homebridge are expected in Celsius and converted to Fahrenheit for display. The current app displays nursery temperature/humidity, living room temperature/humidity, thermostat setpoint, mode, and operating state.

The local endpoint must be reachable from the iPad on the same network. Its HTTPS certificate must be trusted by iPadOS and valid for the endpoint address (including an IP-address Subject Alternative Name when connecting by IP). A Safari certificate warning or TLS failure prevents the app from reading the endpoint.

## Privacy and security

- No Nightstand account or sign-in is used.
- No analytics or advertising code is included in this beta.
- Settings and the weather opt-in are stored in browser local storage.
- Location is requested only when the user enables weather; coordinates are sent to Open-Meteo to retrieve local conditions.
- Indoor sensor requests go to the configured local Pi proxy. Homebridge credentials belong on the proxy, not in this web app.
- The indoor request is read-only. The app does not control thermostats or accessories.

This is a description of the current implementation, not a third-party security audit.

## Known limitations / beta status

- The Homebridge proxy IP and port are hard-coded for one home network.
- There is no in-app setup flow for another user's Homebridge instance, proxy URL, or sensor names.
- The app depends on a separately running Pi proxy and a correctly trusted HTTPS certificate for indoor readings.
- Indoor readings are hidden when the local proxy is unavailable or its response does not match the expected schema.
- Weather requires location permission and an internet connection.
- Screen Wake Lock behavior depends on iPadOS/Safari and is not guaranteed indefinitely.
- Offline support caches the app shell; it does not make live weather or Homebridge data available offline.
- This beta has not been packaged, tested, or submitted as a native iOS/iPadOS App Store application.

## Direction for a public release

A public release would need a configurable and documented Homebridge connection model, safer onboarding for local-network access and certificates, broader device testing, accessibility and reliability work, privacy disclosures, and a deliberate native iPadOS distribution strategy. Simply wrapping this website in a minimal web view may not satisfy Apple's minimum-functionality review requirements. See [Apple's App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) and the [CHANGELOG](../CHANGELOG.md).

## Development notes

The app is plain HTML, CSS, and JavaScript under `nightstand/`. GitHub Pages publishes it from this repository. The service worker caches the app shell and uses a versioned cache name; increment that version when changing cached app assets so installed clients receive updates.
