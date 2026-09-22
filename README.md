# Homebridge + Nest Thermostat: How It's Set Up

Nest Thermostat E → Google Smart Device Management (SDM) API → Homebridge on the Raspberry Pi → Apple HomeKit.

Set up 2026-09-22.

> This folder's `index.html` and `privacy.html` are published at https://fganime.github.io.
> **Don't upload this README there.** It describes the setup and shouldn't be public.

---

## The pieces

| Piece | Where | Notes |
|---|---|---|
| Raspberry Pi | `192.168.5.197` (`raspberrypi.local`) | Debian 12, user `raspfgs`, SSH key login from the Mac (`~/.ssh/id_ed25519`) |
| Homebridge | on the Pi, runs at boot | Dashboard: http://192.168.5.197:8581 · HomeKit code **898-76-700** |
| Nest plugin | `homebridge-google-nest-sdm` | Shows up as **Living Room Thermostat** |
| Plugin config | Pi: `/var/lib/homebridge/config.json` | Holds client ID, **client secret**, refresh token. Backups sit next to it as `config.json.bak-*` |
| Google account | ganimefarid@gmail.com | Owns everything below |
| Google Cloud project | `nest-homebridge-509419` (number `775191755289`) | OAuth client, Pub/Sub |
| Device Access project | `9fda1785-b552-498c-ab38-98f3a32f0e22` | https://console.nest.google.com/device-access ($5 one-time, paid) |
| OAuth client ID | `775191755289-ovkt3qbfu2qf8lcbnok3rljctra985so.apps.googleusercontent.com` | Web application, redirect URI `http://localhost` |
| Pub/Sub topic | `projects/nest-homebridge-509419/topics/nest-events` | `sdm-publisher@googlegroups.com` has Publisher role on it |
| Pub/Sub subscription | `projects/nest-homebridge-509419/subscriptions/homebridge` | The plugin pulls live thermostat events from here |
| Consent-screen pages | https://fganime.github.io (GitHub account `fganime`, repo `fganime.github.io`) | Required for the app to be **published**. Don't delete the repo |

The client secret is intentionally not in this file. It's in the Pi's `config.json` and in Google Cloud → Google Auth Platform → Clients.

---

## Why each part exists

- **Device Access ($5):** Google's only official way to let anything outside Google Home control a Nest.
- **OAuth client + consent screen:** lets you sign in and grant the plugin access. The result is a **refresh token** the plugin uses to talk to Google on its own.
- **Published (not Testing):** apps left in Testing get tokens that expire every 7 days. Publishing required a home page + privacy policy, which is why the GitHub Pages site exists.
- **Pub/Sub:** Google pushes thermostat changes (temp, mode) into the topic; the plugin reads them from the subscription so HomeKit updates instantly. The plugin won't start without it.

---

## If the thermostat stops responding in HomeKit

1. Check the Homebridge log in the dashboard (http://192.168.5.197:8581 → Logs), or:
   ```
   ssh raspfgs@192.168.5.197 'sudo tail -n 50 /var/lib/homebridge/homebridge.log'
   ```
2. If you see auth errors like `invalid_grant`, the token was revoked (password change, removed app access, etc.). Get a new one:

### Getting a new refresh token

1. Open this link, sign in as ganimefarid@gmail.com, click through the "unverified app" warning (**Advanced → Go to Homebridge**), and allow:

   https://accounts.google.com/o/oauth2/v2/auth?client_id=775191755289-ovkt3qbfu2qf8lcbnok3rljctra985so.apps.googleusercontent.com&redirect_uri=http://localhost&response_type=code&access_type=offline&prompt=consent&scope=https://www.googleapis.com/auth/sdm.service%20https://www.googleapis.com/auth/pubsub

2. You'll land on a "can't connect to localhost" page. That's expected. Copy the `code=...` value from the address bar (everything between `code=` and `&scope`). It expires in a few minutes.
3. Exchange it for a refresh token:
   ```
   curl -s https://oauth2.googleapis.com/token \
     -d client_id=775191755289-ovkt3qbfu2qf8lcbnok3rljctra985so.apps.googleusercontent.com \
     -d client_secret=CLIENT_SECRET \
     --data-urlencode code=PASTE_CODE \
     -d grant_type=authorization_code -d redirect_uri=http://localhost
   ```
   If the response has no `refresh_token_expires_in`, the token is permanent.
4. Put the `refresh_token` value into `refreshToken` in the Pi's `config.json` (or the plugin's settings in the Homebridge dashboard), then restart Homebridge:
   ```
   ssh raspfgs@192.168.5.197 'sudo systemctl restart homebridge'
   ```

Gotchas learned the hard way:
- Use the `accounts.google.com` link above, **not** the `nestservices.google.com/partnerconnections/...` one. The Nest page errors with a localhost redirect. It's only needed if you add a *new* Nest device, and then you'd temporarily need a non-localhost redirect.
- The scopes must include **both** `sdm.service` and `pubsub`, or the plugin fails with an event-subscription error.
- Copy the code as text. Reading it off a screenshot risks mixing up `l`/`I`/`1` and `O`/`0`.

---

## Other Pi notes

- **Plex** also runs on the Pi, serving media from a Seagate Backup Plus Ultra Slim (1.8 TB, NTFS) at `/mnt/media`.
- The drive spins down on its own when idle (its firmware handles it, no Pi setting needed). Check with:
  ```
  ssh raspfgs@192.168.5.197 'sudo hdparm -C /dev/sda'
  ```
- `hdparm` and `smartmontools` are installed. Health check: `sudo smartctl -a -d sat /dev/sda`.
