# ADR-0001: Display Selection — SPI Display via fbcp-ili9341

**Date:** 2026-06-23  
**Status:** Accepted *(revised — original draft assumed DSI; see context)*

## Context

The Pi Zero 2W has one connector on its short edge: a 22-pin CSI ribbon port for the camera. It does **not** expose a DSI display port. This was not caught in the initial draft of this ADR, which incorrectly assumed a combined CSI/DSI port existed on the Zero 2W (as it does on some other Pi models). The error was flagged during PR review and confirmed against Raspberry Pi's hardware documentation. Everything that followed from that assumption — the rejection of SPI on bandwidth grounds, the selection of a DSI panel — was therefore invalid and is retracted here.

The corrected constraint set:

- Pi Zero 2W exposes: CSI (camera), mini-HDMI, micro-USB, 40-pin GPIO header.
- No DSI port is available. A DSI display cannot be connected without an intermediary board that itself occupies the GPIO header — which is needed for the button matrix.
- The enclosure target is 149×68×22mm. The Pi Zero 2W at 30×65mm leaves ~19mm on each side for enclosure walls and controls. This width budget is what makes the layout viable; it cannot be traded away for a larger SBC.
- Target display: ~2.4", 320×240, thin enough to sit flush in 22mm total depth.
- Target frame rate: **30fps** is sufficient for 8-bit and 16-bit era emulation. This is the revised constraint that changes the SPI calculus.

## Decision

Use an **SPI display** (ILI9341 or compatible controller, 320×240, 2.4") driven by the **fbcp-ili9341** framebuffer copy driver on Batocera.

fbcp-ili9341 uses DMA and pushes the SPI bus at up to ~125MHz effective on a Zero 2W, achieving 30–60fps at 320×240 in 16-bit color on the quad-core CPU. The 30fps target is comfortably met. SPI panels in this size are thin (~2mm PCB + glass), inexpensive, and well-documented under Linux.

The specific module (Waveshare, Adafruit, generic breakout) is not locked in at this ADR stage — any ILI9341-compatible 2.4" panel with a matching pinout will work. Module selection happens at procurement.

## Considered Alternatives

**DSI display (original decision — retracted)**  
The initial draft selected a Waveshare 2.4" DSI panel on the grounds that DSI would avoid SPI bandwidth limits. This was based on an incorrect assumption that the Pi Zero 2W exposes a DSI port. It does not. Retracted on hardware constraint grounds.

**Mini-HDMI to small HDMI panel**  
The Zero 2W exposes mini-HDMI, making this electrically viable. However: HDMI panels in the 2.4"–3.5" range include a driver board that adds 6–10mm of depth behind the display face, which is incompatible with a 22mm total enclosure depth. A right-angle mini-HDMI adapter plus cable also creates a fragile mechanical joint at a point where the board is close to the enclosure wall. Rejected on depth and mechanical reliability grounds.

**Switch SBC to Pi 3A+ (which has DSI)**  
The Pi 3A+ has a native DSI port and would enable the original display plan. However, the Pi 3A+ is 56mm wide vs the Zero 2W's 30mm — in a 68mm-wide enclosure, that leaves 6mm total margin for walls and side controls. The button layout (2×3 action + L/R) requires ~19mm per side and cannot be accommodated. Rejected on enclosure geometry grounds.

**SPI at single-threaded speeds (~9–12fps)**  
The initial ADR rejected SPI citing this figure, which is correct for naive single-threaded writes. fbcp-ili9341 does not use naive writes — it uses DMA and pushes the bus harder than a software loop can. The 9–12fps figure was not representative of the actual driver performance on this hardware. Rejection of SPI was therefore premature and is reversed.

## Consequences

- The specific ILI9341 module must be verified for SPI clock compatibility with the Zero 2W before ordering. Most 2.4" modules run fine at 40–62MHz SPI; a few cheaper ones throttle to 16MHz and will not hit 30fps.
- fbcp-ili9341 configuration requires specifying GPIO pin assignments, SPI bus speed, and display orientation in Batocera. This is documented in the driver's README and is a known-quantity setup step.
- 30fps at 320×240 covers NES, SNES, GBA, and Game Boy. N64 and PSP content will require downscaling and may stutter — this is accepted for the target platform scope.
- The CSI camera port remains free. A camera add-on is theoretically possible but not in scope.
- SPI occupies several GPIO pins. The button matrix PCB routing must avoid those pins; this is a layout constraint for Phase 2.
