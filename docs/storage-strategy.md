# Storage Strategy

This document outlines the allocation of storage and backup workflows across the homelab infrastructure.

---

## Workload Allocation

```mermaid
flowchart LR
    subgraph Compute Storage - HP Elite Mini 600 G9
        NVMe[Primary NVMe SSD<br/>Boot]
        SSD2[Secondary SSD]
        NVMe --> PVE[Proxmox OS]
        NVMe --> AppData[Fast App Runtime Data & DBs]
        SSD2 --> VMDisks[VM / Container Virtual Disks]
    end

    subgraph Bulk Storage - Synology DS420+
        Volume[RAID/SHR Volume]
        Volume --> Media[Movies, TV Shows, Photos]
        Volume --> LargeData[Large Application Storage]
        Volume --> Backups[Service & HA Backups]
        Volume --> Templates[Proxmox ISO Images & LXC Templates]
    end
```

### 1. Compute Node (HP Elite Mini 600 G9)
- **Capacity**: see [hardware/compute.md](../hardware/compute.md) for drive models and sizes.
- **Purpose**: Primary NVMe hosts the Proxmox OS, system logs, and Proxmox snapshots. Secondary SSD hosts VM/LXC virtual disks and fast app runtime data/DBs.
- **Content kept local**: only latency-sensitive content — VM/LXC disk images and Proxmox snippets. Static content (ISO images, container templates) is offloaded to the Synology; see the full Proxmox storage breakdown in [services/proxmox.md](../services/proxmox.md#storage-configuration).

### 2. NAS (Synology DS420+)
- **Capacity**: see [hardware/storage.md](../hardware/storage.md) for volume size and shared folders.
- **Current Use**: Hosting media files and temporary Home Assistant VM storage.
- **Future Role**: Pure NAS dedicated to bulk media (movies, TV shows, photos), central backup target, network storage shares (NFS/SMB), and static Proxmox content (ISO images, LXC templates).

---

## Backup Strategy

To ensure seamless disaster recovery, application-level backups are decoupled from hypervisor VM disk images.

```mermaid
flowchart TD
    HA[Home Assistant OS] -->|Built-in Auto Backup| HABackup[HA Backup Package]
    HABackup -->|Store via SMB/NFS| SynologyShare[Synology 'homeassistant_backups' Share]
    SynologyShare -->|Hyper Backup| Dest[Hyper Backup Target]
    Dest --> USB[External USB Drive]
    Dest --> Offsite[Cloud / Remote NAS]
```

### Key Backup Rules
- **Application-Level Backups**: Use Home Assistant's built-in backup engine to generate full system backups automatically.
- **Synology Shared Folder & Access**: See [hardware/storage.md](../hardware/storage.md) for the dedicated share and service account.
- **Secondary Backup (Hyper Backup)**: Back up the `.tar` backup files from Synology to external media or offsite cloud storage. Do *not* rely solely on VM image snapshots.

---

## General VM/LXC Backup Strategy

Home Assistant's backup approach above is deliberately different from a generic VM backup: it is application-level and portable, which matters because Home Assistant is safety-relevant. Other services don't share that constraint, so they follow a staged approach instead.

### Baseline: Proxmox vzdump

Until a service has a proven need for something better, every VM/LXC other than Home Assistant is protected by Proxmox's built-in `vzdump`, scheduled and targeted at a Synology share.

```mermaid
flowchart TD
    VMs[Proxmox VMs / LXCs<br/>Jellyfin, Docker VM, etc.] -->|Scheduled vzdump| VZDump[vzdump Backup Files]
    VZDump -->|Store via NFS/SMB| SynologyShare[Synology 'proxmox_backups' Share]
    SynologyShare -->|Hyper Backup| Dest[Hyper Backup Target]
    Dest --> USB[External USB Drive]
    Dest --> Offsite[Cloud / Remote NAS]
```

- **Scope**: whole VM/LXC disk and config snapshot. Crash-consistent, not application-consistent.
- **Target**: dedicated `proxmox_backups` share on the Synology DS420+, separate from `homeassistant_backups`.
- **Secondary backup**: reuse the existing Hyper Backup path to external/offsite storage rather than introducing a second mechanism.

### Upgrading to App-Level Backups

`vzdump` is a safety net, not the end state. As each planned service is actually deployed, evaluate whether it warrants an app-level backup instead of, or alongside, `vzdump`, based on what the application supports:

| Service | Native Backup Support | Recommended Approach |
| --- | --- | --- |
| **Paperless-ngx** | `document_exporter` / `document_importer` (full portable export) | App-level export, same pattern as Home Assistant |
| **Immich** | `pg_dump` / `pg_dumpall` for metadata only | DB dump plus filesystem sync of the library folder |
| **Jellyfin** | No native export; media already lives on the NAS | Periodic copy of the config/DB directory only |
| **Docker VM, MQTT, AdGuard Home, etc.** | No native export | `vzdump` baseline is sufficient |

This table is not a commitment to build these integrations now. It exists so that when a service moves from "Planned" to "Deployed" in [services/README.md](../services/README.md#service-status-matrix), its backup method is a deliberate choice rather than an afterthought.
