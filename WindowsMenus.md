
# Windows, menus, and draggable panels

## Window widgets

Most “windows” in Haven are instances of `haven.Window` with a decoration (`Window.Deco`) that draws the frame and caption controls.

Nurgling frequently uses three patterns:

1. **Standard window**: `add(new Window(...))` (often hidden by default)
2. **Draggable panel**: wrap a widget or window in `NDraggableWidget`
3. **Titlebar button injection**: replace a window’s decoration to add custom buttons

## Heavy widgets initialization

`haven.GameUI` includes an `initHeavyWidgets()` method that creates some heavier Nurgling windows and hides them by default.

Nurgling also overrides/extends this pattern in `nurgling.NGameUI.attached()`.

If you are adding a new major window that should exist once per session, initializing it in `attached()` (or an `initHeavyWidgets()` style method) is usually correct.

## Flower menu

The “flower menu” is a context menu shown on right-click (and certain other interactions).

- base type: `haven.FlowerMenu`
- Nurgling type: `nurgling.NFlowerMenu`

Notable Nurgling behaviors:

- Auto-selection of petals based on config (`autoFlower`) and “auto petal” lists.
- Optional injection of extra actions like “Save Tree Location” / “Save Bush Location” based on `NCore.LastActions.gob`.

## The bots menu

`nurgling.widgets.NBotsMenu` is a custom icon menu composed of grouped layouts.

Key concepts:

- Bot descriptors are sourced from `BotRegistry.allowedInBotMenu()`.
- Each bot has an icon directory `resources/nurgling/bots/icons/<bot.iconPath>/{u,d,h}`.
- Clicking an icon instantiates and runs its `Action`.

See [How to add a bot](BotRegistry.md).

## Custom titlebar buttons (example: Inventory sort)

`nurgling.NGameUI` includes a safe titlebar-button injection pattern:

- locate the Inventory window (by caption, or by walking parent chain)
- replace its decoration via `wnd.chdeco(new InvSortDeco(...))`
- the new `InvSortDeco` extends `Window.DefaultDeco` and adds an `IButton`

This is a strong template when you want to add per-window controls without forking the base window implementation.

## Drag mode vs interaction mode

Nurgling has a global “drag mode” gate in `NCore` (`NCore.Mode.DRAG` vs `IDLE`).

- When in DRAG mode, draggable panels show their frames and can be repositioned.
- Some widgets (like `NMenuGridWdg`) forward clicks differently depending on the mode.

If your new panel behaves oddly with clicks, verify how it should behave when `ui.core.mode == DRAG`.
