# Jellyfin Media Server

Jellyfin is the planned self-hosted media system for streaming movies and TV shows.

---

## Architecture Plan

```mermaid
flowchart LR
    subgraph Compute - HP Elite Mini 600 G9
        PVE[Proxmox VE Host] --> Jellyfin[Jellyfin LXC / VM]
        iGPU[Intel UHD 770 iGPU] -. Quick Sync Passthrough .-> Jellyfin
    end

    subgraph Storage - Synology DS420+
        NAS[Synology DS420+] --> Movies[/volume1/Media/Movies]
        NAS --> TV[/volume1/Media/TV Shows]
    end

    Jellyfin -- Mount NFS / SMB --> Movies
    Jellyfin -- Mount NFS / SMB --> TV
```

---

## Deployment Specs

- **Host Platform**: Proxmox VE on HP Elite Mini 600 G9.
- **Deployment Type**: LXC container or lightweight Linux VM (TBD).
- **Hardware Acceleration**: Intel UHD Graphics 770 Quick Sync Video (QSV) passed through for hardware video encoding/decoding.
- **Media Storage**: Remote mounts over SMB or NFS from the Synology DS420+ NAS (`Media/` shared folder).
