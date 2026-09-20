# Storage Node: Synology DS420+

The Synology DS420+ acts as the central Network Attached Storage (NAS) for the homelab.

---

## Specifications

| Component | Detail |
| --- | --- |
| **Model** | Synology DS420+ |
| **Volume Size** | 6 TB existing storage volume |
| **Hypervisor (Temporary)** | Synology Virtual Machine Manager (VMM) |
| **Current Services** | Media storage, temporary Home Assistant VM host |

---

## Physical Drives

| Model | Capacity | Family | Interface | Speed |
| --- | --- | --- | --- | --- |
| ST4000VN008-2DR166 | 4 TB | Seagate IronWolf (NAS) | SATA 6 Gb/s | 5900 RPM |
| ST4000VN008-2DR166 | 4 TB | Seagate IronWolf (NAS) | SATA 6 Gb/s | 5900 RPM |
| WD30EFRX-68EUZN0 | 3 TB | WD Red (NAS) | SATA 6 Gb/s | 5400 RPM (IntelliPower) |

---

## Primary Shared Folders

- `Media/`: Contains Movies, TV shows, and other streaming media.
- `HomeAssistantBackups/`: Dedicated share reserved for automated Home Assistant backups.
- `ProxmoxBackups/` (**Planned**, not yet created): NFS target for scheduled `vzdump` backups of all VMs/LXCs except Home Assistant. See [services/proxmox.md](../services/proxmox.md).
- `ProxmoxTemplates/` (**Planned**, not yet created): NFS target for Proxmox ISO images and LXC container templates, offloaded from local compute storage. See [services/proxmox.md](../services/proxmox.md).

---

## Access Control & Security

Access control follows a least-privilege pattern, scoped per share. The mechanism differs by protocol:

### SMB/CIFS Shares (user-account based)

- **`HomeAssistantBackups`**: A dedicated Synology user account named `ha-backup` is configured, restricted exclusively to read/write access on this share. No other share access is granted.
- This is the template to reuse for any future app-level backup export (e.g. Paperless-ngx, Immich) that needs its own SMB/CIFS credential — see the backup approach table in [docs/storage-strategy.md](../docs/storage-strategy.md).

### NFS Shares (host-restricted, Proxmox-planned)

`ProxmoxBackups/` and `ProxmoxTemplates/` (both **Planned**) are consumed by Proxmox over NFS rather than SMB, so access control is host-based rather than account-based:

- **Export restriction**: Each NFS export should be restricted to the Proxmox host's IP only — no other client should be permitted to mount these shares.
- **Root squash**: Enable squash mapping so the Proxmox host's root user is mapped to a non-privileged user on the Synology side, rather than granted root-equivalent access to the share.
- **Scope**: Each share should only expose the content type it's named for (backups vs. ISO/templates) — avoid combining them into a single general-purpose export.

### `Media/` (current state, not access-restricted)

- No dedicated account or host restriction is currently configured — it is an open LAN share for general/family access. This is a deliberate current-state choice, not an oversight.
- If a service like Jellyfin (planned) later needs its own read access, consider giving it a scoped read-only account rather than relying on open access, at that point.

---

## Migration Strategy

Currently, the Home Assistant OS VM runs on Synology VMM. 

**Target State**:
- Migrate the Home Assistant VM off Synology VMM onto Proxmox VE on the HP Elite Mini host.
- Re-purpose the Synology DS420+ strictly as a high-capacity storage and backup target.
