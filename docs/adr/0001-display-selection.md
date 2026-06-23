# ADR-0001: Display Selection — Waveshare 2.4" DSI

**Date:** 2026-06-23  
**Status:** Accepted

## Context

The Pi Zero 2W has one combined CSI/DSI ribbon connector. Whatever display is chosen consumes that port entirely — there is no second bus available without a HAT that itself occupies the GPIO header. The enclosure target is approximately 149×68×22mm, which sets a hard upper bound on screen size and constrains how deeply any connector or ribbon can protrude behind the board.

The display must be readable during emulation (320×240 minimum for NES/SNES content), responsive enough to not stutter at typical RetroArch frame rates (~60 fps), and physically mountable flush against the front face of the enclosure without a secondary PCB adapter board.

## Decision

Use the **Waveshare 2.4" DSI display** (320×240, direct ribbon connection to the Pi Zero 2W DSI port).

## Considered Alternatives

**SPI displays (ILI9341 or similar, SPI bus)**  
SPI panels are cheap and widely supported under Batocera/fbcp-ili9341. The problem is bandwidth: the ILI9341 at its typical max SPI clock (~50 MHz effective) delivers around 9–12 fps at 320×240 in 16-bit color. That is insufficient for smooth emulation. The driver overhead on a Pi Zero 2W also competes with the emulator CPU budget in a way that a hardware DSI path does not. Rejected on throughput grounds.

**Small HDMI panels (3.5"–4" via mini-HDMI)**  
The Pi Zero 2W exposes only mini-HDMI, which is electrically viable. However, HDMI panels in this size range are physically thicker due to driver boards and frame assemblies, pushing the enclosure depth well past 22mm. They also require a mini-HDMI cable or right-angle adapter that creates a fragile mechanical joint at the thinnest point of the board. Rejected on form-factor grounds.

**Official Raspberry Pi DSI touchscreen (7")**  
Far too large for the enclosure. Not considered seriously beyond noting the DSI port compatibility.

**No-name DSI modules from AliExpress**  
Datasheets and Batocera compatibility are unreliable. The Waveshare part has documented DTS overlays and a known-good track record with Pi Zero. Rejected in favour of a part with credible vendor documentation.

## Consequences

- The CSI port is permanently occupied; a camera add-on is not possible without replacing the display.
- Display bring-up depends on the Waveshare DSI overlay being present and correctly configured in Batocera — this is a known quantity but still a setup step.
- 320×240 is comfortable for 8-bit and 16-bit era content; anything requiring higher resolution (e.g., N64, PSP) will rely on downscaling and will look soft.
- The DSI ribbon length constrains how far the display can be offset from the Pi board — enclosure layout must account for this.
