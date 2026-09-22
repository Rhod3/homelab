# Storage Node: Synology DS420+

The Synology DS420+ acts as the central Network Attached Storage (NAS) for the homelab.

---

## Specifications

| Component | Detail |
| --- | --- |
| **Model** | Synology DS420+ |
| **Volume Size** | 6.1 TB (Btrfs, SHR) — see [Storage Pool & Volume](#storage-pool--volume) |
| **Hypervisor (Temporary)** | Synology Virtual Machine Manager (VMM) |
| **Current Services** | Media storage, temporary Home Assistant VM host |

---

## Network Configuration

| Setting | Value |
| --- | --- |
| IP Address | 192.168.0.10 (static, set in DSM) |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.0.1 |
| DNS | 192.168.0.1 |

Static IP is configured directly in DSM (Control Panel → Network → Network Interface) rather than via a router DHCP reservation, chosen to be outside the router's DHCP pool — see [hardware/networking.md](networking.md#router-configuration-as-observed) for the DHCP pool range and router configuration.

---

## Physical Drives

| Model | Capacity | Family | Interface | Speed |
| --- | --- | --- | --- | --- |
| ST4000VN008-2DR166 | 4 TB | Seagate IronWolf (NAS) | SATA 6 Gb/s | 5900 RPM |
| ST4000VN008-2DR166 | 4 TB | Seagate IronWolf (NAS) | SATA 6 Gb/s | 5900 RPM |
| WD30EFRX-68EUZN0 | 3 TB | WD Red (NAS) | SATA 6 Gb/s | 5400 RPM (IntelliPower) |

---

## Storage Pool & Volume

| Setting | Value |
| --- | --- |
| **Storage Pool** | Storage Pool 1 — all 3 physical drives above |
| **RAID Type** | Synology Hybrid RAID (SHR), 1-drive fault tolerance |
| **Pool Capacity** | 6.4 TB allocated, 0 Bytes free at the pool level (fully assigned to Volume 1) |
| **Multiple Volume Support** | No — single volume spans the entire pool |
| **Volume Encryption** | Disabled |
| **Volume** | Volume 1 |
| **File System** | Btrfs |
| **Volume Capacity** | 6.1 TB total — 4.9 TB used, 1.2 TB free (~80% utilized) |
| **Data Scrubbing** | Scheduled periodically; last completed 2026-08-27 |

> **Capacity Warning**: DSM currently flags Volume 1 as running low on free space (~20% free) and recommends adding or replacing drives with larger capacity. Factor this in now that `proxmox_backups` and `proxmox_images` are provisioned below — if backup/template volume grows significant, revisit capacity planning rather than assuming headroom.

Since the volume is confirmed Btrfs, **data checksums are supported** and can be enabled per shared folder (Btrfs-only DSM feature). Recommended on `homeassistant_backups`, `proxmox_backups`, and `proxmox_images` for corruption detection on backup data; optional on `tv`. Note checksums add metadata overhead, which is worth weighing given the capacity warning above.

---

## Primary Shared Folders

- `tv/`: Contains streaming media — the only media share, used by the planned Jellyfin deployment (see [services/jellyfin/README.md](../services/jellyfin/README.md)).
- `homeassistant_backups/`: Dedicated share reserved for automated Home Assistant backups.
- `proxmox_backups/` (**Created**): NFS target for scheduled `vzdump` backups of all VMs/LXCs except Home Assistant. See [services/proxmox.md](../services/proxmox.md#storage-configuration).
- `proxmox_images/` (**Created**): NFS target for Proxmox ISO images and LXC container templates, offloaded from local compute storage. See [services/proxmox.md](../services/proxmox.md#storage-configuration).

All four shares use `snake_case` naming — consistent across the board.

---

## Access Control & Security

Access control follows a least-privilege pattern, scoped per share. The mechanism differs by protocol:

### SMB/CIFS Shares (user-account based)

- **`homeassistant_backups`**: A dedicated Synology user account named `ha-backup` is configured, restricted exclusively to read/write access on this share. No other share access is granted.
- This is the template to reuse for any future app-level backup export (e.g. Paperless-ngx, Immich) that needs its own SMB/CIFS credential — see the backup approach table in [docs/storage-strategy.md](../docs/storage-strategy.md#upgrading-to-app-level-backups).

### NFS Shares (host-restricted)

`proxmox_backups/` and `proxmox_images/` (**Created**, NFS permissions configured) are consumed by Proxmox over NFS rather than SMB, so access control is host-based rather than account-based:

- **Export restriction**: Each NFS export is restricted to the Proxmox host's static IP (192.168.0.2 — see [hardware/compute.md](compute.md#network-configuration)) only — no other client is permitted to mount these shares.
- **Root squash**: Squash mapping is enabled so the Proxmox host's root user is mapped to a non-privileged user on the Synology side, rather than granted root-equivalent access to the share.
- **Scope**: Each share only exposes the content type it's named for (backups vs. ISO/templates) — not combined into a single general-purpose export.

### `tv` (current state, NFS enabled and host-restricted)

- **NFS is enabled** on this share, exported **Read Only** to the **Proxmox host** (`192.168.0.2`) only. It remains accessible over SMB/CIFS for general/family LAN access as before.
- **Squash: Map all users to admin.** Required because NFS permission checks happen server-side against the caller's real UID — an unprivileged LXC's mapped UID (~100000+, not the container's apparent `root`) doesn't correspond to any NAS user, so requests were rejected outright regardless of the share's permissive (`777`) file modes. Squashing every caller to one known, readable account sidesteps the mismatch. Safe here since the export is read-only.
- The export authorizes the **Proxmox host's** IP, not the Jellyfin container's (`192.168.0.30`), because the host mounts the share and passes it into the container via a bind mount point — the container cannot mount NFS directly (a Linux kernel restriction, unrelated to this export's config) — see [services/proxmox.md](../services/proxmox.md#service-level-storage-media-app-data) and [services/jellyfin/README.md](../services/jellyfin/README.md#deployment-specs) for the full story.

---

## Migration Strategy

The Home Assistant OS VM currently runs on Synology VMM and is planned to migrate to Proxmox VE, after which the Synology DS420+ becomes strictly a storage and backup target. See [services/home-assistant/README.md](../services/home-assistant/README.md) for the full migration plan.
