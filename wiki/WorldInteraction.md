
# World interaction

## Core types

- `nurgling.NMapView` extends `haven.MapView`.
- World objects are `haven.Gob` instances in `ui.sess.glob.oc`.

NMapView is where Nurgling concentrates:

- selection modes (areas, gobs, measurement)
- overlays (tile/gob highlights)
- routing and navigation helpers
- click destination “path line” drawing

## Sending world actions

Nurgling standardizes most world action messages in `nurgling.NUtils`.

Common operations:

- **Left-click gob**: `NUtils.clickGob(gob)`
- **Right-click gob**: `NUtils.rclickGob(gob)`
- **Activate with held item**: `NUtils.activateItem(gob, shift)`
- **Open/activate gob**: `NUtils.activateGob(gob)`

Under the hood these send `map.wdgmsg("click"...)` and `map.wdgmsg("itemact"...)` payloads.

If you are implementing automation, prefer these helpers so your code stays consistent with existing client conventions.

## Selection modes in `NMapView`

NMapView tracks a number of interactive modes:

- `isAreaSelectionMode` – selecting ground rectangles
- `isGobSelectionMode` – selecting gobs
- `isChatAreaSharingMode` – Alt+Ctrl+LMB sharing
- `zoneMeasureMode` / `zoneClearMode` – measurement tool

It also tracks:

- `selectedGob`
- current selection coords while dragging

When adding a new tool mode, follow the existing pattern:

- add an atomic boolean flag
- capture input in `mousedown/mousemove/mouseup`
- render overlays in `draw()` or via a `Gob.Overlay`

## Navigation integrations

NMapView owns a **non-singleton** `ChunkNavManager`:

- `initializeWithGenus(genus)` should be called once the profile/genus is known.
- the manager is accessed via `getChunkNavManager()`.

The chunk navigation system is Nurgling’s higher-level pathfinding/navigation layer used by several automation bots.

## Overlays

NMapView initializes certain overlays on first draw:

- rock tile highlighting

It also uses dummy gobs and overlays for area labels and route markers.

If you are adding an overlay, try to:

- keep the rendering side effects inside the render/tick thread
- avoid per-frame allocations where possible (NMapView includes caching patterns)
