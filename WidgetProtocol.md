
# Widgets and the server protocol

## `@RName` factories

The server sends widget-creation commands that refer to a symbolic widget name. On the client side:

- Classes are annotated with `@RName("...")`
- A nested `public static class $_ implements Factory` provides `create(UI ui, Object[] args)`

Nurgling modifies several `haven` factories to instantiate Nurgling subclasses. This is how `NGameUI`, `NMapView`, `NInventory`, `NGItem`, and `NFlowerMenu` become the live objects without needing server changes.

## `wdgmsg` vs `uimsg`

- **Client → server**: `wdgmsg("...")` sends a command for this widget.
- **Server → client**: `uimsg("...")` delivers state updates.

A useful rule of thumb:

- if you are trying to *perform an action*, it will be a `wdgmsg` (e.g., `"click"`, `"itemact"`, `"take"`).
- if you are trying to *react to server state*, you implement/override `uimsg`.

## Examples in Nurgling

### Intercepting item actions

`nurgling.NGItem.wdgmsg(...)` intercepts `"take"` and `"iact"` to record the candidate item name into `NCharacterInfo`.

### Custom inventory gestures

`nurgling.NWItem.mousedown(...)` adds modifier-based actions:

- Alt+Shift+Click: `wdgmsg("transfer-same", item, ascending)`
- Alt+Ctrl+Click: `wdgmsg("drop-same", item, ascending)`

These messages are handled on the **inventory widget** side, not the server, so they are an example of a *client-local extension* pattern.

### World interaction messages

`nurgling.NUtils` is the main “message helper” for world interaction. Examples:

- click a gob: `map.wdgmsg("click", ..., button=1, ..., gobid, ...)`
- right-click activate: `map.wdgmsg("click", ..., button=3, flags=1, ...)`
- itemact on gob: `map.wdgmsg("itemact", ..., gobid, ...)`

If you are writing automation, using `NUtils` is typically preferable to constructing raw `wdgmsg` payloads yourself.

## Finding widgets at runtime

Useful patterns already implemented in `NGameUI`:

- `getWindow(String caption)` – returns the first matching `Window`.
- `getWindows(String caption)` – returns all matching windows.
- `getWindowWithButton(String cap, String button)` – locate a window by caption that contains a specific `Button` label.

When adding new UI, using a stable window caption (or some other stable widget ID) will make automation much more robust.
