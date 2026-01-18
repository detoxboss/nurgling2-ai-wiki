# Areas and specialisations

## Overview

Areas are rectangular (or multi-grid) regions stored in `gui.map.glob.map.areas`.

They are used for:
- navigation targets (e.g., `GotoArea`)
- storage automation (bots search for specific areas)
- “specialisations”: semantic labels that let bots locate functional areas (e.g., “curding tubs”, “compost bins”).

## Key classes

- `nurgling.areas.NArea`
  - Data model for an area.
  - Supports a list of `Specialisation` values (`name` + optional `subtype`).

- `nurgling.areas.NContext`
  - Utility layer for finding areas by specialisation and other criteria.

## Specialisations in practice

Bots that work with specialised areas typically:
- define a `new NArea.Specialisation("SomeName")`
- use `NContext.findSpec(...)` or similar helper

This provides a stable contract for automation even when multiple areas exist with similar shapes.

