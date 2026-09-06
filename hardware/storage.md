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

## Primary Shared Folders

- `Media/`: Contains Movies, TV shows, and other streaming media.
- `HomeAssistantBackups/`: Dedicated share reserved for automated Home Assistant backups.

---

## Access Control & Security

- **Backup Account**: A dedicated Synology user account named `ha-backup` is configured.
- **Permissions**: Restricted exclusively to read/write access on the `HomeAssistantBackups` shared folder.

---

## Migration Strategy

Currently, the Home Assistant OS VM runs on Synology VMM. 

**Target State**:
- Migrate the Home Assistant VM off Synology VMM onto Proxmox VE on the HP Elite Mini host.
- Re-purpose the Synology DS420+ strictly as a high-capacity storage and backup target.
