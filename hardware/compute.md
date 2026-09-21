# Compute Node: HP Elite Mini 600 G9

The HP Elite Mini 600 G9 serves as the primary compute node for the homelab, running Proxmox VE.

---

## Specifications

| Component | Detail |
| --- | --- |
| **Model** | HP Elite Mini 600 G9 |
| **CPU** | Intel Core i7-12700T (12 Cores / 20 Threads) |
| **RAM** | 32 GB DDR5 |
| **Storage** | 256 GB NVMe SSD (primary, OS/boot) + 512 GB SSD (secondary, second M.2 port) |
| **iGPU** | Intel UHD Graphics 770 (Quick Sync Video support) |
| **Hypervisor** | Proxmox VE 9.2 |

---

## Network Configuration

| Setting | Value |
| --- | --- |
| IP Address | 192.168.0.2 (static) |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.0.1 |
| DNS | 192.168.0.1 |

Static IP is outside the router's DHCP pool (192.168.0.100–192.168.0.249) — see [hardware/networking.md](networking.md) for the router's DHCP configuration and the full static assignment list.

---

## Current Role

- Runs Proxmox VE as the base operating system.
- Acts as the main hypervisor for virtual machines and LXC containers.
- Targeted host for Home Assistant OS migration from Synology VMM.
- Target host for Jellyfin media server with GPU passthrough.

---

## Future Upgrade Roadmap

- [x] **Storage Upgrade**: Added a 512 GB SSD in the second M.2 port for additional virtual disk space.
- [ ] **Memory Upgrade**: Upgrade RAM to 64 GB DDR5 as service density increases.
- [ ] **Hardware Transcoding**: Pass through Intel UHD Graphics 770 / Quick Sync to Jellyfin for efficient hardware video encoding/decoding.
- [ ] **Additional Nodes**: Consider adding secondary compute nodes for high-availability Proxmox clustering if needed.
