# Yuki System

Yuki System is an open-source modular ecosystem for smart devices, voice assistants, and IoT integration.  
It allows users to control multiple devices (PCs, mobile devices, smart speakers, IoT devices) through a central server, using a unified protocol.

> **Note:** This project is in early development. Many components are still being built and may change rapidly.

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
              ├── yuki-device-pc
              ├── yuki-device-android
              ├── yuki-device-clock
              ├── yuki-device-light
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
| `yuki-device-pc` | PC Client | Executes commands from the server |
| `yuki-device-android` | Android Client | Executes commands and notifications on Android devices |

---

## Getting Started

Currently, Yuki System is under active development.  

> Code for the core server and device clients will be made public incrementally as development progresses.

---

## Roadmap

- [ ] Complete Yuki Core server MVP  
- [ ] Finalize Yuki Protocol specification  
- [ ] Develop Yuki WebUI with basic device management  
- [ ] Connect first devices: `yuki-device-pc` & `yuki-speaker`  
- [ ] Expand device ecosystem (`yuki-device-android`, IoT devices, robots)  
- [ ] Integrate AI-based voice assistant functionality  

---

---

## License

This repository and its documentation are licensed under the **MIT License**.  
See [LICENSE](./LICENSE) for details.

---

## Notes

- Yuki System is modular: each component has its own repository.  
- This meta repository is designed to give an overview and central documentation.  