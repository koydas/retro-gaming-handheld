# ADR-0006: Emulation Scope — Supported Systems and FPS Viability

**Date:** 2026-06-24  
**Status:** Accepted

## Context

The Pi Zero 2W has well-defined performance ceilings: four ARM Cortex-A53 cores at 1GHz and 512MB LPDDR2 RAM. These figures are sufficient for 8-bit and 16-bit era emulation but fall short of what is needed to run PlayStation 1, Nintendo 64, or Sega Saturn titles reliably at 30fps across their libraries. The 30fps minimum target is established in ADR-0001; the platform is RetroPie on Raspberry Pi OS Bookworm (32-bit) as decided in ADR-0004.

Establishing explicit scope prevents design drift: button layout, display resolution (320×240), audio latency targets, and Phase 5 software configuration all follow from the emulation target. An out-of-scope system is not a stretch goal — it is a system for which library-wide 30fps cannot be guaranteed on this hardware, whether because the hardware cannot run it at all (N64, Saturn) or because performance is too title-dependent for a reliable scope claim (PS1).

**Scope of this analysis:** This ADR covers CPU-side emulation performance only — whether each system's recommended core can sustain 30fps game logic on the Pi Zero 2W. Whether FBTFT can deliver those frames to the display at 30fps is a separate prerequisite covered in ADR-0005 (open question Q1 in CLAUDE.md). Both conditions must hold for end-to-end 30fps to be achievable. If FBTFT throughput proves insufficient, the display driver path must be revisited and the conclusions of this ADR will need to be re-evaluated against the actual displayed frame rate.

## Decision

**Supported systems (in scope):**

- Atari 2600 and 7800
- Sega Master System and Game Gear
- NES / Famicom
- Game Boy and Game Boy Color
- Sega Genesis / Mega Drive
- SNES / Super Famicom
- Game Boy Advance

**Explicitly out of scope:**

- PlayStation 1 (PS1 / PSX)
- Nintendo 64 (N64)
- Sega Saturn
- All later generations

The out-of-scope systems are excluded because library-wide 30fps cannot be guaranteed on Pi Zero 2W hardware. The reasons differ by system: N64 and Sega Saturn are a hard capability ceiling — no core or configuration makes them consistently viable at 30fps on this SoC. PlayStation 1 is excluded on two independent grounds: (1) library consistency — PCSX-ReARMed can run a meaningful subset of PS1 titles on Zero 2W, but performance is highly game-dependent and 30fps cannot be guaranteed across the PS1 library; (2) controller layout — this handheld provides D-pad, 4 face buttons, L1/R1, and Start/Select, with no L2/R2 and no analog sticks. A significant portion of the PS1 library requires analog input or L2/R2, making it unplayable on this hardware regardless of emulation performance. See Considered Alternatives for details.

## FPS Analysis — 30fps Minimum Target

Platform: RetroPie on Pi Zero 2W (4× ARM Cortex-A53 @ 1GHz, 512MB LPDDR2, Raspberry Pi OS Bookworm 32-bit).

| System | Recommended Core (RetroPie) | FPS Status | Notes / Mitigations |
|--------|----------------------------|------------|---------------------|
| Atari 2600 | Stella (`stella2014`) | **Solid** | Trivial CPU load; no mitigations needed |
| Atari 7800 | ProSystem | **Solid** | Trivial CPU load |
| NES / Famicom | FCEUmm | **Solid** | Consistent 60fps; FCEUmm is lightweight and mature |
| Sega Master System / Game Gear | Genesis Plus GX | **Solid** | |
| Game Boy / Game Boy Color | Gambatte | **Solid** | Gambatte is the lighter option; mGBA also solid on GBC but heavier |
| Sega Genesis / Mega Drive | Genesis Plus GX or PicoDrive | **Solid** | PicoDrive has ARM-native assembly paths; both cores are viable |
| SNES / Super Famicom | snes9x2010 | **Solid** | Do not use `snes9x-current` — too heavy for Zero 2W. **Exception — not covered by this Solid rating:** SA-1 chip games (Kirby Super Star) and SuperFX games (Star Fox, Yoshi's Island) may need `frameskip = 1` or drop below 30fps; see Consequences. |
| Game Boy Advance | gpSP | **Solid** | gpSP has ARM-native optimizations and achieves 60fps reliably. mGBA is **marginal** on Zero 2W — most titles pass 30fps but demanding games may dip; gpSP is the recommended default |

**Status definitions:**

- **Solid** — stable ≥30fps across the system's library with the recommended core; no special configuration required for the general case
- **Marginal** — 30fps achievable with specific core selection, configuration tuning, or frameskip; some titles may not qualify
- **Unreliable** — cannot consistently meet 30fps; excluded from supported scope

## Considered Alternatives

**Include PlayStation 1 (PCSX-ReARMed)**  
Rejected on two independent grounds.

*Performance:* PCSX-ReARMed is the standard PS1 core for ARM hardware and includes ARM-native assembly paths. On a Pi Zero 2W it runs a subset of PS1 titles, but the library is large and performance is highly game-dependent: CPU-intensive titles (Final Fantasy VII–IX, many RPGs with heavy geometry or FMV sequences) regularly drop below 30fps without aggressive configuration tuning, and frameskip introduces timing artefacts that are visible on a 320×240 display. The Pi Zero 2W's 512MB RAM is also a constraint for titles that use large texture pages. Guaranteeing 30fps across the PS1 library is not feasible on this hardware.

*Controller layout:* The PS1 DualShock has L1, L2, R1, R2, two analog sticks (with L3/R3 clicks), 4 face buttons, D-pad, Start, and Select. This handheld provides D-pad, 4 face buttons, L1/R1 shoulder buttons, and Start/Select — no L2/R2, no analog sticks. A large portion of the PS1 library either requires analog input (3D platformers, driving games, action games) or maps critical actions to L2/R2. These titles are unplayable on this hardware regardless of emulation performance, so including PS1 in scope would misrepresent what the device can actually play.

**Include Nintendo 64 (Mupen64Plus-Next)**  
Mupen64Plus is the standard N64 emulator on RetroPie. N64 requires software rendering at acceptable accuracy levels; the Pi Zero 2W has no GPU pipeline accessible to the emulation stack in a useful way. Even with aggressive resolution downscaling and frameskip, many N64 titles run at 15–20fps on this hardware. Rejected on hard performance ceiling grounds.

**Include Sega Saturn (Yabause or Kronos)**  
Saturn emulation is CPU-intensive and memory-hungry due to the console's dual-CPU architecture. Yabause and Kronos both require significantly more than what the Zero 2W provides. No viable configuration exists for the Zero 2W. Rejected on hardware ceiling grounds.

**Set scope to NES / Game Boy only (more conservative)**  
The Pi Zero 2W handles SNES and Genesis reliably with the correct cores. Excluding them would be unnecessarily restrictive. The SNES (snes9x2010) and Genesis (Genesis Plus GX / PicoDrive) have well-established, ARM-optimized cores with a documented history of running on constrained ARM hardware. Rejected as overly conservative.

## Consequences

- Button layout, display resolution (320×240), and audio configuration are designed around 8-bit and 16-bit era emulation. These are appropriate for the in-scope systems and do not need to accommodate N64 or PS1 controller layouts or higher audio bandwidth.
- The RetroPie installation in Phase 5 should enable only in-scope emulators. Shipping with N64 or PS1 cores enabled misleads testing and wastes SD card space.
- ADR-0001 Consequences previously noted that "N64 and PSP content will require downscaling and may stutter — this is accepted for the target platform scope." That framing is superseded by this ADR: PS1, N64, and Saturn are out of scope, not a tolerated tradeoff. ADR-0001 is updated accordingly.
- GBA (gpSP) and SNES (snes9x2010) are the most demanding in-scope systems and should be used as the performance benchmark during Phase 5 testing.
- SA-1 and SuperFX SNES titles (Star Fox, Yoshi's Island, Kirby Super Star) are a known edge case within the supported scope. If 30fps cannot be sustained with `frameskip = 1` on these specific titles, they are treated as exceptions — not a reason to revise the SNES scope entry.
- mGBA on GBA is documented as marginal rather than excluded. If gpSP accuracy issues affect specific titles during Phase 5, mGBA can be evaluated on a per-game basis; the default recommendation remains gpSP.
