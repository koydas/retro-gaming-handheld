# GPIO Pin Allocation — Pi Zero 2W

Tracks which of the 40 GPIO header pins are spoken for and which are available for the button matrix PCB. Update this document whenever a pin assignment is added, moved, or confirmed in Phase 2.

## Summary

| Category | GPIO count | Pins |
|----------|-----------|------|
| SPI0 display bus (hardware-fixed) | 4 | GPIO 8, 9, 10, 11 |
| Display control — DC, RST, BL (tentative) | 3 | GPIO 18, 25, 27 |
| UART — reserved for debug | 2 | GPIO 14, 15 |
| Available for button matrix | 15 | see table below |
| I2C (avoid unless needed) | 2 | GPIO 2, 3 |
| HAT EEPROM (not usable as GPIO) | — | pins 27/28 |
| Power / GND | — | pins 1, 2, 4, 6, 9, 14, 17, 20, 25, 30, 34, 39 |

15 GPIO available for buttons. 12 buttons direct-wired fits with 3 spare; a matrix approach (e.g., 4×3) would use 7 and leave more headroom.

---

## Full 40-pin Reference

| Board pin | BCM GPIO | Hardware alt | Assignment | Status |
|-----------|----------|-------------|------------|--------|
| 1  | —       | 3.3V        | Power rail  | Power |
| 2  | —       | 5V          | Power rail  | Power |
| 3  | GPIO 2  | I2C1 SDA    | —           | **Avoid** — keep I2C free |
| 4  | —       | 5V          | Power rail  | Power |
| 5  | GPIO 3  | I2C1 SCL    | —           | **Avoid** — keep I2C free |
| 6  | —       | GND         | —           | GND |
| 7  | GPIO 4  | GPCLK0      | —           | **Available** |
| 8  | GPIO 14 | UART TX     | UART debug  | **Reserved** |
| 9  | —       | GND         | —           | GND |
| 10 | GPIO 15 | UART RX     | UART debug  | **Reserved** |
| 11 | GPIO 17 | —           | —           | **Available** |
| 12 | GPIO 18 | PWM0        | Display BL (backlight, tentative) | Tentative |
| 13 | GPIO 27 | —           | Display RST (tentative) | Tentative |
| 14 | —       | GND         | —           | GND |
| 15 | GPIO 22 | —           | —           | **Available** |
| 16 | GPIO 23 | —           | —           | **Available** |
| 17 | —       | 3.3V        | Power rail  | Power |
| 18 | GPIO 24 | —           | —           | **Available** |
| 19 | GPIO 10 | SPI0 MOSI   | Display MOSI | **Fixed — SPI0** |
| 20 | —       | GND         | —           | GND |
| 21 | GPIO 9  | SPI0 MISO   | SPI0 bus (occupied even if display is write-only) | **Fixed — SPI0** |
| 22 | GPIO 25 | —           | Display DC (tentative) | Tentative |
| 23 | GPIO 11 | SPI0 SCLK   | Display SCLK | **Fixed — SPI0** |
| 24 | GPIO 8  | SPI0 CE0    | Display CS  | **Fixed — SPI0** |
| 25 | —       | GND         | —           | GND |
| 26 | GPIO 7  | SPI0 CE1    | —           | **Available** — but requires `dtoverlay=spi0-1cs` in `/boot/firmware/config.txt`; SPI0 claims both CE0 and CE1 by default, even with one device attached |
| 27 | —       | ID_SD       | HAT EEPROM  | Not usable as GPIO |
| 28 | —       | ID_SC       | HAT EEPROM  | Not usable as GPIO |
| 29 | GPIO 5  | —           | —           | **Available** |
| 30 | —       | GND         | —           | GND |
| 31 | GPIO 6  | —           | —           | **Available** |
| 32 | GPIO 12 | PWM0        | —           | **Available** |
| 33 | GPIO 13 | PWM1        | —           | **Available** |
| 34 | —       | GND         | —           | GND |
| 35 | GPIO 19 | SPI1 MISO   | —           | **Available** (SPI1 capable; usable as GPIO since SPI1 not in use) |
| 36 | GPIO 16 | SPI1 CE2    | —           | **Available** |
| 37 | GPIO 26 | —           | —           | **Available** |
| 38 | GPIO 20 | SPI1 MOSI   | —           | **Available** |
| 39 | —       | GND         | —           | GND |
| 40 | GPIO 21 | SPI1 SCLK   | —           | **Available** |

---

## Notes

**SPI0 (fixed by hardware):** GPIO 8, 9, 10, 11 are the SPI0 bus. GPIO 9 (MISO) is occupied by SPI0 even though the ILI9341 display is write-only — the Pi SPI controller holds the pin. Do not route anything else to these four pins.

**GPIO 7 (SPI0 CE1) — conditional:** Under the default `dtparam=spi=on`, the SPI0 controller claims both chip-select lines: CE0 on GPIO 8 and CE1 on GPIO 7. GPIO 7 is available for button use only if `dtoverlay=spi0-1cs` is added to `/boot/firmware/config.txt`, which restricts SPI0 to CE0 only. Phase 5 software configuration must include this overlay if GPIO 7 is assigned to a button in Phase 2.

**Display control pins (tentative):** GPIO 25 (DC), GPIO 27 (RST), and GPIO 18 (BL) are community-standard choices for FBTFT dtoverlay on ILI9341. They are not hardware-fixed — any free GPIO can be used, configured via the stock `dtoverlay=fbtft` overlay with the `ili9341` controller parameter in `/boot/firmware/config.txt`. The exact parameter syntax must be confirmed against RPi OS Bookworm documentation before Phase 5 — the README Phase 5 checklist reflects this open item. Finalise these assignments in Phase 2 before PCB routing. If backlight is hardwired to 3.3V (no PWM dimming), GPIO 18 is freed.

**UART (reserved):** GPIO 14/15 are kept free for serial debug access during bring-up. They can be repurposed for buttons after Phase 5 is stable, but blocking them now avoids rework.

**Available for button matrix:** GPIO 4, 7*, 17, 22, 23, 24, 5, 6, 12, 13, 16, 19, 20, 21, 26 — 15 pins. Button count is ~12 (D-pad ×4, face ×4, shoulder ×2, Start/Select ×2); direct wiring uses 12 pins and leaves 3 free. A 4×3 matrix uses 7 pins if GPIO is tight. *GPIO 7 requires `dtoverlay=spi0-1cs` in config.txt — see GPIO 7 note above.

**I2C (avoid unless needed):** GPIO 2/3 are the primary user-facing I2C bus (I2C1, `/dev/i2c-1`). I2C0 is the HAT EEPROM bus on pins 27/28 — separate bus, not general-purpose. No I2C device is planned, but keeping I2C1 free costs nothing and avoids rework if a sensor or expansion is added later.
