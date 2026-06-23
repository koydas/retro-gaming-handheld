# ADR-0002: Battery and Charging — TP4056 + 3.7V LiPo ~4000mAh

**Date:** 2026-06-23  
**Status:** Accepted — provisional *(charge-and-play topology unresolved; see Consequences)*

## Context

The handheld needs an onboard power supply that fits the 149×68×22mm enclosure, provides enough runtime to be useful (target: 2+ hours at moderate emulator load), charges safely over USB, and does not require custom PCB circuitry to implement correctly on a first build. LiPo cells are dangerous if overcharged, over-discharged, or shorted — any chosen solution must include both charge management and over-discharge protection.

The Pi Zero 2W draws roughly 0.5W idle and peaks near 1.5W under emulator load. A ~4000mAh cell at 3.7V nominal is ~14.8Wh raw; at ~80% boost efficiency that yields ~11.8Wh usable at 5V. At 1.5W peak draw this gives a theoretical ceiling of roughly 6–8 hours; realistic play is 3–4 hours accounting for display load and occasional spikes.

## Decision

Use a **TP4056 module with on-board DW01A protection IC** paired with a **single-cell 3.7V LiPo around 4000mAh**, boost-converted to 5V for the Pi.

## Considered Alternatives

**Adafruit PowerBoost 1000C (or clone)**  
The PowerBoost integrates boost conversion and charging in one board, which is appealing. It was rejected because it tops out at 1A charge current and the boost output is limited to ~1A continuous — marginal when the Pi peaks and the display is active simultaneously. It also costs more and is physically larger than a TP4056 module, with less flexibility in how the charging port is routed. Rejected on output headroom and layout flexibility grounds.

**USB-C PD negotiation for charging**  
PD requires a protocol negotiation IC (e.g., FUSB302 or a PD sink controller), custom PCB work, and firmware or hard-coded configuration resistors to request the correct voltage. This is a sensible investment on a rev 2 board but is out of scope for a first build where the goal is to learn the basics without compounding risk. A TP4056 charges from any 5V USB source including old phone chargers, which is more forgiving during iteration. Rejected as premature complexity.

**18650 lithium-ion cells**  
18650 cells are cylindrical and require a dedicated cell holder with spring contacts, adding mechanical complexity and depth. The cells themselves are rigid, meaning the battery compartment must be sized exactly to the cell diameter (18mm) — this wastes internal volume compared to a flat LiPo cell that can be shaped to fill the available cavity. Capacity per millimeter of depth is also lower than a flat pouch cell at this size. Rejected on form-factor efficiency grounds.

**Off-the-shelf USB power bank PCB**  
Donor power bank boards are cheap and handle charging and boost in one unit, but they introduce an unnecessary idle drain from their own microcontroller, have unpredictable pass-through behaviour under load (some cut out briefly when plugging in a charger), and are physically designed around 18650 cells. Rejected due to unreliable pass-through and wasted volume.

## Consequences

- A protection circuit is required — a TP4056 module *without* the DW01A protection IC must not be used with a bare LiPo cell. The module choice matters; this must be verified at procurement.
- The TP4056 charges at a fixed current set by a single resistor (default 1A on most modules). This is adequate for a 4000mAh cell (0.25C) but not adjustable without hardware modification.
- A separate boost converter is needed between the LiPo output and the Pi's 5V input. This is a discrete component and a wiring step, but adds one more failure point to audit during assembly.
- Over-discharge protection on the DW01A cuts out at ~2.5V per cell — the Pi will hard-reset without a clean shutdown. A low-battery GPIO signal or voltage monitor IC is not included in this design and should be revisited if data corruption on the SD card becomes a problem.
- LiPo cells of this capacity in flat form factor are commonly sourced from tablet/phone replacement markets. Physical dimensions vary between vendors even at the same mAh rating — the enclosure battery compartment must have a few millimeters of tolerance built in.
- **Charge-and-play topology is unresolved — decision deferred to Phase 2.** The TP4056 is not a power-path charger: when the Pi runs while USB is connected, the charger sees the boost converter's draw as part of the battery load, which prevents the CC/CV termination condition from being met. The result is indefinite charging that never terminates cleanly. Additionally, the 1A USB input must supply both the Pi (~0.5–1.5W) and the charge current simultaneously; if the Pi spikes, the input rail may sag and cause a brownout. **Component damage risk:** the DW01A over-voltage protection cuts at ~4.3V, so the LiPo cell itself is protected from destructive overcharge regardless of termination behavior. The real risk is SD card corruption on unclean shutdown, not hardware damage. Two resolution paths exist for Phase 2, chosen based on available PCB/enclosure space: (a) **Power-path IC** (e.g., BQ24074) — a true load-sharing/power-path manager that separates the system load from the charge path, allows correct CC/CV termination while the Pi runs, preferred if space allows; CN3165 is not a valid alternative here — it is a CC/CV charger with the same termination problem as the TP4056; (b) **Power-off-to-charge constraint** — document that the device must be powered off before connecting USB, eliminate the risk by eliminating simultaneous operation, acceptable for a v1 learning device if space does not permit option (a).
