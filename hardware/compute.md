# Compute Node: HP Elite Mini 600 G9

The HP Elite Mini 600 G9 serves as the primary compute node for the homelab, running Proxmox VE.

---

## Specifications

| Component | Detail |
| --- | --- |
| **Model** | HP Elite Mini 600 G9 |
| **CPU** | Intel Core i7-12700T (12 Cores / 20 Threads) |
| **RAM** | 32 GB DDR5 |
| **Storage** | 256 GB NVMe SSD |
| **iGPU** | Intel UHD Graphics 770 (Quick Sync Video support) |
| **Hypervisor** | Proxmox VE |

---

## Current Role

- Runs Proxmox VE as the base operating system.
- Acts as the main hypervisor for virtual machines and LXC containers.
- Targeted host for Home Assistant OS migration from Synology VMM.
- Target host for Jellyfin media server with GPU passthrough.

---

## Future Upgrade Roadmap

- [ ] **Storage Upgrade**: Replace or add a 1 TB+ NVMe SSD to accommodate more virtual disk space.
- [ ] **Memory Upgrade**: Upgrade RAM to 64 GB DDR5 as service density increases.
- [ ] **Hardware Transcoding**: Pass through Intel UHD Graphics 770 / Quick Sync to Jellyfin for efficient hardware video encoding/decoding.
- [ ] **Additional Nodes**: Consider adding secondary compute nodes for high-availability Proxmox clustering if needed.
