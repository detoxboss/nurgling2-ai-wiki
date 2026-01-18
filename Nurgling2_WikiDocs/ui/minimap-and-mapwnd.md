# Minimap and map window integration

## Key classes

- `nurgling.widgets.NMapWnd extends haven.MapWnd`
  - Hosts the minimap widget and map-related UI.
  - Intercepts some click flows for mod features.

- `nurgling.widgets.NMiniMap`
  - Minimap implementation and marker/overlay logic.

## Forager path recording hook

Forager path recording is handled in the map window click handler:

- When the Forager bot window is open *and* recording is enabled, a **plain left click** on the minimap adds a waypoint.
- The handler is deliberately gated behind “no modifiers” (no Ctrl/Shift/etc.), because minimap clicks with modifiers are used by other features.

Practical implication:
- While recording a forager path, do **not** hold modifiers when clicking the minimap.

