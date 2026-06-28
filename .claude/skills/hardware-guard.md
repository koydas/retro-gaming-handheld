# hardware-guard

## When to Apply
When any message proposes or evaluates hardware components, display interfaces, SBC choices, display driver software, OS versions, or power topology for this project. Also when answering questions about what is or is not possible on this hardware.

## Expected Behavior

Before accepting or proposing any hardware suggestion, validate it against the confirmed facts below. If a suggestion conflicts with a confirmed fact, reject it by name with the specific reason.

### Wrong assumptions already caught in this project — reject on sight

| Suggestion | Why it is wrong |
|---|---|
| DSI display on Pi Zero 2W | The Pi Zero 2W has **no DSI port**. Any DSI solution requires an intermediary board that occupies the 40-pin GPIO header, which is needed for the button matrix. |
| `fbcp-ili9341` as the display driver | Requires DispmanX, which is not available on Raspberry Pi OS Bookworm (KMS/DRM stack). Upstream development stalled since 2019. See ADR-0005. |
| Pi 3A+ (or any Pi wider than ~31mm) as the SBC | Pi 3A+ is 56mm wide. In a 68mm enclosure, 56mm leaves 6mm total margin for walls and controls. The button layout requires ~19mm per side. Enclosure geometry rules it out. |
| HDMI panel (any small HDMI display) | HDMI panels in the 2.4"–3.5" range include a driver board adding 6–10mm of depth behind the glass. Total enclosure depth is 22mm — incompatible. |
| CN3165 as a charge-and-play solution | CN3165 is a CC/CV charger with the same termination problem as the TP4056. It is not a power-path IC. See ADR-0002, open question Q2. |

### Open questions — do not treat as resolved

These three questions are explicitly unresolved and block specific decisions. State them as open whenever relevant; do not assume an answer.

- **Q1** — Does FBTFT (`fb_ili9341`) hit 30fps at 320×240 on Pi Zero 2W + RPi OS Bookworm? **Unverified.** Blocks full display procurement and the provisional status on ADR-0005. Requires a physical display and measurement.
- **Q2** — Is there PCB space for a power-path IC (e.g., BQ24074) for charge-and-play? **Unresolved.** Blocks ADR-0002 and Phase 2 PCB layout. Answer is determined by the layout, not by spec.
- **Q3** — Does RPi OS Trixie (Debian 13, 32-bit) support FBTFT and the RetroPie installer on Pi Zero 2W? **Unverified.** OS is pinned to Bookworm (Legacy) until confirmed. Blocks ADR-0004 and Phase 5.

### Active hardware constraints (non-negotiable)

- Enclosure: 149 × 68 × 22mm
- SBC: Raspberry Pi Zero 2W, 30 × 65mm
- Display: SPI (ILI9341 or compatible, 320×240, 2.4"), FBTFT driver (`fb_ili9341`)
- Battery: single-cell 3.7V LiPo ~4000mAh, TP4056 module with on-board DW01A protection IC
- OS: Raspberry Pi OS (Legacy, 32-bit) — Bookworm + RetroPie
- SPI0 bus (GPIO 8/9/10/11) is consumed by the display and must not be reused

## Constraints
- Do not propose DSI, fbcp-ili9341, Pi 3A+, or HDMI panels.
- Do not treat Q1, Q2, or Q3 as resolved — flag them explicitly.
- When a hardware claim is uncertain, flag it rather than asserting it confidently.
- Do not propose hardware changes that violate the physical dimensions.
- Verify specs against manufacturer documentation before writing or updating ADRs — the DSI assumption in ADR-0001 v1 is the canonical example of what skipping this step costs.

## References
- CLAUDE.md — "Non-negotiable physical constraints", "Established hardware facts", "Open questions"
- `docs/adr/0001-display-selection.md` — DSI retraction, HDMI rejection, SPI calculus
- `docs/adr/0002-battery-and-charging.md` — CN3165 rejection, Q2 charge-and-play options
- `docs/adr/0004-os-selection.md` — OS pinning to Bookworm, Q3
- `docs/adr/0005-display-driver.md` — fbcp-ili9341 rejection, FBTFT decision, Q1 status
