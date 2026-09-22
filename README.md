# Homelab Documentation

Welcome to the homelab documentation repository. This repository tracks the architecture, hardware configuration, networking setup, and deployed services for the homelab environment.

The goal of this project is to start with a simple, low-cost setup and progressively evolve it into a robust, high-availability homelab.

---

## Architecture Quick Links

- **Overview & Strategy**: [docs/architecture.md](docs/architecture.md)
- **Storage & Backup Strategy**: [docs/storage-strategy.md](docs/storage-strategy.md)
- **Network Strategy**: [docs/network-strategy.md](docs/network-strategy.md)
- **Hardware Overview**:
  - [hardware/compute.md](hardware/compute.md) - HP Elite Mini 600 G9 (Proxmox VE Host)
  - [hardware/storage.md](hardware/storage.md) - Synology DS420+ NAS
  - [hardware/networking.md](hardware/networking.md) - Network topology & future VLAN layout
  - [hardware/power.md](hardware/power.md) - Power management & UPS plan
  - [hardware/mini-rack.md](hardware/mini-rack.md) - KWS Rack v2, 3D-printed 10" enclosure (planned)
- **Services Directory**:
  - [services/README.md](services/README.md) - Service index & status
  - [services/proxmox.md](services/proxmox.md) - Hypervisor configuration
  - [services/home-assistant/README.md](services/home-assistant/README.md) - Home Assistant OS setup & migration
  - [services/jellyfin/README.md](services/jellyfin/README.md) - Jellyfin media server & HW transcoding
