# CLAUDE.md — Retro Gaming Handheld

This file gives Claude the context needed to work on this project without re-deriving it from scratch each session.

## What this project is

A scratch-built retro gaming handheld based on the Raspberry Pi Zero 2W, running RetroPie on Raspberry Pi OS (32-bit). It is a **learning project**, not a product. The primary goals are PCB design, power delivery, and mechanical tolerances. Playing Pokémon is a bonus.

Current phase: **Phase 1 — Design** (specs and layout, no hardware in hand yet).

## Non-negotiable physical constraints

| Constraint | Value | Source |
|---|---|---|
| Enclosure target | 149 × 68 × 22mm | README |
| SBC | Pi Zero 2W, 30 × 65mm | README |
| SBC width budget | ~19mm each side for walls + controls | derived from enclosure width |
| Display area | ~2.4", flush in 22mm depth | design target |

**Do not propose alternatives that violate these.** A Pi 3A+ (56mm wide) does not fit. A thick HDMI panel does not fit.

## Established hardware facts

- The Pi Zero 2W exposes **CSI (camera), mini-HDMI, micro-USB, and 40-pin GPIO**. It does **not** have a DSI display port. Any ADR or suggestion assuming DSI on this board is wrong.
- Display interface: **SPI** (ILI9341 or compatible), driven by **FBTFT (`fb_ili9341` kernel module)** on RPi OS Bookworm/KMS. Do not suggest fbcp-ili9341 — it requires DispmanX which is not available on current RPi OS. 30fps at 320×240 is the target; FBTFT performance on Pi Zero 2W is unverified (see ADR-0005).
- SPI occupies several GPIO pins — the button matrix layout must route around them.
- Battery: single-cell 3.7V LiPo ~4000mAh, charged via TP4056 module **with DW01A protection IC**. Boost converter required for 5V rail.
- Enclosure: FDM on Bambu Lab printer, PETG for final print, PLA acceptable for fit-check iterations.

## Architecture Decision Records

All significant hardware and design decisions are documented in `docs/adr/`. Read the ADRs before proposing changes to established decisions — if a decision needs to change, write or update an ADR rather than silently changing the README or hardware table.

**If you write an ADR**, follow the format in `docs/adr/README.md`:
- Four sections: Context, Decision, Considered Alternatives, Consequences.
- Rejected alternatives must explain *why* they were rejected. "Not considered" is not a reason.
- Be honest about tradeoffs. No marketing language.
- If an ADR supersedes a previous one, update the old ADR's status line and document the correction in the new ADR's Context.

## Open questions — verify before hardware procurement

These are unresolved uncertainties that must be answered before committing to components or layout. Do not treat them as solved.

| # | Question | Blocks |
|---|----------|--------|
| Q1 | Does FBTFT (`fb_ili9341`) on Pi Zero 2W + RPi OS Bookworm achieve 30fps at 320×240? fbcp-ili9341 did via DMA optimisation; FBTFT is a generic kernel driver and its throughput on this hardware is unmeasured. | Display module procurement, ADR-0005 |
| Q2 | Charge-and-play topology: is there space on the PCB for a true power-path IC (e.g., BQ24074)? CN3165 is a CC/CV charger — same problem as TP4056 — and is not a valid option here. If space does not allow a power-path IC, device must be powered off to charge. | ADR-0002, Phase 2 PCB layout |
| Q3 | Does Raspberry Pi OS Trixie (Debian 13, 32-bit) support FBTFT (`fb_ili9341`) and the RetroPie installer on Pi Zero 2W? Bookworm is now the Legacy release. ADR-0004 pins to Bookworm until this is verified. | ADR-0004, Phase 5 |

## Spec-first discipline

This project is in R&D. No hardware has been purchased. **Verify specs against manufacturer documentation before writing or updating ADRs.** The DSI assumption in ADR-0001 v1 is an example of what happens when this step is skipped.

When in doubt about a hardware claim, flag it explicitly rather than asserting it confidently.

## PR review comments

When reading review comments on a PR (from Codex or humans), the full workflow is:

1. **Fix** — apply the correction in the codebase (code, ADR, README, whatever is wrong).
2. **Resolve** — mark the review thread as resolved on GitHub.
3. **Reply** — post a reply on the thread explaining what was changed and why.
4. **Re-trigger Codex** — post a comment on the PR with `@codex review` so Codex re-reviews the updated content.

Do all four steps, in order, for every actionable comment. Do not fix silently without replying, and do not reply without fixing.

## Language

- Code, filenames, and documentation: **English**
- Conversation with the user: **French**
