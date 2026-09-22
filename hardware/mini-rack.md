# Mini Rack: KWS Rack v2 (3D-Printed 10-inch)

**Status: Planned.** Nothing has been printed or assembled yet — this document tracks the design/sizing plan and will be updated as parts are printed and the rack is built.

The mini rack is an enclosure for equipment that currently sits loose in its existing location — not a replacement location, not a topology change. It exists purely to give the HP Elite Mini, Synology, switch, and associated cabling a tidier, more organized physical home.

---

## Design Source

| Item | Detail |
| --- | --- |
| **Design** | [KWS Rack v.2 — Heavy-Duty 10-Inch Homelab Rack](https://makerworld.com/en/models/2139130-kws-rack-v-2-heavy-duty-10-inch-homelab-rack) (MakerWorld) |
| **Designer** | Ilan Kushnir (KWS Labs) — [kwslabs.com/rack](https://www.kwslabs.com/rack) |
| **Standard** | 10-inch rack width (not 19-inch) |
| **Modularity** | Frames stack in 3U or 6U sections; minimum 3U, maximum tested 30U; recommended starting point is a 6U base |
| **Material (designer's recommendation)** | PETG — PLA is explicitly called out as unsuitable (not heat-resistant enough for homelab use) |

---

## Print Plan

| Setting | Value |
| --- | --- |
| **Printer** | Bambu Lab P1S |
| **Material** | PETG-HF |
| **Location once built** | Same physical location as current equipment — no relocation planned |

PETG-HF matches the designer's own material recommendation, so no deviation needed there.

---

## Planned Contents & Sizing

U height is meant to be derived from what actually goes in the rack rather than picked upfront. Below is the current draft based on what you've specified, cross-checked against the official KWS module library where a matching module exists.

### Front Elevation (approximate, to scale by U)

Bottom-to-top order: NAS, power shelf, Mini PC, switch, patch panel (heavier/base items low, networking gear up top).

```
┌────────────────────────────────────┐
│    (reserved: router, if added)    │  TBD
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┤
│         Patch Panel (GeeekPi)      │  0.5U
├────────────────────────────────────┤
│         TP-Link LS1008G            │  1U
├────────────────────────────────────┤
│         HP Elite Mini 600 G9       │  1U
├────────────────────────────────────┤
│         Power Bricks Shelf         │  2U
│                                    │
├────────────────────────────────────┤
│                                    │
│         Synology DS420+            │  4U
│                                    │
│                                    │
└────────────────────────────────────┘

  8.5U confirmed + TBD router
  → build target: 9U (no router) or 12U (with router)
```

The table below is the authoritative source — update it first when contents change, then keep this diagram in sync.

| Component | Planned U | Mounting Plan | Notes |
| --- | --- | --- | --- |
| Synology DS420+ | 4U | Official [KWS "Synology 4-Bay NAS module"](https://makerworld.com/en/collections/17479078-kws-rack-system) | Matches your figure exactly — purpose-built module |
| Power bricks | 2U | Official KWS "2U Power Supplies Shelf" (vented, cable-management-oriented) | Matches your figure exactly |
| HP Elite Mini 600 G9 | 1U (**confirmed**) | 177 × 175 × 34 mm (W × D × H). No official KWS module; needs a custom tray | 175mm depth should still be checked against the chosen frame depth in the KWS generator before finalizing the tray design |
| Gigabit switch | 1U (**confirmed**) | TP-Link LS1008G — 127 × 66.5 × 23 mm, fanless desktop switch (see [hardware/networking.md](networking.md#current-components)). No official KWS module for this model; needs a custom or adapted tray | — |
| Patch panel | 0.5U (**confirmed**) | GeeekPi 12-port patch panel (off-the-shelf product) | Resolves the earlier discrepancy — this is a separate product from the official KWS "keystone" module (2–3U), and 0.5U is correct for the GeeekPi unit |
| Router (potential) | TBD | Not yet decided whether this is the existing TP-Link Archer C2300 or a future replacement | Tied to the still-open decisions in [docs/network-strategy.md](../docs/network-strategy.md) |

**Running total:** 4U (Synology) + 2U (power) + 1U (Mini PC) + 1U (switch) + 0.5U (patch panel) = **8.5U confirmed**, excluding the router. Per the KWS stacking convention (6U base + 3U/6U extensions), that rounds up to a **9U build (6U base + 3U extension)** if no router goes in the rack, or **12U (6U base + 6U extension)** if a router module needs to be reserved.

---

## Assembly Hardware (per KWS spec, per 1U section)

| Part | Quantity |
| --- | --- |
| Brass M5 heat-set inserts (8mm) | 12 |
| M6×8mm socket head screws | 16 |
| M6×10mm socket head screws | 24 |
| M6 nylon self-locking nuts | as needed per frame |
| 10mm×5mm round magnets (optional, for magnetic back plate) | 14 per 6U leg / 2 per 3U leg |

Scale quantities by the final U count once determined.

---

## Accessories Under Consideration

Not planned for the initial build, but noted as possible future additions:

- Brush panel (cable pass-through)
- PDU / power strip mount
- UPS back-panel holder — relevant once [hardware/power.md](power.md) UPS plans firm up

---

## Open Questions

1. **HP Elite Mini mounting** — 1U is confirmed, but no purpose-built KWS module exists for this mini PC. Options:
   - Design/print a custom vented shelf sized to the Mini's 177 × 175 × 34 mm footprint (more work, cleanest fit).
   - Use a generic flat/vented shelf if one exists in the broader KWS ecosystem, accepting a looser fit.
2. **Switch mounting** — 1U is confirmed, but the LS1008G (127 × 66.5 × 23 mm) has no official KWS module. Same two options as above: custom tray vs. generic shelf.
3. **Router** — pending the broader network segmentation decisions in [docs/network-strategy.md](../docs/network-strategy.md); affects whether "potential router" is a rack tenant at all, and if so, which model and how much U to reserve.
4. **Final total U** — 9U if no router is added, 12U if a router slot is reserved.

---

## References

- [KWS Rack v.2 on MakerWorld](https://makerworld.com/en/models/2139130-kws-rack-v-2-heavy-duty-10-inch-homelab-rack)
- [KWS Labs project site](https://www.kwslabs.com/rack)
- [KWS Rack System collection (modules/accessories)](https://makerworld.com/en/collections/17479078-kws-rack-system)
