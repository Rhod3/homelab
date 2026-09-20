# Proxmox VE Hypervisor

Proxmox Virtual Environment (VE) serves as the core compute virtualization platform for the homelab, running on the HP Elite Mini 600 G9.

---

## Configuration Summary

- **Host Hardware**: HP Elite Mini 600 G9 — see [hardware/compute.md](../hardware/compute.md) for full specs
- **Deployment Strategy**:
  - **Virtual Machines (VMs)**: Used for workloads requiring dedicated OS kernels, full isolation, or specialized OS bundles (e.g. Home Assistant OS).
  - **LXC Containers**: Used for lightweight applications sharing the host Linux kernel (e.g. Jellyfin, utility services).

---

## Hosted Workloads

- **Home Assistant OS VM**: (Migration target from Synology VMM)
- **Jellyfin**: Planned LXC container with Intel Quick Sync GPU passthrough.
- **Docker VM**: Planned general-purpose Linux VM running containerized services.

---

## Storage Configuration

Proxmox storage is split by content type: latency-sensitive content stays local, static/bulk content moves to the Synology DS420+. See [docs/storage-strategy.md](../docs/storage-strategy.md) for the rationale behind this split.

| Storage Name | Type | Backing | Content Types | Purpose |
| --- | --- | --- | --- | --- |
| `local` | Directory | Primary 256 GB NVMe (226 GB `pve/root` LV) | Snippets | Proxmox OS (implicit), cloud-init/hook snippets |
| `vm-disks` | LVM-Thin | Secondary 512 GB SSD | Disk image, Container | VM and LXC virtual disks — kept local for I/O performance |
| `nas-images` | NFS | Synology `proxmox_images` share | ISO image, Container template | Installer ISOs and LXC templates — static, infrequently read, no benefit from local NVMe |
| `nas-backups` | NFS | Synology `proxmox_backups` share | VZDump backup file | Scheduled `vzdump` backups for all VMs/LXCs except Home Assistant |

**Not a Proxmox storage entry**: Home Assistant's own backup engine writes directly to the Synology `homeassistant_backups` share (SMB/NFS) from within the HA VM — it does not go through Proxmox's storage layer. See [services/home-assistant/README.md](home-assistant/README.md).

### Why Disk image / Container stay local

Running live VM/LXC disks over NFS would tie VM I/O performance to the network path (currently Gigabit switch — see [hardware/networking.md](../hardware/networking.md)), well below local SSD throughput, and would turn a network hiccup into a VM stall. This keeps compute and storage responsibilities separate per the repo's guiding principles.

### Status

- **Synology side**: Done. The `proxmox_backups` and `proxmox_images` shared folders are created with host-restricted NFS permissions — see [hardware/storage.md](../hardware/storage.md).
- **Proxmox side**: Done. The `nas-images` and `nas-backups` NFS storage entries are added under Datacenter → Storage, pointed at the shares above.
- **`vm-disks`**: Done. Created as its own LVM-Thin pool/VG on the secondary 512 GB SSD (467.28 GB), separate from `pve`.
- **Legacy pool cleanup**: Done. Proxmox's default installer had placed a second thin pool (`pve/data`, ~141 GB) on the *primary* NVMe alongside `local` — this was the installer's default behavior when only one disk is selected at install time, not an intentional storage tier. It was removed and its space reclaimed into `pve/root` (now 226 GB), so `local-lvm` no longer exists and the storage list matches the table above exactly, with no unused legacy pool.
