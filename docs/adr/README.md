# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) for the retro gaming handheld project.

An ADR is a short document that captures a significant design or hardware choice: what was decided, why it was decided, and what alternatives were rejected and why. The point is not bureaucracy — it's to have a readable record so that, six months later, the answer to "why did I do it this way?" isn't a blank stare.

## Format

Each ADR follows this structure:

```
# ADR-NNNN: <short title>

**Date:** YYYY-MM-DD
**Status:** Accepted | Superseded by ADR-XXXX | Deprecated

## Context
What situation led to this decision? What constraints existed?

## Decision
What was chosen?

## Considered Alternatives
What else was evaluated, and why was each rejected?

## Consequences
What does this decision make easier or harder going forward?
```

## How to add a new ADR

1. Pick the next number in sequence.
2. Copy the format above into `docs/adr/NNNN-short-slug.md`.
3. Fill in all four sections — including honest rejections in **Considered Alternatives**.
4. Set **Status** to `Accepted`.
5. If the new ADR supersedes an old one, update the old ADR's status to `Superseded by ADR-NNNN`.

## Index

| # | Title | Status |
|---|-------|--------|
| [0001](0001-display-selection.md) | Display Selection | Accepted |
| [0002](0002-battery-and-charging.md) | Battery and Charging | Accepted |
| [0003](0003-enclosure-manufacturing.md) | Enclosure Manufacturing | Accepted |
| [0004](0004-os-selection.md) | OS Selection | Accepted |
| [0005](0005-display-driver.md) | Display Driver Stack | Accepted — provisional |
| [0006](0006-emulation-scope.md) | Emulation Scope and FPS Viability | Accepted |
