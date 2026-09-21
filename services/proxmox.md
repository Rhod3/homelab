# Proxmox VE Hypervisor

Proxmox Virtual Environment (VE) serves as the core compute virtualization platform for the homelab, running on the HP Elite Mini 600 G9.

**Current version**: 9.2 — see [hardware/compute.md](../hardware/compute.md) for host specs.

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
- **Future services** (MQTT/Zigbee2MQTT, AdGuard Home, Immich, Paperless-ngx, etc.): each deployed as its own dedicated LXC or VM, sized and provisioned individually rather than consolidated into a shared container host — see [services/README.md](README.md) for current status.

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

### Service-Level Storage (Media, App Data)

Only Proxmox's own operational data — `vzdump` backups and ISO/template files — is registered as Proxmox-level NFS storage (`nas-backups`, `nas-images` above), as genuine Datacenter → Storage entries.

Data that belongs to an individual service (e.g. Jellyfin's media library, or future services like Immich's photo library or Paperless-ngx's document store) was originally planned to be mounted directly inside that service's own VM/LXC as a guest-level NFS/SMB client, so each service's storage dependency stays self-contained in its own config rather than the host accumulating per-service mount points. That plan still holds for VMs and *privileged* LXCs, but turned out to be a hard dead end for **unprivileged** LXCs — the default and strongly preferred container type for lightweight services (see [services/jellyfin/README.md](jellyfin/README.md)).

**Why**: the Linux kernel's NFS client filesystem doesn't support being mounted from inside an unprivileged user namespace (it lacks the `FS_USERNS_MOUNT` capability that filesystems like `overlay` or `fuse` have). An unprivileged container's "root" only has full capabilities within its own nested namespace, and NFS's mount permission check requires genuine, host-level `CAP_SYS_ADMIN` — something no AppArmor policy or Proxmox `features` flag can grant a truly unprivileged container. This was discovered hands-on while deploying Jellyfin, after extensive troubleshooting ruled out AppArmor, export ACLs, NFS protocol version, and privileged-port settings one by one — see [services/jellyfin/README.md](jellyfin/README.md) for the full trail, and this community write-up for independent confirmation of the same workaround: [Proxmox forum — Mounting NFS share to an unprivileged LXC](https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/).

**Revised pattern for unprivileged LXCs**: the Proxmox host mounts the NFS share itself (as real root, so the kernel restriction doesn't apply) into a plain directory under `/mnt/`, and the mount is passed into the container as a bind-style mount point (`pct set <vmid> -mp0 /mnt/<host-path>,mp=<container-path>`). This is *not* a registered Proxmox storage entry — it doesn't appear under Datacenter → Storage, and each service still gets its own dedicated host mount point rather than sharing one — so it's a narrow, kernel-forced exception to the scoping principle above, not an abandonment of it. The trade-off accepted: the host now carries one mount point per NFS-backed unprivileged service, in exchange for keeping those containers unprivileged.

### Status

- **Synology side**: Done. The `proxmox_backups` and `proxmox_images` shared folders are created with host-restricted NFS permissions — see [hardware/storage.md](../hardware/storage.md).
- **Proxmox side**: Done. The `nas-images` and `nas-backups` NFS storage entries are added under Datacenter → Storage, pointed at the shares above.
- **`vm-disks`**: Done. Created as its own LVM-Thin pool/VG on the secondary 512 GB SSD (467.28 GB), separate from `pve`.
- **Legacy pool cleanup**: Done. Proxmox's default installer had placed a second thin pool (`pve/data`, ~141 GB) on the *primary* NVMe alongside `local` — this was the installer's default behavior when only one disk is selected at install time, not an intentional storage tier. It was removed and its space reclaimed into `pve/root` (now 226 GB), so `local-lvm` no longer exists and the storage list matches the table above exactly, with no unused legacy pool.
