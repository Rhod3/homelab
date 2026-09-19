# AGENTS.md

## Repository Purpose

This repository documents and evolves a personal homelab.

The current architecture is intentionally simple and low-cost. The long-term goal is to grow into a more capable setup while keeping services reliable, maintainable, and easy to migrate.

Primary components:
- Compute: HP Elite Mini 600 G9 running Proxmox VE
- Storage: Synology DS420+
- Smart home: Home Assistant OS
- Media: Jellyfin, planned
- Networking: ASUS router and Gigabit switch initially, managed VLAN-capable networking later

## Canonical Documentation

Use these files as the source of truth:

- `README.md`: main index and high-level design principles
- `docs/architecture.md`: overall architecture and topology
- `docs/storage-strategy.md`: storage and backup strategy
- `hardware/compute.md`: HP Elite Mini / Proxmox host details
- `hardware/storage.md`: Synology DS420+ details
- `hardware/networking.md`: current and future network design
- `hardware/power.md`: UPS and power planning
- `services/README.md`: service status index
- `services/proxmox.md`: Proxmox notes
- `services/home-assistant/README.md`: Home Assistant setup, migration, and backup policy
- `services/jellyfin/README.md`: Jellyfin planning

## Guiding Principles

When suggesting or implementing changes:

1. Keep the initial setup simple.
2. Avoid recommending new hardware unless there is a clear need.
3. Prefer reliable, boring infrastructure over clever complexity.
4. Keep compute and storage responsibilities separate.
5. Use the Synology mainly for storage, media, and backups.
6. Use Proxmox as the central virtualization layer.
7. Introduce VLANs only when the network is ready for the added complexity.
8. Favor migration-friendly designs.

## Documentation Standards

- Use Markdown files.
- Keep documentation modular.
- Put hardware documentation under `hardware/`.
- Put service documentation under `services/`.
- Put architecture, storage, backup, and planning docs under `docs/`.
- Prefer tables for specs, service status, VLANs, IP plans, and hardware inventories.
- Prefer Mermaid diagrams for topology, backup flows, and architecture diagrams.
- Update `README.md` or `services/README.md` when adding major new documentation.

## Implementation Standards

When adding real configurations in the future:

- Keep service-specific files next to their documentation.
- Use one folder per complex service, for example:
  - `services/home-assistant/`
  - `services/jellyfin/`
  - `services/adguard-home/`
  - `services/immich/`
- Document assumptions, hostnames, storage mounts, ports, and backup strategy.
- Do not invent IP addresses, VLAN IDs, credentials, or hostnames unless explicitly asked.
- Do not include secrets, API keys, passwords, tokens, or private certificates in the repo.
- Use example files for sensitive configs, such as `.env.example`.

## Safety Rules

Before suggesting destructive or risky operations:

- Ask before deleting data, wiping disks, reformatting storage, changing RAID/SHR layouts, or modifying boot devices.
- Ask before changing live network topology, VLAN routing, firewall rules, or DHCP/DNS behavior.
- Clearly distinguish between current state, planned state, and speculative future ideas.
- Prefer reversible migration paths.
- For Home Assistant, prefer backup-and-restore migration over direct VM disk manipulation unless there is a clear reason.

## Service Planning Checklist

When proposing a new service, include:

- Purpose
- Recommended host: VM, LXC, Docker VM, or Synology package
- Storage needs
- Network ports
- Backup requirements
- Dependencies
- Security considerations
- Whether it belongs on Proxmox or Synology
- Documentation file location