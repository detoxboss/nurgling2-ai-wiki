# Chunk navigation (ChunkNav)

## Purpose

Chunk navigation is a navigation subsystem that builds a graph of traversable “chunks”/tiles and can route the player to targets such as areas.

## Key classes

- `nurgling.navigation.ChunkNavManager`
  - Maintains navigation data.
  - Provides methods like `navigateToArea(NArea area, NGameUI gui)`.

- `nurgling.actions.bots.GotoArea`
  - Scenario-only bot step that delegates to `ChunkNavManager` to reach an `NArea`.

## How `GotoArea` works

`GotoArea`:
1. Reads an `areaId` scenario parameter.
2. Resolves it through `gui.map.glob.map.areas`.
3. Fetches `ChunkNavManager` from the `NMapView`.
4. Calls `navigateToArea(...)`.

Operational requirements:
- ChunkNav must be initialized and have sufficient data.
- The target area must exist and be reachable.

