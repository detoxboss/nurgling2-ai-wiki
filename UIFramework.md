
# UI Framework

## Widget tree fundamentals

Haven’s UI is a tree of `haven.Widget` instances rooted at `ui.root`. Each widget:

- has coordinates (`c`) and size (`sz`)
- can receive draw calls (`draw(GOut g)`)
- receives input events (`mousedown`, `mouseup`, `mousemove`, `keydown`, …)
- communicates with the server via `wdgmsg(String msg, Object... args)`
- receives server-originating events via `uimsg(String msg, Object... args)`

Nurgling follows the same model and implements features mostly by:

- subclassing widgets (`NMapView`, `NInventory`, `NFlowerMenu`, etc.)
- wrapping widgets in draggable containers (`NDraggableWidget`)
- hooking widget messages (`NGItem.wdgmsg`, `NWItem.mousedown`, etc.)

## Global UI extension: `nurgling.NUI`

`NUI` extends `haven.UI` and is instantiated in `haven.GLPanel.newui(...)`.

Key responsibilities:

- Adds `NCore` to the root widget and binds it.
- Tracks session info (`NSessInfo`) and persists verification/subscription flags.
- Implements an inspect mode driven by modifier keys (Shift / config-dependent).
- Provides adjustable UI opacity and window background mode.

## In-game UI extension: `nurgling.NGameUI`

`NGameUI` is constructed via the `gameui` `@RName` factory. It is the central place to:

- access primary widgets (map, main inventory, equipment, menus)
- create/attach Nurgling widgets that need to exist once per session
- expose helper methods like `getWindow(String caption)`

Important patterns in `NGameUI`:

- “heavy widgets” are initialized in `attached()` to avoid repeated init during construction/reconnect.
- UI composition commonly uses **`NDraggableWidget`** for panels that can be repositioned.

## Draggable panels: `nurgling.widgets.NDraggableWidget`

Nurgling implements a standardized draggable container:

- Drawn and interactive only when `ui.core.mode == NCore.Mode.DRAG`.
- Stores position/locked/visibility/flip state in `NDragProp` keyed by panel name.
- Optionally supports flipping content layout.

If you add a new persistent UI panel, wrapping it in `NDraggableWidget` is usually the correct pattern.

## Key inputs and bindings

Nurgling uses two mechanisms:

- Java AWT key events (`KeyEvent`) in `NUI.keydown(...)`.
- Haven’s `KeyBinding`/`KeyMatch` in widgets like `NMapView`.

When adding new hotkeys, prefer `KeyBinding.get("id", KeyMatch...)` where possible to integrate with the client’s keybinding system.
