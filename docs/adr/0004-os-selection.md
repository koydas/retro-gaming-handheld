# ADR-0004: OS Selection — RetroPie on Raspberry Pi OS

**Date:** 2026-06-23  
**Status:** Accepted — provisional *(specific RPi OS / RetroPie version with DispmanX support unconfirmed; see Decision and Consequences)*

## Context

The project initially targeted Batocera Linux as the operating system, chosen for its reputation as a ready-to-use retro gaming image. During ADR-0001 review, a driver compatibility issue surfaced: fbcp-ili9341 — the SPI display driver required by the chosen display interface — is built on DispmanX, the legacy Pi graphics stack. Batocera's current builds for Pi use KMS/DRM, which is incompatible with DispmanX. Batocera's own TFT documentation confirms that SPI/GPIO screens require pre-compiled driver binaries and that support may be limited to older 32-bit images. This made the display path uncertain and blocked ADR-0001 from being finalised.

The OS choice therefore directly affects whether the display works at all, and must be resolved before procurement.

## Decision

Use **RetroPie** on **Raspberry Pi OS (32-bit)**.

RetroPie is built on top of Raspberry Pi OS, which historically maintained the DispmanX graphics stack on 32-bit builds — the environment fbcp-ili9341 was originally developed and tested against, with well-documented community setups on Pi Zero hardware.

**However, this assumption is not confirmed for current releases.** The fbcp-ili9341 README (Feb 2024 update) notes that later Raspberry Pi OS releases no longer have DispmanX active by default, even on 32-bit images. A specific RetroPie release / Raspberry Pi OS version where DispmanX remains functional must be identified and pinned before procurement. The alternative is to switch to a KMS-compatible SPI driver (e.g., the `fb_ili9341` kernel module), which would remove the DispmanX dependency entirely and allow any current RPi OS version to work.

This ADR is therefore provisional pending one of: (a) identification of a pinned RetroPie/RPi OS release with confirmed DispmanX support on Pi Zero 2W; or (b) confirmation that a KMS-compatible driver path works for the chosen ILI9341 panel.

## Considered Alternatives

**Batocera Linux**  
The original choice. Batocera is a polished, self-contained gaming image with a good frontend and broad emulator support. Rejected because its current Pi builds use KMS/DRM, which breaks fbcp-ili9341 and leaves the SPI display without a confirmed driver path. Resolving this would require pinning an older Batocera release or sourcing a pre-compiled DispmanX-compatible binary — neither is verified, and both add fragility to a first build. Rejected on display driver compatibility grounds.

**Plain Raspberry Pi OS (Lite) without RetroPie**  
Technically viable — fbcp-ili9341 works, and emulators can be installed manually. Rejected because the manual frontend and emulator configuration overhead is out of scope for this project. RetroPie packages the same foundation with a usable gaming layer on top.

**Lakka**  
Lakka is a LibreELEC-based gaming OS with a clean frontend. It has the same KMS/DRM issue as Batocera for fbcp-ili9341. Additionally, Lakka documentation for SPI displays on Pi Zero is sparse compared to RetroPie. Rejected on the same driver grounds as Batocera.

**Recalbox**  
Similar situation to Batocera — modern builds use KMS, SPI display support is not well documented for the Pi Zero 2W. Rejected on driver compatibility grounds.

## Consequences

- **ADR-0001 display driver path remains provisional.** RetroPie is the correct OS direction, but the specific image version must be pinned to one where DispmanX is available, or the driver must be switched to a KMS-compatible alternative. This is a Phase 5 prerequisite — verify before ordering the display module.
- RetroPie requires more manual configuration than Batocera (emulator setup, controller mapping, scraping). This is accepted — the project is a learning exercise and software configuration is within scope.
- The 32-bit OS constraint is not a problem for the Pi Zero 2W: 512MB RAM is below the threshold where 64-bit addressing provides a benefit, and 32-bit images are well-supported on this hardware.
- RetroPie receives less frequent updates than Batocera. For a stationary emulation device that is not networked, this is not a concern.
- Phase 5 setup steps (README) must reference RetroPie configuration rather than Batocera image flashing.
