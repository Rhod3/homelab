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

Static IP is outside the router's DHCP pool — see [hardware/networking.md](networking.md) for the DHCP pool range, router configuration, and the full static assignment list.

### Host Interfaces (`/etc/network/interfaces`)

| Interface | Hardware / Role | Notes |
| --- | --- | --- |
| `nic0` | Onboard Intel I219-family NIC (`e1000e` driver, PCI `00:1f.6`) | Sole uplink, bridge port of `vmbr0`. TSO/GSO/GRO offloads disabled — see [Known Issues](#known-issues) |
| `vmbr0` | Linux bridge, `bridge-ports nic0` | Carries the host IP (`192.168.0.2/24`) and every VM/LXC. `bridge-stp off`, `bridge-fd 0` (Proxmox defaults) |
| `nic1` | **No such device** | Stale `inet manual` stanza: `ethtool -i nic1` returns `No such device` (2026-10-10). Probably left over from interface-name pinning or hardware present at install. Harmless, so it was left in place |

`ifreload -a -s` (syntax check) prints `warning: vmbr0: bridge-fd: value of out range "0"`. This is expected and harmless: the 2–255 range only applies with STP on, and `0` is the Proxmox default for a bridge with `bridge-stp off`.

---

## Current Role

- Runs Proxmox VE as the base operating system.
- Acts as the main hypervisor for virtual machines and LXC containers.
- Targeted host for Home Assistant OS migration from Synology VMM.
- Target host for Jellyfin media server with GPU passthrough.

---

## Known Issues

### `e1000e` "Detected Hardware Unit Hang" (onboard NIC lock-up)

**Status**: Fix applied and verified 2026-10-10, including after a reboot. Monitoring for recurrence.

**Symptom**: The host and every guest disappear from the network at once. They don't answer ping, the web UI (8006) or SSH, and ARP to `.2`/`.30`/`.31`/`.32` stays `incomplete`. The host itself keeps running: the power LED stays on, and the switch port link LED stays green with occasional orange activity. The NIC does not recover by itself.

**Incident, 2026-10-10**:

| Time (CEST) | Event |
| --- | --- |
| 15:50:49 | First `e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang`, after ~7.7 days of uptime on kernel `7.0.14-20-pve`. Repeated every 2 s from then on |
| 16:59:20 | Short press on the power button → clean ACPI shutdown (`systemd-poweroff`). Guests stopped cleanly, so no crash-consistency concerns |
| 17:01:50 | Powered back on. Host, CT 100, VM 101, CT 102 and the `/mnt/pve-nfs-tv` NFS mount all came back on their own |

Total outage: ~70 minutes. The previous boot (kernel `7.0.14-16-pve`, 2026-09-13 → 2026-10-02, 19 days) logged **zero** hangs. That points at the newer kernel, but it's a single event, not proof.

**Cause**: A long-standing bug in the Intel I219 / `e1000e` combination. The NIC's hardware offloads (TCP segmentation and receive coalescing, i.e. TSO/GSO/GRO) can lock up the transmit queue under some traffic patterns.

**Fix applied**: Turn those offloads off on `nic0`. A `post-up` line on the `vmbr0` stanza in `/etc/network/interfaces` reapplies it each time the bridge comes up, including at boot:

```
auto vmbr0
iface vmbr0 inet static
        ...
        bridge-fd 0
        post-up /usr/sbin/ethtool -K nic0 tso off gso off gro off
```

- Backup of the pre-fix file: `/etc/network/interfaces.bak-2026-10-10` on `pve`. Copy it back and run `ifreload -a` to undo.
- Trade-off: segmentation work moves from the NIC to the CPU. This is negligible at Gigabit on the i7-12700T.
- If the network is later changed through the Proxmox GUI, check that the `post-up` line is still there.

**Verification**:

```bash
ethtool -k nic0 | grep -E 'tcp-segmentation-offload|generic-segmentation|generic-receive'   # all three should be "off"
journalctl -k | grep -c 'Unit Hang'                                                          # should stay 0
```

- [x] Applied live and via `ifreload -a` on 2026-10-10. All three offloads report `off`
- [x] Rebooted 2026-10-10 (up since 17:27:36). All three offloads still report `off`, so the `post-up` line works on its own at boot

**If it happens again** (escalation options, least invasive first):

| Option | Pros | Cons |
| --- | --- | --- |
| Pin the previous kernel: `proxmox-boot-tool kernel pin 7.0.14-16-pve`, reboot | Directly tests the kernel-regression theory. That kernel ran 19 days without a hang. Undone with `kernel unpin` | Misses security and bug fixes while pinned. Diagnostic step only |
| Disable Energy-Efficient Ethernet (`ethtool --set-eee nic0 eee off`) or PCIe ASPM (`pcie_aspm=off` kernel parameter) | Covers the less common hang causes that offloads don't | `pcie_aspm=off` affects every PCIe device and increases idle power draw |
| Watchdog that pings the gateway and resets `nic0` (or reboots) on failure | Automatic recovery, whatever the root cause | Treats the symptom, not the cause, and adds a moving part. Best as a safety net alongside a fix |

**Recovery without a monitor**: a short (~1 s) press on the power button triggers a clean shutdown even while the NIC is hung, because the OS is still alive. Use a long press (hard off) only if the short press does nothing after ~2 minutes.

---

## Future Upgrade Roadmap

- [x] **Storage Upgrade**: Added a 512 GB SSD in the second M.2 port for additional virtual disk space.
- [ ] **Memory Upgrade**: Upgrade RAM to 64 GB DDR5 as service density increases.
- [ ] **Hardware Transcoding**: Pass through Intel UHD Graphics 770 / Quick Sync to Jellyfin for efficient hardware video encoding/decoding.
- [ ] **Additional Nodes**: Consider adding secondary compute nodes for high-availability Proxmox clustering if needed.
