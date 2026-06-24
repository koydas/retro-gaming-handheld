# Bill of Materials

Components required for the retro gaming handheld. Update this document when a part is confirmed, spec'd, ordered, or received. Passive components (resistors, capacitors, connectors) and wiring belong in the Phase 2 PCB BOM once the schematic exists.

**Current phase:** Phase 1 — Design. No parts ordered yet.

---

## Main Components

| # | Component | Key Spec | Qty | Procurement status | Blocking question / note |
|---|-----------|----------|-----|-------------------|--------------------------|
| 1 | Raspberry Pi Zero 2W | 4× ARM Cortex-A53 @ 1GHz, 512MB RAM, 30×65mm | 1 | Can order | SBC choice locked — no dependency |
| 2 | 2.4" SPI display module (ILI9341) | 320×240, SPI interface, clock ≥40MHz rated | 1 | **Blocked — Q1** | Q1: FBTFT 30fps on Pi Zero 2W unverified. Order after Q1 resolved. Reject modules rated <40MHz SPI. |
| 3 | 3.7V LiPo cell | Flat pouch, ~4000mAh, dimensions TBD | 1 | **Blocked — enclosure dims** | Battery cavity designed in Phase 3. Dimensions vary widely at this capacity — confirm cell dimensions fit before ordering. Order after Phase 3 fit-check print. |
| 4 | TP4056 module with DW01A protection | CC/CV charger + over-discharge protection, micro-USB input | 1 | Can order | **Critical:** verify DW01A protection IC is present on the specific module — many cheap listings omit it. See ADR-0002. |
| 5 | 5V boost converter module | Input: 3.0–4.2V LiPo, output: 5V, ≥1A continuous | 1 | **Pending spec** | Part TBD. Peak draw: Pi Zero 2W ~1.5W + display ~0.3W ≈ 0.36A at 5V; 1A provides sufficient headroom. Common options: MT3608-based module, XL6009. Confirm output ripple is acceptable for Pi before ordering. |
| 6 | Tactile switches | 6×6mm or similar, low actuation force (<1N), through-hole for PCB | ~12 | **Pending Phase 2** | Count and footprint confirmed in Phase 2 PCB schematic. Button layout: D-pad ×4, face ×4, shoulder ×2, Start/Select ×2. |
| 7 | microSD card | ≥16GB, Class 10 / A1 rated | 1 | Can order | No project dependency. A1-rated cards reduce latency for OS + ROM access. |
| 8 | Custom PCB façade | KiCad design → JLCPCB manufacture | 1 | **Phase 2 deliverable** | Not purchased — designed and ordered in Phase 2 after GPIO map and button layout are finalised. |
| 9 | FDM enclosure | PETG (final print), PLA (fit-check iterations) | 1 | **Phase 3 deliverable** | Not purchased — printed in Phase 3. PETG filament likely already on hand; procure if not. |
| 10 | Power-path IC (optional) | e.g., BQ24074 or equivalent | 0–1 | **Blocked — Q2** | Q2: charge-and-play topology unresolved. If Q2 resolves to "power-path IC", this becomes required. If Q2 resolves to "power-off-to-charge", this line drops. Decide in Phase 2. |

---

## Procurement Order

Dependencies dictate this sequence:

1. **Now (no blockers):** Pi Zero 2W, TP4056 + DW01A module, microSD card.
2. **After Q1 resolved:** 2.4" ILI9341 display module.
3. **After Phase 2 PCB schematic:** Tactile switches (count and footprint confirmed), PCB façade order to JLCPCB. Resolve Q2 and decide on power-path IC.
4. **After Phase 3 fit-check print:** LiPo cell (dimensions confirmed against battery cavity). Boost converter (current budget confirmed against power rail design). Final PETG enclosure print.

---

## Out of Scope for This BOM

The following are intentionally excluded — they belong in Phase 2 deliverables or are too granular for a top-level BOM:

- PCB passives (resistors, caps, diodes) — specified in KiCad schematic
- Display ribbon cable / FPC connector — confirmed in Phase 2 PCB layout
- Power wiring (wire gauge, connectors between TP4056, boost converter, Pi) — Phase 4 assembly notes
- USB charging cable — commodity item, not tracked
