# Diagrams

Visual overviews of the retro gaming handheld architecture. All diagrams are Mermaid and render on GitHub.

---

## 1. System Architecture Overview

Top-level hardware block diagram showing how all components connect.

```mermaid
flowchart LR
    subgraph POWER["Power System"]
        USB_PORT["micro-USB\nCharging Port"]
        TP_MOD["TP4056 Module\nTP4056 charger + DW01A protection"]
        LIPO["3.7V LiPo\n~4000mAh"]
        BOOST["5V Boost Converter\n3.0–4.2V → 5V, ≥1A"]
    end

    subgraph COMPUTE["SBC"]
        PI["Raspberry Pi Zero 2W\n1GHz quad-core · 512MB RAM\n30×65mm"]
    end

    subgraph DISP["Display"]
        ILI["2.4 inch ILI9341 TFT\n320×240 · SPI\nFBTFT fb_ili9341"]
    end

    subgraph INPUT["Input"]
        BTNS["Button Matrix\nD-pad x4 · Face x4\nL/R x2 · Start/Select x2\n12 buttons total"]
    end

    USB_PORT -->|5V in| TP_MOD
    TP_MOD <-->|charge / protect| LIPO
    LIPO -->|3.7V| BOOST
    BOOST -->|5V regulated| PI
    PI <-->|SPI0 bus\nGPIO 8/9/10/11| ILI
    PI <-->|GPIO\n12 pins| BTNS
```

---

## 2. Power Delivery Topology

Detailed power path from battery cell through protection, boost conversion, and load. The **Q2 open question** (charge-and-play topology) is shown as an unresolved decision branch that must be resolved before Phase 2 PCB layout.

```mermaid
flowchart TD
    USB["micro-USB 5V\nCharging Input"]

    subgraph TP_MODULE["TP4056 Module (single PCB)"]
        direction TB
        TP["TP4056\nCC/CV charger\n1A fixed charge rate\n0.25C for 4000mAh cell"]
        DW["DW01A\nOver-discharge cutout ~2.5V per cell\nOver-voltage protection ~4.3V"]
    end

    LIPO["LiPo Cell\n3.7V nominal · ~4000mAh\n~14.8Wh raw / ~11.8Wh usable at 5V"]

    BOOST["5V Boost Converter\nInput 3.0–4.2V  Output 5V, >=1A\nMT3608 or XL6009 — TBD"]

    PI_LOAD["Raspberry Pi Zero 2W\n~0.5W idle · ~1.5W peak"]
    DISP_LOAD["ILI9341 Display\n~0.3W"]

    RUNTIME["Estimated runtime\nTarget: 2+ h at moderate load\nRealistic: 3–4 h play\nTheoretical max: ~7.8 h"]

    Q2{{"Q2 — UNRESOLVED\nCharge-and-play topology:\nTP4056 is not a power-path IC.\nBoost draw prevents CC/CV\ntermination while Pi runs."}}

    OPT_A["Option A — preferred if space allows\nPower-path IC e.g. BQ24074\nSeparates charge path from system load\nCorrect termination while Pi runs"]

    OPT_B["Option B — space-constrained fallback\nPower-off-to-charge constraint\nDevice must be off before USB connect\nSimpler PCB, but no hot-swap"]

    USB --> TP
    TP <--> DW
    DW <-->|charge / protect| LIPO
    DW -->|protected discharge\nTP4056 OUT+/OUT-| BOOST
    BOOST --> PI_LOAD
    BOOST --> DISP_LOAD
    PI_LOAD --> RUNTIME
    DISP_LOAD --> RUNTIME

    BOOST -. "Decide in Phase 2\nbefore PCB layout" .-> Q2
    Q2 --> OPT_A
    Q2 --> OPT_B
```

---

## 3. Display Driver Software Stack

Software and kernel layers from the SPI hardware bus up to the RetroPie UI. Rejected driver alternatives are shown separately.

```mermaid
flowchart TD
    subgraph CHOSEN["Chosen Driver Stack"]
        direction TB
        RP["RetroPie\nInstall and config layer"]
        ES["EmulationStation\nGame browser and launcher"]
        RA["RetroArch\nEmulator runner — targets /dev/fb1"]
        FBDEV["fbdev KMS emulation\nTranslates framebuffer calls\ninto KMS/DRM pipeline"]
        FB1["/dev/fb1\nFramebuffer device"]
        FBTFT["fb_ili9341  FBTFT kernel module\nLoaded via dtoverlay\nRPi OS Bookworm · KMS/DRM stack\nQ1: 30fps at 320x240 UNVERIFIED"]
        ILI_HW["ILI9341 Panel  2.4 inch\nDC GPIO25 · RST GPIO27 · BL GPIO18 (tentative)\nRequires >=40MHz SPI clock"]
        SPI_BUS["SPI0 Hardware Bus\nMOSI GPIO10 · MISO GPIO9 · SCLK GPIO11 · CS GPIO8"]
    end

    subgraph REJECTED["Rejected Alternatives"]
        FBCP["fbcp-ili9341\nREJECTED: requires DispmanX\nnot available on RPi OS Bookworm\nupstream stalled since 2019"]
        DRM_DRV["panel-ilitek-ili9341  DRM native\nNot chosen for v1: sparse docs\nfor ILI9341 + RetroPie setup\nFallback if FBTFT fails Q1"]
    end

    RP --> ES --> RA --> FBDEV --> FB1 --> FBTFT --> ILI_HW --> SPI_BUS
```

---

## 4. GPIO Pin Allocation

Pi Zero 2W GPIO pins grouped by assignment. Power and GND pins omitted. Pins 27/28 (HAT EEPROM) are not general-purpose GPIO and are excluded.

```mermaid
flowchart LR
    PI["Raspberry Pi Zero 2W\n40-pin GPIO Header"]

    SPI0["SPI0 Bus — FIXED\nGPIO 8  SPI0 CE0 / Display CS\nGPIO 9  SPI0 MISO\nGPIO 10 SPI0 MOSI\nGPIO 11 SPI0 SCLK\nDo not route anything else here"]

    DISP_CTRL["Display Control — TENTATIVE\nGPIO 25  DC data/command\nGPIO 27  RST reset\nGPIO 18  BL backlight PWM0\nFinalise before Phase 2 PCB routing\nAny free GPIO can substitute"]

    UART_PINS["UART Debug — RESERVED\nGPIO 14  TX\nGPIO 15  RX\nKeep free for Phase 5 bring-up\nCan release for buttons after stable"]

    I2C_PINS["I2C1 — AVOID\nGPIO 2  SDA\nGPIO 3  SCL\nNo I2C device planned\nKeep free to avoid rework"]

    COND_PIN["Conditional — GPIO 7\nSPI0 CE1 by default\nFree only if dtoverlay=spi0-1cs\nadded to config.txt"]

    BTN_PINS["Available for Button Matrix — 14 pins\nGPIO 4 5 6 12 13 16 17\nGPIO 19 20 21 22 23 24 26\n12 buttons direct-wired: 2 spare\n(+1 if GPIO 7 freed via dtoverlay=spi0-1cs)\n4x3 matrix 7 pins: 7 spare"]

    PI --> SPI0
    PI --> DISP_CTRL
    PI --> UART_PINS
    PI --> I2C_PINS
    PI --> COND_PIN
    PI --> BTN_PINS

    classDef fixed fill:#c62828,stroke:#b71c1c,color:#fff
    classDef tentative fill:#e65100,stroke:#bf360c,color:#fff
    classDef reserved fill:#1565c0,stroke:#0d47a1,color:#fff
    classDef avoid fill:#616161,stroke:#424242,color:#fff
    classDef conditional fill:#f57f17,stroke:#e65100,color:#fff
    classDef available fill:#2e7d32,stroke:#1b5e20,color:#fff

    class SPI0 fixed
    class DISP_CTRL tentative
    class UART_PINS reserved
    class I2C_PINS avoid
    class COND_PIN conditional
    class BTN_PINS available
```

**Legend:** 🔴 Fixed (hardware SPI0) · 🟠 Tentative (confirm before routing) · 🔵 Reserved (UART debug) · ⬛ Avoid (I2C, keep free) · 🟡 Conditional (needs dtoverlay override) · 🟢 Available (button matrix)

---

## 5. Project Phases and Dependencies

Phase 1 through Phase 5 with the three open questions (Q1, Q2, Q3) shown as explicit blockers.

```mermaid
flowchart TD
    P1(["Phase 1 — Design  ACTIVE\nDimensions · button ergonomics\nGPIO map · battery cavity sizing\nPi mounting strategy"])

    Q1{{"Q1 — Must resolve before full\ndisplay procurement:\nDoes FBTFT hit 30fps at 320x240\non Pi Zero 2W + Bookworm?\nOrder 1x ILI9341 and measure."}}

    Q1_PASS["Q1 Pass\nFBTFT viable at 30fps\nProceed with ILI9341 procurement"]
    Q1_FAIL["Q1 Fail\nEvaluate DRM native panel driver\nor alternative display technology"]

    Q2{{"Q2 — Must resolve during Phase 2:\nCharge-and-play topology.\nIs there PCB space for\na power-path IC e.g. BQ24074?"}}

    P2(["Phase 2 — PCB Facade\nKiCad schematic and layout\nJLCPCB DFM and order\nGPIO assignments finalised"])

    P3(["Phase 3 — Enclosure\nCAD model in Bambu Studio\nFit-check print in PLA\nTolerance refinement\nFinal print in PETG"])

    P4(["Phase 4 — Assembly\nPower wiring: LiPo to TP4056\nBoost to Pi · PCB facade seated\nDisplay installed · smoke test"])

    Q3{{"Q3 — Evaluate before Phase 5 OS install:\nDoes RPi OS Trixie Debian 13\nsupport FBTFT + RetroPie\non Pi Zero 2W?"}}

    Q3_BOOK["Pin to Bookworm Legacy\nKnown working with FBTFT\nand RetroPie installer"]
    Q3_TRIX["Upgrade path to Trixie\nIf verified compatible\nbefore Phase 5"]

    P5(["Phase 5 — Software\nOS flash · RetroPie install\nFBTFT dtoverlay config\nButton mapping · ROM load\nBenchmark with GBA and SNES"])

    P1 --> Q1
    Q1 -->|"FBTFT viable"| Q1_PASS
    Q1 -->|"under 30fps"| Q1_FAIL
    Q1_PASS --> P2

    P1 --> Q2
    Q2 --> P2

    P1 --> P3

    P2 --> P4
    P3 --> P4

    P4 --> Q3
    Q3 -->|"Trixie unverified\ncurrent safe choice"| Q3_BOOK
    Q3 -->|"verify compatibility\nbefore committing"| Q3_TRIX
    Q3_BOOK --> P5
    Q3_TRIX --> P5
```

---

## 6. Emulation Scope

Systems supported at 30fps vs explicitly out of scope, with the reasoning for each exclusion.

```mermaid
mindmap
  root((Pi Zero 2W\nEmulation Scope))
    In Scope  30fps Solid
      Atari 2600 / 7800
        Stella and ProSystem cores
        Trivial CPU load
      NES / Famicom
        FCEUmm core
        Consistent 60fps
      Game Boy / Game Boy Color
        Gambatte core
      Sega Master System / Game Gear
        Genesis Plus GX core
      Sega Genesis / Mega Drive
        Genesis Plus GX or PicoDrive
      SNES / Super Famicom
        snes9x2010 core
        Exception: SA-1 and SuperFX chips may dip
      Game Boy Advance
        gpSP core  solid 60fps ARM-native optimisations
        mGBA core  marginal  some titles dip below 30fps
    Out of Scope
      PlayStation 1
        Performance not guaranteed library-wide
        Missing L2 R2 and analog sticks
        Large portion of library requires both
      Nintendo 64
        Hard performance ceiling 15 to 20fps
        Software rendering only  no GPU access
      Sega Saturn
        Dual-CPU architecture exceeds Zero 2W
        Memory requirements too high
      All later generations
        Beyond hardware ceiling
```
