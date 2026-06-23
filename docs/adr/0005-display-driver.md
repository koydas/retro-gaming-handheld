# ADR-0005: Display Driver Stack — FBTFT/KMS over DispmanX/fbcp-ili9341

**Date:** 2026-06-23  
**Status:** Accepted — provisional *(FBTFT performance at 30fps on Pi Zero 2W unverified; see Consequences)*

## Context

ADR-0001 originally named fbcp-ili9341 as the SPI display driver, and ADR-0004 noted that modern Raspberry Pi OS releases (Bookworm and later) no longer expose DispmanX by default — meaning fbcp-ili9341 would not run on a current OS without manually re-enabling a legacy graphics stack. The project preference is a modern, maintainable solution that does not require pinning to an old OS release or working around deprecated infrastructure.

The two driver paths available for an ILI9341 SPI panel on current RPi OS (KMS/DRM-based) are:

- **FBTFT** (`fb_ili9341`) — a framebuffer driver for TFT displays that is part of the mainline Linux kernel (staging/fbtft). Loaded via `dtoverlay` on RPi OS, it exposes `/dev/fb1` and works under KMS through the `fbdev` emulation layer. It is the standard community approach for SPI panels on current Raspberry Pi OS.
- **DRM panel driver** (`panel-ilitek-ili9341`) — a native DRM/KMS panel driver present in mainline Linux. More correct architecturally but requires a DRM-aware frontend; RetroPie's RetroArch can use DRM/KMS output but configuration is more complex and community documentation is sparse for this specific panel.

## Decision

Use **FBTFT (`fb_ili9341`)**, configured via `dtoverlay` in `/boot/firmware/config.txt` on Raspberry Pi OS Bookworm (32-bit).

FBTFT is part of the mainline Linux kernel, actively maintained, and works natively with the KMS-based RPi OS without any legacy stack restoration. It is the standard documented path for SPI TFT panels in the Raspberry Pi OS and RetroPie communities on current images. The native DRM panel driver would be cleaner long-term but has significantly less community documentation for ILI9341 + RetroPie, which adds setup risk for a first build.

The fbcp-ili9341 driver is explicitly **not used** — it requires DispmanX, which is not available on current RPi OS Bookworm, and its upstream development has stalled (last substantive update 2019, Feb 2024 note acknowledges incompatibility with modern distros).

## Considered Alternatives

**fbcp-ili9341 (DispmanX-based)**  
The originally proposed driver. Highly optimised — uses DMA and achieves 60fps at 320×240 — but requires the legacy DispmanX graphics stack, which is not available on RPi OS Bookworm by default. Re-enabling it requires downgrading the firmware or using `dtoverlay=vc4-fkms-v3d` (fake-KMS), which is itself deprecated. Upstream development has stopped. Rejected because freezing the OS to keep a stale driver functional creates maintenance debt incompatible with this project's preference for a modern, maintainable stack.

**DRM panel driver (`panel-ilitek-ili9341`)**  
The architecturally correct long-term path — a native DRM/KMS driver with no legacy dependencies. Present in mainline Linux kernel. Rejected for the first build because community documentation for this driver with RetroPie + Pi Zero 2W is sparse, and debugging a poorly-documented DRM pipeline is out of scope when FBTFT covers the need with much better-documented setup steps. Should be revisited for a rev 2 if FBTFT performance is insufficient.

**Switching to a display with native KMS support (HDMI, DSI)**  
Eliminates the SPI driver problem entirely. Rejected — both interfaces were already excluded by hardware constraints (DSI absent on Pi Zero 2W; HDMI panels too thick for the enclosure). See ADR-0001.

## Consequences

- **Performance is unverified.** fbcp-ili9341 achieved 60fps through DMA optimisation. FBTFT is a generic kernel driver without the same low-level tuning — achieving 30fps at 320×240 on Pi Zero 2W through the `fbdev` KMS layer needs to be measured before committing to hardware. If FBTFT cannot hit 30fps, the DRM panel driver or a different display technology must be evaluated.
- FBTFT configuration requires a `dtoverlay` entry in `config.txt` specifying the panel, SPI bus, GPIO pins (reset, DC, CS), and bus speed. This is well-documented in the RPi OS FBTFT wiki.
- The driver is compatible with any current Raspberry Pi OS Bookworm (32-bit) image. No version pinning required — this ADR removes the provisional constraint from ADR-0004.
- RetroPie's EmulationStation and RetroArch can both target `/dev/fb1` (the FBTFT framebuffer device). Configuration is documented in the RetroPie community for FBTFT panels.
- SPI bus speed remains a tuning variable. Most ILI9341 panels support 40–62MHz; the Pi Zero 2W SPI controller can be pushed to ~125MHz. Higher clock speeds improve framerate directly — this is the primary lever if 30fps is marginal.
