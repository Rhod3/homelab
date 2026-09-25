# Services Directory

This directory tracks all deployed and planned services within the homelab environment.

---

## Service Status Matrix

| Service | Hosting Target | Status | Documentation |
| --- | --- | --- | --- |
| **Proxmox VE** | HP Elite Mini 600 G9 | Deployed | [services/proxmox.md](proxmox.md) |
| **Home Assistant OS** | Synology VMM → Proxmox VE | Active (Migration Planned) | [services/home-assistant/README.md](home-assistant/README.md) |
| **Jellyfin** | Proxmox VE (LXC, CT 100) | Deployed | [services/jellyfin/README.md](jellyfin/README.md) |
| **AdGuard Home** | Proxmox VE | Planned | [services/adguard-home/README.md](adguard-home/README.md) |
| **Immich** | Proxmox VE | Planned | TBD |
| **Paperless-ngx** | Proxmox VE | Planned | TBD |

---

## Deployment Guidelines

- Add services progressively rather than deploying everything at once.
- Maintain dedicated documentation for complex core services in subdirectories (e.g. [services/home-assistant/README.md](home-assistant/README.md)).
