# Services Directory

This directory tracks all deployed and planned services within the homelab environment.

---

## Service Status Matrix

| Service | Hosting Target | Status | Documentation |
| --- | --- | --- | --- |
| **Proxmox VE** | HP Elite Mini 600 G9 | Deployed | [services/proxmox.md](services/proxmox.md) |
| **Home Assistant OS** | Synology VMM → Proxmox VE | Active (Migration Planned) | [services/home-assistant/README.md](services/home-assistant/README.md) |
| **Jellyfin** | Proxmox VE (LXC, CT 100) | Deployed | [services/jellyfin/README.md](services/jellyfin/README.md) |
| **Docker Engine** | Proxmox VE | Planned | TBD |
| **MQTT / Zigbee2MQTT** | Proxmox VE | Planned | TBD |
| **AdGuard Home / Pi-hole** | Proxmox VE | Planned | TBD |
| **Immich** | Proxmox VE | Planned | TBD |
| **Paperless-ngx** | Proxmox VE | Planned | TBD |

---

## Deployment Guidelines

- Add services progressively rather than deploying everything at once.
- Maintain dedicated documentation for complex core services in subdirectories (e.g. [services/home-assistant/README.md](services/home-assistant/README.md)).
