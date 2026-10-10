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
- **Jellyfin**: Deployed as unprivileged LXC (CT 100) with Intel Quick Sync GPU passthrough — see [services/jellyfin/README.md](jellyfin/README.md).
- **TeslaMate**: Deployed as unprivileged LXC (CT 102, `192.168.0.32`) from the community-scripts TeslaMate script — see [services/teslamate/README.md](teslamate/README.md).
- **Future services** (MQTT/Zigbee2MQTT, AdGuard Home, Immich, Paperless-ngx, etc.): each deployed as its own dedicated LXC or VM, sized and provisioned individually rather than consolidated into a shared container host — see [services/README.md](README.md#service-status-matrix) for current status.
- **Arr Stack Docker VM** (deployed, VM 101 `arr-stack`, `192.168.0.31`, Debian 13 from the cloud image + cloud-init): the one deliberate exception to the rule above — a single Debian VM running Docker Compose for Prowlarr, Radarr, Sonarr, qBittorrent, and related apps. Justified because the stack is one logical service: the apps must share a single filesystem for hardlinks, are wired together by API keys, and are deployed and updated together. A VM rather than LXC(s) because the apps need **read/write** NFS access, which unprivileged LXCs can't mount (see below) and which the host-bind-mount workaround complicates with UID shifting. This is not a general-purpose Docker host; other services still get their own LXC/VM. See [services/arr-stack/README.md](arr-stack/README.md).

---

## Storage Configuration

Proxmox storage is split by content type: latency-sensitive content stays local, static/bulk content moves to the Synology DS420+. See [docs/storage-strategy.md](../docs/storage-strategy.md) for the rationale behind this split.

| Storage Name | Type | Backing | Content Types | Purpose |
| --- | --- | --- | --- | --- |
| `local` | Directory | Primary 256 GB NVMe (226 GB `pve/root` LV) | Snippets | Proxmox OS (implicit), cloud-init/hook snippets |
| `vm-disks` | LVM-Thin | Secondary 512 GB SSD | Disk image, Container | VM and LXC virtual disks — kept local for I/O performance |
| `nas-images` | NFS | Synology `proxmox_images` share | ISO image, Container template, Import | Installer ISOs, LXC templates and cloud disk images (e.g. the Debian 13 `genericcloud` image used for VM 101) — static, infrequently read, no benefit from local NVMe. **Import** was added for the arr stack VM: it lets the GUI's *Add → Import Hard Disk* copy a downloaded `qcow2` onto `vm-disks` |
| `nas-backups` | NFS | Synology `proxmox_backups` share | VZDump backup file | Scheduled `vzdump` backups for all VMs/LXCs except Home Assistant — see [Backup Jobs](#backup-jobs) |

**Not a Proxmox storage entry**: Home Assistant's own backup engine writes directly to the Synology `homeassistant_backups` share (SMB/NFS) from within the HA VM — it does not go through Proxmox's storage layer. See [services/home-assistant/README.md](home-assistant/README.md).

### Why Disk image / Container stay local

Running live VM/LXC disks over NFS would tie VM I/O performance to the network path (currently Gigabit switch — see [hardware/networking.md](../hardware/networking.md#current-topology)), well below local SSD throughput, and would turn a network hiccup into a VM stall. This keeps compute and storage responsibilities separate per the repo's guiding principles.

### Service-Level Storage (Media, App Data)

Only Proxmox's own operational data — `vzdump` backups and ISO/template files — is registered as Proxmox-level NFS storage (`nas-backups`, `nas-images` above), as genuine Datacenter → Storage entries.

Data that belongs to an individual service (e.g. Jellyfin's media library, or future services like Immich's photo library or Paperless-ngx's document store) was originally planned to be mounted directly inside that service's own VM/LXC as a guest-level NFS/SMB client, so each service's storage dependency stays self-contained in its own config rather than the host accumulating per-service mount points. That plan still holds for VMs and *privileged* LXCs, but turned out to be a hard dead end for **unprivileged** LXCs — the default and strongly preferred container type for lightweight services (see [services/jellyfin/README.md](jellyfin/README.md)).

**Why**: the Linux kernel's NFS client filesystem doesn't support being mounted from inside an unprivileged user namespace (it lacks the `FS_USERNS_MOUNT` capability that filesystems like `overlay` or `fuse` have). An unprivileged container's "root" only has full capabilities within its own nested namespace, and NFS's mount permission check requires genuine, host-level `CAP_SYS_ADMIN` — something no AppArmor policy or Proxmox `features` flag can grant a truly unprivileged container. This was discovered hands-on while deploying Jellyfin, after extensive troubleshooting ruled out AppArmor, export ACLs, NFS protocol version, and privileged-port settings one by one — see [services/jellyfin/README.md](jellyfin/README.md) for the full trail, and this community write-up for independent confirmation of the same workaround: [Proxmox forum — Mounting NFS share to an unprivileged LXC](https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/).

**Revised pattern for unprivileged LXCs**: the Proxmox host mounts the NFS share itself (as real root, so the kernel restriction doesn't apply) into a plain directory under `/mnt/`, and the mount is passed into the container as a bind-style mount point (`pct set <vmid> -mp0 /mnt/<host-path>,mp=<container-path>`). This is *not* a registered Proxmox storage entry — it doesn't appear under Datacenter → Storage, and each service still gets its own dedicated host mount point rather than sharing one — so it's a narrow, kernel-forced exception to the scoping principle above, not an abandonment of it. The trade-off accepted: the host now carries one mount point per NFS-backed unprivileged service, in exchange for keeping those containers unprivileged.

### Status

- **Synology side**: Done. The `proxmox_backups` and `proxmox_images` shared folders are created with host-restricted NFS permissions — see [hardware/storage.md](../hardware/storage.md#nfs-shares-host-restricted).
- **Proxmox side**: Done. The `nas-images` and `nas-backups` NFS storage entries are added under Datacenter → Storage, pointed at the shares above.
- **`vm-disks`**: Done. Created as its own LVM-Thin pool/VG on the secondary 512 GB SSD (467.28 GB), separate from `pve`.
- **Legacy pool cleanup**: Done. Proxmox's default installer had placed a second thin pool (`pve/data`, ~141 GB) on the *primary* NVMe alongside `local` — this was the installer's default behavior when only one disk is selected at install time, not an intentional storage tier. It was removed and its space reclaimed into `pve/root` (now 226 GB), so `local-lvm` no longer exists and the storage list matches the table above exactly, with no unused legacy pool.

---

## Backup Jobs

Scheduled `vzdump` jobs, configured under **Datacenter → Backup** and written to the `nas-backups` storage above. This is the baseline described in [docs/storage-strategy.md](../docs/storage-strategy.md#baseline-proxmox-vzdump).

| Setting | Value | Notes |
| --- | --- | --- |
| Node | `pve` | |
| Guests | CT 100 (Jellyfin), VM 101 (arr stack, first archive 2026-09-30), CT 102 (TeslaMate, added 2026-10-03) | Home Assistant is deliberately excluded — it uses its own app-level backups. VM 101 was added to this job rather than given its own `snapshot` job: one tested job to maintain, and a clean shutdown keeps the arr apps' SQLite DBs consistent; downloads just pause for ~1 min and the VPN reconnects |
| Storage | `nas-backups` | |
| Mode | `stop` | Clean shutdown before the archive is taken, so Jellyfin's SQLite DB is consistent rather than crash-consistent. Costs ~30 s of downtime per run |
| Compression | ZSTD | |
| Notes template | `{{guestname}}` | Labels each archive with the guest name in the Backups view |
| Retention | `keep-daily=7, keep-weekly=4, keep-monthly=3, keep-yearly=2` | Set node-wide in `/etc/vzdump.conf` (`prune-backups`), not on the job or the storage. At most 16 archives per guest |
| Notifications | Notification system → `mail-to-root` | Mail goes to the email address set on `root@pam` |
| Schedule | Daily at `05:00` | Chosen as a time when nobody is streaming, since `stop` mode briefly takes Jellyfin offline |

### How retention works

Retention (`prune-backups`) is applied **per guest, per storage**, right after each backup of that guest finishes. Each `keep-*` rule keeps the newest backup in each of the last N periods (days, ISO weeks, months, years) that actually contain a backup, so missed nights don't use up slots. Rules are applied from shortest to longest period, and each one only considers backups older than those already kept. Anything no rule keeps is deleted.

- **Precedence**: a retention set on the job wins over one set in `/etc/vzdump.conf`, which in turn wins over the storage's own **Backup Retention** setting. If nothing is set anywhere, the default is `keep-all`.
- **Current choice — node-wide**: retention lives in `/etc/vzdump.conf` on `pve`, next to the `tmpdir` setting below. It therefore applies to every backup this node writes, to any storage, unless a job sets its own. Trade-off: it isn't visible in the storage's or the job's Retention tab (the storage tab can show different values that are silently overridden), and it would have to be repeated on any future second node. Check it with `grep prune /etc/vzdump.conf`.
- **Manual backups count**: a manual `vzdump` run on the same day as the scheduled job gets pruned by `keep-daily`, because only the newest backup of each day is kept. Mark pre-upgrade backups **Protected** (Backups view → Edit) to exempt them.
- **Preview before changing values**: `pvesm prune-backups nas-backups --vmid <id> --dry-run` lists each archive as `keep` or `remove` without deleting anything.

### Node-wide setting: `tmpdir: /var/tmp`

Set in `/etc/vzdump.conf` on `pve`. It is required for backing up **unprivileged LXCs** to `nas-backups`.

- **Why**: without it, `vzdump` stages temporary files in a `.tmp` folder next to the archive, on the NFS share. The share has root squash enabled (see [hardware/storage.md](../hardware/storage.md#nfs-shares-host-restricted)), so the NAS records that folder as owned by the squashed user rather than root. `tar` for an unprivileged container runs as the container's mapped root (UID 100000), which the NAS doesn't recognize, so the first CT 100 backup failed with `tar: …/vzdump-lxc-100-….tmp: Cannot open: Permission denied`. Pointing `tmpdir` at local disk fixes this without weakening root squash, which was the other option considered and rejected.
- **Impact in `stop`/`snapshot` mode**: negligible. The staging area only holds the container's config files (`pct.conf`, `pct.fw`, a few KB), and the archive itself still goes straight to `nas-backups`.
- **Risk — suspend mode**: in `suspend` mode, `vzdump` `rsync`s the **entire container filesystem** into `tmpdir`. That would land on `pve/root`, the filesystem Proxmox itself runs on, and a large container could fill it. `vzdump` falls back to suspend automatically when `snapshot` mode is requested on storage that can't take snapshots. Watch for `trying 'suspend' mode instead` in backup logs. `vm-disks` is LVM-thin and supports snapshots, so current jobs are not affected.
- **Why `/var/tmp` and not `/tmp`**: `/var/tmp` is on disk and Debian doesn't clear it at boot. `/tmp` may be RAM-backed on newer Debian-based Proxmox releases.

### Verification

- **First successful run**: 2026-09-27. CT 100: 3.3 GiB read, 1.81 GB archive, guest back online after 30 s. The log confirmed `mp0` (`/mnt/tv`) was excluded as a bind mount, so media is never copied into backups.
- **Test restore**: passed on 2026-09-27. The 2026-09-27 archive was restored as CT 900 on `vm-disks`, and Jellyfin started with its data intact: `jellyfin.db` was present with an empty `-wal` file, confirming `stop` mode captured a fully written DB. Procedure, reusable for any LXC:
  ```bash
  pct restore 900 /mnt/pve/nas-backups/dump/<archive>.tar.zst --storage vm-disks
  pct set 900 --delete net0     # the copy keeps the original's static IP and MAC, so it must not join the network
  pct start 900
  pct exec 900 -- systemctl status <service>
  pct stop 900 && pct destroy 900   # destroy refuses a running container
  ```
  Expected noise: with no network, apps log connection errors, e.g. Jellyfin's plugin repository check throws an `HttpClient` stack trace. This is harmless.
- **VM 101 (arr stack)**: nightly archives present since 2026-09-30 (`pvesm list nas-backups --vmid 101`), ~2.8 GB each once the stack was configured. Only the VM's own disk is captured — the `/data` NFS mount of the `tv` share is not a VM disk, so media is never included. The 05:00 shutdown/restart was confirmed harmless: the NFS mount, all containers and the VPN tunnel came back on their own. A test restore has not been done yet — see [services/arr-stack/README.md](arr-stack/README.md#future-work).
- **CT 102 (TeslaMate)**: first archive 2026-10-03 (5.0 GiB of container data). Test restore passed the same day with the procedure above: all services active and the PostgreSQL data queryable — see [services/teslamate/README.md](teslamate/README.md#deployment-steps).
