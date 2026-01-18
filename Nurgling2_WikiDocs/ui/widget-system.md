# UI widget and window system

Nurgling2 uses Haven’s widget system. Most UI is built from these primitives:

- `haven.Widget` – base UI node.
- `haven.Window` – movable container widget.
- `haven.Button`, `haven.IButton`, `haven.Label`, `haven.TextEntry`, `haven.Dropbox`, `haven.Listbox` – standard controls.

Nurgling adds a number of custom widgets (e.g., `NDropbox`, `NScenarioButton`, etc.) but the lifecycle and composition model remains the same.

## Widget composition pattern

Widgets are composed by calling `add(childWidget, position)`.

Typical layout approach in this client:

- Build rows as lightweight container widgets (e.g., `new Widget(new Coord(w,h))`).
- Add controls into each row at explicit coordinates.
- Advance a `prev` pointer and place new rows relative to `prev.pos("bl")`.

Example pattern (conceptual):

```java
Widget prev = add(new Label("Title"), new Coord(0, 0));
prev = add(new TextEntry(UI.scale(200), ""), prev.pos("bl").add(UI.scale(0, 5)));
```

## Event handling

Two common ways UI reacts to user input:

1. **Override control callbacks**
   - `Button.click()`
   - `TextEntry.changed()`
   - `Dropbox.change(item)`

2. **Override widget message handling**
   - `wdgmsg(String msg, Object... args)`
   - Used for window close, drag/drop, and other low-level interactions.

## The “wdgmsg” channel

Haven’s UI uses a message bus pattern:

- Widgets send messages to parents (and ultimately the server) with `wdgmsg`.
- Bots simulate interaction by sending the same messages (e.g., map click injection).

When you implement a custom window, you often intercept:

- `msg == "close"` – to persist settings or shut down background tasks cleanly.

## Recommended Nurgling conventions

- Put “feature UI” in `nurgling.widgets.*`.
- If it’s a bot UI, place it under `nurgling.widgets.bots.*`.
- Persist settings via a `*Prop` class stored in `NConfig` (per character if appropriate).

