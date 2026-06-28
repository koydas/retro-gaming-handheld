# adr-workflow

## When to Apply
When creating a new ADR, updating an existing ADR, or making any change that affects a hardware decision, OS selection, driver choice, enclosure manufacturing, emulation scope, or power topology.

## Expected Behavior

1. **Read first**: Before writing, read all existing ADRs in `docs/adr/` to understand what is already decided and why. Never write an ADR that contradicts an existing one without explicitly superseding it.

2. **Pick the next number**: Check the index in `docs/adr/README.md` and increment from the last entry. File name format: `NNNN-short-slug.md`.

3. **Write exactly four sections** in this order — no others:
   - **Context** — situation, constraints, what forced the decision
   - **Decision** — what was chosen and how it is configured
   - **Considered Alternatives** — every evaluated option with a specific reason for rejection; "not considered" is not a reason
   - **Consequences** — what is now easier, what is now harder, what open questions remain

4. **Include the date/status header** immediately after the title, following the format in `docs/adr/README.md`:
   ```
   **Date:** YYYY-MM-DD
   **Status:** Accepted
   ```
   Valid Status values: `Accepted`, `Accepted — provisional` (if an open question leaves a dependency unresolved), or `Superseded by ADR-XXXX`. Do not omit the `**Date:**` line — existing ADRs in this repo always include it.

5. **Update the index**: Add the new ADR to the table in `docs/adr/README.md`.

6. **If superseding an old ADR**: Update the old ADR's Status line to `Superseded by ADR-NNNN` and document the correction in the new ADR's Context section. Both changes go in the same commit.

7. **Cross-consistency check** (mandatory after every ADR add or edit):
   - Read every ADR in `docs/adr/`.
   - Verify that every fact stated in multiple ADRs agrees: hardware specs, OS version, driver names, FPS targets, enclosure dimensions, emulation scope, open question status.
   - Verify that every cross-reference names the correct ADR number and accurately describes what that ADR says.
   - Fix any contradiction or gap found — in the same commit. Do not defer.

8. **Update living docs**: If the decision changes GPIO assignments or component status, update `docs/gpio-map.md` and/or `docs/bom.md` in the same commit (see `living-docs-sync` skill).

9. **Update README hardware table** if the decision changes a component listed there.

## Constraints
- Do not add sections beyond the four required.
- Do not write "not considered" as a rejection reason — name the specific reason.
- Do not use marketing language or hedge rejections as "could be revisited."
- Do not leave cross-ADR contradictions for a later commit.
- Do not supersede an ADR without simultaneously creating its replacement.
- Be explicit about open questions; do not paper over uncertainty.

## References
- `docs/adr/README.md` — format reference and ADR index
- `docs/adr/0001-display-selection.md` through `0006-emulation-scope.md` — all current ADRs
- CLAUDE.md — "Architecture Decision Records" section
