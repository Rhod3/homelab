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
