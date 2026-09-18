# Yuki System

Yuki System is an open-source modular ecosystem for smart devices, voice assistants, and IoT integration.  
It allows users to control multiple devices (PCs, mobile devices, smart speakers, IoT devices) through a central server, using a unified protocol.

> This README is a short overview. For architecture, the full protocol reference, and the security model, see [DOCUMENTATION.md](DOCUMENTATION.md) ([русский](DOCUMENTATION.ru.md), [日本語](DOCUMENTATION.ja.md)). This file is also available in [русский](readme.ru.md) and [日本語](readme.ja.md).

---

## Overview

Yuki System consists of multiple modules, each with a specific role:

```text
yuki-system (meta / this repository)
        │
        ▼
yuki-core (server / brain)
        │
        ├── yuki-protocol (communication protocol & SDK)
        │
        ├── yuki-webui (web interface)
        │
        └── Devices (all clients connect via yuki-protocol)
              ├── yuki-device-pc (Windows)
              ├── yuki-device-pc-linux (Linux)
              ├── yuki-device-android
              ├── yuki-humidifier (ESP32 smart humidifier)
              ├── yuki-device-frame (FrameOS)
              └── yuki-speaker (ESP32 smart speaker)
```
## Repository Structure

This repository is the **meta repository**. It provides:

- Documentation for the Yuki System ecosystem  
- Architecture diagrams and component descriptions  
- Roadmap and development guide  
- Links to all individual repositories  

> Actual code resides in separate repositories for each module.

---

## Components

| Module | Role | Description |
|--------|------|-------------|
| `yuki-core` | Server / Brain | Handles command routing, AI processing, and device management |
| `yuki-protocol` | Protocol & SDK | Defines message formats, command types, and SDKs for connecting devices |
| `yuki-webui` | Web Interface | Allows users to monitor devices, send commands, and manage automation |
| `yuki-speaker` | ESP32 Smart Speaker | Provides voice input/output, connects to the server |
| `yuki-device-pc` | Windows Client | Executes commands from the server |
| `yuki-device-pc-linux` | Linux Client | Same feature set as `yuki-device-pc`, for Linux desktops |
| `yuki-device-android` | Android Client | Executes commands and notifications on Android devices |
| `yuki-humidifier` | ESP32 Smart Humidifier | Wi-Fi humidifier device, controllable through the server |

---

## Getting Started

Start `yuki-core`, then `yuki-webui`, then configure whichever device clients you need - each
module's own README has exact install/run instructions. See [DOCUMENTATION.md](DOCUMENTATION.md)
for a full walkthrough and the security model.

---

## Roadmap

- [x] Complete Yuki Core server MVP  
- [x] Finalize Yuki Protocol specification (`yuki/1.0`)  
- [x] Develop Yuki WebUI with basic device management  
- [x] Connect first devices: `yuki-device-pc` & `yuki-device-android`  
- [x] Expand device ecosystem (`yuki-device-pc-linux`, `yuki-humidifier`)  
- [ ] Integrate AI-based voice assistant functionality  
- [ ] `yuki-device-frame` (FrameOS)  

---

---

## License

This repository and its documentation are licensed under the **MIT License**.  
See [LICENSE](./LICENSE) for details.

---

## Notes

- Yuki System is modular: each component has its own repository.  
- This meta repository is designed to give an overview and central documentation.  