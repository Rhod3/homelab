# Power & Protection

This document outlines the power infrastructure and uninterruptible power supply (UPS) strategy.

---

## Current State

- Connected directly to standard AC mains power.
- No dedicated UPS protection currently installed.

---

## Future UPS Deployment Plan

Add a dedicated UPS unit to protect core infrastructure.

### Protected Equipment
- Synology DS420+ NAS
- HP Elite Mini 600 G9 Compute Host
- Network Switch & TP-Link Router

### Objectives
1. **Outage Ride-Through**: Maintain operations during brief voltage sags and short outages.
2. **Graceful Shutdown**: Trigger automated clean shutdowns of Proxmox VMs and Synology DSM before battery exhaustion.
3. **Core Availability**: Keep Home Assistant and local network equipment powered during short outages.
