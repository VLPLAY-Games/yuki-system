# Yuki System

Yuki System is an open-source modular ecosystem for smart devices and home automation. It lets you
control multiple devices (PCs, phones, IoT devices) through one central server, using a single
shared protocol every client speaks.

This repository is the **meta repository**: it has no code of its own, just documentation, the
architecture overview, and links to every module's own repository. `yuki-speaker` is a separate,
older ESP32 module not covered by this document or by the ecosystem's 1.0.0 hardening pass.

Also available in [русский](readme.ru.md) and [日本語](readme.ja.md).

## Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Modules](#modules)
- [Protocol](#protocol)
- [Security model](#security-model)
- [Setting up a deployment](#setting-up-a-deployment)
- [Roadmap](#roadmap)
- [License](#license)

## Overview

Every device in the ecosystem - a Windows/Linux/Android remote-control client, an ESP32 humidifier,
whatever comes next - connects to one server (`yuki-core`) over WebSocket, authenticates with a
shared token, and exchanges status/commands/metrics with it in a common JSON format
([`yuki-protocol`](https://github.com/VLPLAY-Games/yuki-protocol)). An admin manages the whole thing
from a web dashboard ([`yuki-webui`](https://github.com/VLPLAY-Games/yuki-webui)): approve new
devices, send commands, watch metrics, browse the audit log, rotate the auth token.

Nothing here assumes cloud infrastructure - `yuki-core` is a single Python process you run on your
own network, and the default configuration (plaintext WebSocket, a token generated on first boot)
is meant to work immediately on a home LAN with zero setup, while still supporting TLS, stronger
authentication, and systemd-managed secrets for anyone who wants them.

## Architecture

```text
                          ┌──────────────┐
                          │  yuki-core   │  Python / asyncio WebSocket server
                          │ (the "brain")│  ws://host:8000  (/device, /webui)
                          └──────┬───────┘
                 ┌───────────────┼────────────────────────┐
                 │               │                         │
          ┌──────▼─────┐  ┌──────▼──────┐          ┌───────▼────────┐
          │ yuki-webui │  │  Devices     │          │  yuki-protocol │
          │ Flask +    │  │  (/device)   │          │  shared SDK,   │
          │ browser JS │  │              │          │  one per       │
          │ (/webui)   │  └──────┬───────┘          │  language      │
          └────────────┘         │                  └────────────────┘
                 ┌────────────────┼─────────────────────┬───────────────┐
                 │                │                     │               │
        ┌────────▼───────┐ ┌──────▼───────┐  ┌──────────▼────────┐ ┌────▼─────────────┐
        │ yuki-device-pc │ │ yuki-device- │  │ yuki-device-pc-   │ │ yuki-humidifier   │
        │ (Windows, C#)  │ │ android      │  │ linux (Python/    │ │ (ESP32-C3, C++)   │
        │                │ │ (Kotlin)     │  │ GTK4/libadwaita)  │ │                   │
        └────────────────┘ └──────────────┘  └────────────────────┘ └───────────────────┘
```

`yuki-core` is the only component every other module talks to directly. Devices connect to
`/device`, authenticate with a shared bearer token, and exchange status/commands/metrics. Browsers
connect to `/webui` (served the static dashboard by the `yuki-webui` Flask app) and, after
authenticating with their own token, get the same real-time view plus admin actions (approve
devices, rotate the token, browse the audit log, ...).

## Modules

| Module | Language | Role |
|---|---|---|
| [`yuki-core`](https://github.com/VLPLAY-Games/yuki-core) | Python | Server / "brain": routing, device auth/authorization, metrics/audit storage, token rotation |
| [`yuki-protocol`](https://github.com/VLPLAY-Games/yuki-protocol) | Python / C# / JavaScript / C++ / Kotlin | The shared message format, implemented per language |
| [`yuki-webui`](https://github.com/VLPLAY-Games/yuki-webui) | Python (Flask) + JS | Web dashboard: device list, groups, tags, widgets, settings |
| [`yuki-device-pc`](https://github.com/VLPLAY-Games/yuki-device-pc) | C# (WinForms) | Windows remote-control client |
| [`yuki-device-pc-linux`](https://github.com/VLPLAY-Games/yuki-device-pc-linux) | Python (GTK4/libadwaita) | Linux remote-control client, same feature set as the Windows one |
| [`yuki-device-android`](https://github.com/VLPLAY-Games/yuki-device-android) | Kotlin | Android remote-control client |
| [`yuki-humidifier`](https://github.com/VLPLAY-Games/yuki-humidifier) | C++ (Arduino/ESP32) | Smart humidifier firmware |

Each module is its own repository with its own README covering exact install/build/run
instructions, configuration, and dependencies - this document stays at the ecosystem level.

## Protocol

All WebSocket traffic is JSON, one message per frame, no binary framing:

```json
{"protocol": "yuki/1.0", "type": "...", "id": "...", "timestamp": 0, "payload": {}}
```

`type` selects the message's meaning (`hello`, `welcome`, `status`, `command`, `command_result`,
`metrics`, `device_to_device`, `challenge`, `auth`, ...). Every implementation validates `protocol`
is exactly `yuki/1.0` and rejects anything else - there's no cross-version compatibility shim, so a
deployment should run one protocol version everywhere. See
[`yuki-protocol`](https://github.com/VLPLAY-Games/yuki-protocol)'s own README for the full message
catalogue, including the two device-authentication handshakes below.

## Security model

- **Device authentication**: a shared bearer token, either the legacy way (`auth_token` sent
  directly in `hello`) or via challenge-response (`hello{nonce_c}` → `challenge{nonce_s}` →
  `auth{hmac}`, `hmac = HMAC-SHA256(token, "{nonce_c}:{nonce_s}")`), which never puts the token on
  the wire at all. `yuki-core` picks the method per-connection based on whether `hello` carries
  `nonce_c` - existing devices keep working unchanged, new ones can opt into the stronger handshake
  with no server-side configuration. Either way, comparisons are constant-time.
- **Token source and rotation**: the token is either admin-managed (`YUKI_AUTH_TOKEN` env var, or a
  systemd credential via `LoadCredential=yuki_auth_token:...`) - in which case `yuki-core` never
  rotates it - or auto-generated and stored in `.token` (mode 600), rotated automatically every
  `YUKI_TOKEN_ROTATION_HOURS` (default 24h) by a background task, or on demand from `yuki-webui`.
  After any rotation the previous token stays valid for `YUKI_TOKEN_GRACE_MINUTES` (default 30) so a
  device that was briefly offline isn't locked out before it picks up the `token_update` push.
- **Device authorization**: presenting a valid token isn't enough on its own - a device new to
  `yuki-core` is held pending until an admin approves it from `yuki-webui` (or it's pre-seeded into
  the authorized set). Devices can also be blacklisted outright.
- **WebUI login**: `yuki-webui` ships with a default `admin`/`admin` login (hashed on disk, not
  plaintext) - change it from the dashboard's Settings panel once you're in. The browser's
  real-time `/webui` socket to `yuki-core` requires its own token handshake (fetched from
  `yuki-webui` only after you're logged in) *and* an allow-listed `Origin` header
  (`YUKI_WEBUI_ALLOWED_ORIGINS` on the `yuki-core` side, defaulting to `yuki-webui`'s own default
  host/port) - a page served from anywhere else can't open that socket even if it somehow obtained a
  valid token, and the control-plane socket isn't reachable at all by an unauthenticated browser.
- **Rate limiting**: `yuki-core` limits handshake attempts per source IP (before any device_id is
  even trusted - stops one IP from brute-forcing many device_ids, each of which would otherwise get
  its own fresh quota) as well as commands per device_id after the handshake.
- **Capability allow-lists**: every client (PC, Linux, Android, the humidifier) advertises which
  commands it's willing to run and refuses anything not on that list *locally*, independent of
  whatever `yuki-core` allows - a compromised or misconfigured server can't make a device run a
  command its owner explicitly disabled.
- **Transport encryption**: **off by default everywhere** (plain `ws://`/`http://`), matching a
  typical trusted-LAN home-automation deployment. Every component supports opting into TLS -
  `yuki-core` (`YUKI_TLS_ENABLED`/`YUKI_TLS_CERT`/`YUKI_TLS_KEY`), `yuki-webui`
  (`YUKI_WEBUI_TLS_ENABLED`/...), `yuki-humidifier` (`ENABLE_TLS` in `Config.h`), and the desktop/
  mobile clients simply by using a `wss://`/`https://` address - see each module's own README for
  exact variable names. If TLS is explicitly enabled but misconfigured (missing cert/key), both
  `yuki-core` and `yuki-webui` refuse to start rather than silently falling back to plaintext -
  nothing pretends to be encrypted when it isn't. `yuki-webui`'s Settings page has a read-only
  Encryption panel showing the true current state of both the page itself and its core connection.
- **Secrets at rest**: the core token (when file-based) and webui credentials are stored with
  restrictive file permissions (mode 600) and hashed (webui password) where applicable; the Windows
  client encrypts its stored token with DPAPI; the shared token is never written to either SQLite
  database.

None of this replaces running the ecosystem on a network you actually trust (your home LAN, a VPN,
...) - it raises the bar for what a device or browser on that network can do to each other, it
doesn't turn `yuki-core` into something safe to expose directly to the public internet.

## Setting up a deployment

1. Start `yuki-core` (see its README) - note the generated token in `.token`, or set
   `YUKI_AUTH_TOKEN` yourself before first run.
2. Start `yuki-webui`, log in with `admin`/`admin`, and change the password from Settings.
3. Configure each device (humidifier via its own AP setup page, PC/Linux/Android clients via their
   own UI) with `yuki-core`'s address and the token from step 1.
4. New devices show up pending in `yuki-webui` - approve them there.
5. Optionally turn on TLS everywhere once the deployment is otherwise working, per module README.

## Roadmap

- [x] Complete Yuki Core server MVP
- [x] Finalize Yuki Protocol specification (`yuki/1.0`)
- [x] Develop Yuki WebUI with basic device management
- [x] Connect first devices: `yuki-device-pc` & `yuki-device-android`
- [x] Expand device ecosystem (`yuki-device-pc-linux`, `yuki-humidifier`)
- [x] Harden authentication, transport, and secrets handling for the 1.0.0 release
- [ ] Integrate AI-based voice assistant functionality
- [ ] `yuki-device-frame` (FrameOS)

## License

The whole ecosystem - this repository and every module listed above - is licensed under the
**GNU General Public License v3.0 (GPLv3)**.
