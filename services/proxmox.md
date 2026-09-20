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

## Storage Configuration (Planned)

Proxmox storage is split by content type: latency-sensitive content stays local, static/bulk content moves to the Synology DS420+. See [docs/storage-strategy.md](../docs/storage-strategy.md) for the rationale behind this split.

| Storage Name | Type | Backing | Content Types | Purpose |
| --- | --- | --- | --- | --- |
| `local` | Directory | Primary 256 GB NVMe | Snippets | Proxmox OS (implicit), cloud-init/hook snippets |
| `vm-disks` | LVM-Thin | Secondary 512 GB SSD | Disk image, Container | VM and LXC virtual disks — kept local for I/O performance |
| `nas-templates` | NFS | Synology `ProxmoxTemplates` share | ISO image, Container template | Installer ISOs and LXC templates — static, infrequently read, no benefit from local NVMe |
| `nas-backups` | NFS | Synology `ProxmoxBackups` share | VZDump backup file | Scheduled `vzdump` backups for all VMs/LXCs except Home Assistant |

**Not a Proxmox storage entry**: Home Assistant's own backup engine writes directly to the Synology `HomeAssistantBackups` share (SMB/NFS) from within the HA VM — it does not go through Proxmox's storage layer. See [services/home-assistant/README.md](home-assistant/README.md).

### Why Disk image / Container stay local

Running live VM/LXC disks over NFS would tie VM I/O performance to the network path (currently Gigabit switch — see [hardware/networking.md](../hardware/networking.md)), well below local SSD throughput, and would turn a network hiccup into a VM stall. This keeps compute and storage responsibilities separate per the repo's guiding principles.

### Status

This storage layout is **planned**, not yet implemented. The `ProxmoxTemplates` and `ProxmoxBackups` Synology shares referenced above do not exist yet — see [hardware/storage.md](../hardware/storage.md).
