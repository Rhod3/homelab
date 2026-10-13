# Networking Infrastructure

This document details the current topology and future network design. For a broader discussion of alternative segmentation models, gateway options, and rollout sequencing, see [docs/network-strategy.md](../docs/network-strategy.md).

---

## Current Topology

```mermaid
flowchart TD
    Internet[Internet] --> Upstream[Upstream ISP Router<br/>FRITZ!Box - Init7]
    Upstream -->|WAN: 192.168.178.21| Router[TP-Link Archer C2300<br/>LAN: 192.168.0.1/24]
    Router --> Switch[8-Port Gigabit Switch]
    Switch --> HP[HP Elite Mini 600 G9]
    Switch --> Synology[Synology DS420+]
    Switch --> PC[PC & Client Devices]
    Switch --> Spare[Spare Switch Ports]
    Router -.Wi-Fi: Isengard.-> WiFiClients[Wireless Clients]
    Router -.Guest Wi-Fi radio off.-> GuestClients[Guest Clients]
```

### Current Components
- **ISP Router**: FRITZ!Box (exact model not tracked), since the recent change of ISP from Sunrise to Init7 (before 2026-10-13). It hands internet access to the TP-Link router's WAN port (outer side of the double NAT). Not otherwise managed or configured as part of this homelab. The previous Sunrise router used `192.168.1.0/24` (TP-Link WAN `192.168.1.22`, gateway `192.168.1.1`).
- **Router**: TP-Link Archer C2300-class (AC2300 tier)
- **Switch**: TP-Link LS1008G — 8-Port 10/100/1000Mbps unmanaged desktop switch (127 × 66.5 × 23 mm, fanless)

### Router Configuration (as observed)

| Interface | Setting | Value |
| --- | --- | --- |
| WAN | IP Address | 192.168.178.21 (reported by the TP-Link through NAT-PMP to Tailscale on 2026-10-13; not yet read from the TP-Link status page) |
| WAN | Subnet Mask | To confirm on the TP-Link status page |
| WAN | Default Gateway | To confirm on the TP-Link status page (the FRITZ!Box — see Current Components above) |
| WAN | Connection Type | Dynamic IP |
| LAN | IP Address | 192.168.0.1 |
| LAN | Subnet Mask | 255.255.255.0 |
| LAN | DHCP Server | Enabled |
| LAN | DHCP Pool | 192.168.0.100 – 192.168.0.249 |
| LAN | Lease Time | 120 minutes |
| NAT | UPnP / NAT-PMP | Enabled (observed 2026-10-13 via `tailscale netcheck`). Lets any LAN device open port forwards on the TP-Link; whether to turn it off is an open decision, see [services/tailscale/README.md](../services/tailscale/README.md#future-work) |
| LAN | Address Reservations | None configured on the router — static assignments below are set device-side instead |
| Wireless (2.4G/5G) | SSID | `Isengard` |
| Wireless (2.4G/5G) | Mode | 802.11b/g/n mixed |
| Wireless (2.4G) | Channel | Auto (currently channel 2) |
| Guest Network | SSID | `TP-Link_Guest_CD61` |
| Guest Network | Wireless Radio | Off (not currently in use) |
| Guest Network | Client Isolation | "Allow Guests to Access Each Other" — Off |

### Static IP Assignments

Devices below `.100` are outside the router's DHCP pool (`192.168.0.100`–`192.168.0.249`) and are configured with a static IP directly on the device rather than via a router-side DHCP reservation.

Within that `.1`–`.99` static range, a sub-range convention keeps assignments organized ahead of eventual VLAN segmentation:

| Range | Purpose | Maps to (future) |
| --- | --- | --- |
| `.2`–`.29` | Core infrastructure (hypervisor, NAS, network gear) | VLAN 10 — Management |
| `.30`–`.99` | Individual services/apps hosted on Proxmox (LXCs/VMs) | VLAN 20 — Servers |

| Device | IP Address | Configured Via |
| --- | --- | --- |
| HP Elite Mini 600 G9 (Proxmox) | 192.168.0.2 | Proxmox host network config (see [hardware/compute.md](compute.md#network-configuration)) |
| Synology DS420+ | 192.168.0.10 | DSM → Control Panel → Network (see [hardware/storage.md](storage.md#network-configuration)) |
| Jellyfin LXC (CT 100) | 192.168.0.30 | Proxmox VE Advanced Settings during LXC creation (see [services/jellyfin/README.md](../services/jellyfin/README.md)) |
| Arr Stack Docker VM (VM 101, `arr-stack`) | 192.168.0.31 | Proxmox Cloud-Init tab, applied by cloud-init on first boot (see [services/arr-stack/README.md](../services/arr-stack/README.md#deployment-specs)) |
| TeslaMate LXC (CT 102) | 192.168.0.32 | Proxmox VE Advanced Settings during LXC creation (see [services/teslamate/README.md](../services/teslamate/README.md)) |
| Tailscale subnet router LXC (CT 104, `tailscale`) | 192.168.0.34 | Proxmox VE Create CT wizard, Network tab (see [services/tailscale/README.md](../services/tailscale/README.md#deployment-specs)) |

---

## Future Network Roadmap

Segmentation model, gateway/firewall hardware, wireless strategy, DNS/DHCP placement, and remote access are all still open decisions with multiple viable options and explicit trade-offs — see [docs/network-strategy.md](../docs/network-strategy.md) for the full set of propositions and the recommended sequencing. Nothing there is committed until it is reflected in the "Current Topology" section above.
