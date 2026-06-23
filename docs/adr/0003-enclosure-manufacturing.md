# ADR-0003: Enclosure Manufacturing — Bambu Lab FDM

**Date:** 2026-06-23  
**Status:** Accepted

## Context

The enclosure must fit all internal components within approximately 149×68×22mm: the Pi Zero 2W, a flat ~4000mAh LiPo cell, the TP4056 module, a boost converter, the PCB façade, and a 2.4" SPI display (ILI9341 or compatible). Tolerances matter — the display cutout, button holes, and PCB standoffs need to be accurate to within ~0.3mm or the assembly either binds or rattles. The project is explicitly a learning exercise, meaning iteration is expected and the manufacturing method must support rapid design cycles without significant lead time or cost per iteration.

## Decision

Manufacture the enclosure using **FDM 3D printing on a Bambu Lab printer** (PLA or PETG, 0.2mm layer height, ≥3 perimeters on structural walls).

## Considered Alternatives

**Laser-cut acrylic (stacked sandwich construction)**  
Laser cutting produces clean edges and tight XY tolerances (~0.1mm on a decent machine), and stacked acrylic is a common approach for hobbyist enclosures. The problem is that acrylic sandwich construction is inherently 2.5D — curved ergonomic surfaces and integrated snap features are either impossible or require many thin layers. The enclosure design benefits from organic grip curves and integrated PCB bosses that cannot be expressed in flat sheets without approximation. Per-iteration cost is also non-trivial unless a laser cutter is on hand; sending files to a service adds days of lead time. Rejected on geometry expressiveness and iteration speed grounds.

**CNC-machined aluminium or plastic**  
CNC provides excellent tolerances and a premium feel. It is also expensive (typically $50–200+ per part from a service bureau), requires DFM knowledge to avoid undercuts and thin walls, and has multi-day to multi-week lead times. For a first physical prototype where dimensions are expected to change, this is the wrong tool. Rejected on cost and iteration speed grounds.

**Off-the-shelf enclosure (project box, handheld shell)**  
Generic project boxes do not match the required dimensions and lack the precise cutouts and button placement the design needs. Donor shells from commercial handhelds (e.g., Game Boy shells) impose their own geometry constraints and may not accommodate the Pi Zero mounting pattern or the specific display size. Adapting a donor shell to fit custom internals often ends up requiring as much work as printing from scratch, but with less freedom. Rejected because the internals drive the geometry, not the other way around.

**Resin SLA printing (outsourced or desktop)**  
SLA produces higher detail and smoother surfaces than FDM. However, resin parts at this wall thickness (2–3mm) are more brittle than PETG and require post-cure infrastructure. Desktop resin printers also have smaller build volumes — fitting a 149×68mm footprint in a single print is marginal on most consumer SLA machines. The surface quality advantage is not needed for a functional enclosure. Rejected on material toughness and process complexity grounds.

## Consequences

- FDM layer lines are visible. If surface finish matters for a final version, light sanding and filler primer are required — this is accepted as-is for functional iterations.
- Bambu Lab slicer settings (layer height, perimeter count, support placement) directly affect dimensional accuracy at button holes and display cutouts. A calibration print verifying hole diameter and wall thickness should precede the full enclosure print.
- Print time for the full shell at 0.2mm is approximately 4–8 hours per half depending on infill. Failed prints are not zero-cost — filament and time are consumed. Design for supportless overhangs where possible to reduce failure modes.
- PETG is preferred over PLA for the final print due to better impact resistance and lower creep under the battery compression load. PLA is acceptable for early fit-check iterations.
- The Bambu Lab ecosystem ties the workflow to proprietary slicer software (Bambu Studio). CAD files should be exported as STEP and/or STL alongside any native format to avoid lock-in if the printer changes.
