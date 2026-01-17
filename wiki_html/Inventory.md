
# Inventory and item model

## Core types

In Haven’s UI model, inventory is a composition of three key layers:

- `GItem` – the item “model” (resource, tooltip state, server hooks)
- `WItem` – the item “view” (a widget that draws a `GItem` in a slot)
- `Inventory` – the container widget that positions `WItem` instances on a slot grid

Nurgling overrides all three:

- `haven.GItem` factory creates `nurgling.NGItem`
- `WItem` subclass `nurgling.NWItem`
- `haven.Inventory` factory creates `nurgling.NInventory`

## `nurgling.NGItem` (model)

Nurgling adds:

- cached `name` for item display name
- parsed `quality` (via tooltip rawinfo parsing)
- parsed armor values (hard/soft)
- parsed “contents” list (`ui/tt/cont`) with quality per contained item
- `meterUpdated` timestamp tracking on `uimsg("tt")`/`uimsg("meter")`

It also intercepts:

- `wdgmsg("take")` and `wdgmsg("iact")` to record candidate items for UI/automation.

## `nurgling.NWItem` (view)

Key additions:

- Optimized overlay updates: only refresh overlay textures when `tick()` indicates changes.
- Inventory search highlighting: sets `((NGItem)item).isSearched` based on text and `NSearchable` infos.
- Auto-dropper integration: if enabled and item is marked searched, drop items automatically from main inventory.

Modifier click extensions:

- Alt+Shift+Click: `"transfer-same"` for same items sorted by quality (mouse button selects ascending/descending)
- Alt+Ctrl+Click: `"drop-same"` for same items sorted by quality

## `nurgling.NInventory` (container)

NInventory adds:

- optional slot numbering overlay (pre-rendered textures for performance)
- extra UI elements for search and right-side panels (expanded/compact)
- many **automation-friendly query helpers** that block on tasks:

  - `getFreeSpace()`, `getTotalSpace()`, `getItem(...)`, `getItems(...)`, `getWItems(...)`, etc.

### Task-based queries

Most query helpers use the same pattern:

1. construct a task (e.g., `new GetItems(this, alias)`)
2. schedule it: `NUtils.getUI().core.addTask(task)`
3. read the task’s result

This makes the call safe from a bot thread while still reading UI-thread state.

## Practical extension points

### Add a new inventory gesture

- Implement/extend in `NWItem.mousedown(...)` (gesture detection)
- Handle the new message in `NInventory.wdgmsg(...)` (container behavior)

### Add an inventory window control

Follow the titlebar injection pattern (see [Windows and menus](WindowsMenus.md)) so you do not need to fork the upstream `Window` implementation.

### Add new item-derived data

- If it is tooltip-derived, parse `rawinfo` in `NGItem.updateraw()`.
- If it requires per-tick logic, add in `NGItem.tick()`.

Be careful with thread safety: `tick()` runs on the UI thread.
