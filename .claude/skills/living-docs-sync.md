# living-docs-sync

## When to Apply
When making any change that affects GPIO pin assignments, peripheral connections, component selection, procurement status, blocking question resolution (Q1, Q2, Q3), or any new hardware component being introduced. These two files must never be allowed to drift from the ADRs.

## Expected Behavior

### `docs/gpio-map.md` — update when:
- A GPIO pin is assigned to a new function (display control, button, peripheral)
- A tentative assignment is confirmed or changed
- A pin is freed (e.g., backlight hardwired to 3.3V frees GPIO 18)
- Phase 2 finalises the display control pin assignments (DC/RST/BL)

**How to update:**
1. Edit the row in the Full 40-pin Reference table: BCM GPIO, hardware alt, assignment description, status.
2. Update the Summary table at the top: adjust counts and pin lists for the affected category.
3. Verify the updated pin does not conflict with: SPI0 (GPIO 8/9/10/11 — fixed), UART debug (GPIO 14/15 — reserved), I2C1 (GPIO 2/3 — avoid).
4. If GPIO 7 is assigned to a button, note the `dtoverlay=spi0-1cs` requirement in the Notes section.
5. Mark display control pins (GPIO 18/25/27) as **Tentative** until Phase 2 PCB routing locks them; only then change to a confirmed assignment.

### `docs/bom.md` — update when:
- A part's status changes (can order → ordered → received, or blocked → unblocked)
- A blocking question (Q1, Q2, Q3) resolves — update the Blocking question / note column for every affected row and revise the Procurement Order sequence accordingly
- A new component is introduced by a hardware decision

**How to update:**
1. Edit the relevant row's Procurement status and/or Blocking question / note columns.
2. If Q2 resolves to "power-path IC required", add or activate row #10 (BQ24074 or equivalent).
3. If Q2 resolves to "power-off-to-charge", drop row #10 from the BOM.
4. Do not add PCB passives (resistors, caps, diodes), display ribbon cables / FPC connectors, or power wiring — those belong in the Phase 2 KiCad BOM.

### Both files — after any update:
- Check consistency with all ADRs in `docs/adr/`.
- Check consistency with the Hardware table in `README.md`.
- Check consistency with each other (a GPIO assignment must not conflict with a BOM component that uses the same pin).
- Include the updates in the **same commit** as the ADR or decision that triggered them.

## Constraints
- Do not add PCB passives, display FPC connectors, or power wiring to `docs/bom.md`.
- Do not change a GPIO assignment without first verifying it does not conflict with SPI0, UART, or I2C1 pins.
- Do not mark display control pins (GPIO 18/25/27) as confirmed until Phase 2 finalises them.
- Do not commit an ADR change without simultaneously updating these files if they are affected.
- Do not update these files without cross-checking the ADRs — partial updates that contradict the ADRs are worse than no update.

## References
- `docs/gpio-map.md` — GPIO pin allocation reference
- `docs/bom.md` — Bill of materials and procurement status
- CLAUDE.md — "Reference documents" section
- `docs/adr/0001-display-selection.md` — SPI0 bus constraint
- `docs/adr/0002-battery-and-charging.md` — BOM rows 4, 5, 10; Q2 resolution paths
