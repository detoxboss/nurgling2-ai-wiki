
# Architecture

## Top-level runtime graph

At runtime, the client has a few “spines” that everything else hangs off:

1. **Rendering / UI loop**: `haven.GLPanel` owns the render loop and creates the global UI instance.
2. **UI root & widget tree**: `haven.UI` (here: `nurgling.NUI`) owns the widget tree (`ui.root`) and dispatches input/events.
3. **In-game UI**: the server constructs a widget named `gameui`, which Nurgling maps to `nurgling.NGameUI`.
4. **World view**: inside GameUI the `mapview` widget is created; Nurgling maps it to `nurgling.NMapView`.
5. **Automation core**: `nurgling.NCore` is a widget bound into `ui.root` and provides a task queue, periodic services, and DB lifecycle.

## Startup sequence (conceptual)

1. `haven.GLPanel.newui(...)` instantiates `nurgling.NUI`.
2. `NUI` constructor attaches `NCore` to `ui.root` and binds it.
3. Server sends widget creation messages. When it creates `gameui`, the `haven.GameUI.$_` factory returns `new nurgling.NGameUI(...)`.
4. GameUI builds standard sub-widgets (chat, menus, portrait, buff list, etc.). Nurgling adds many draggable containers (`NDraggableWidget`).
5. During `GameUI.attached()`, heavy widgets are created once (`areas`, `cookbook`, `encyclopedia`, `blueprint`). Nurgling overrides `NGameUI.attached()` as well to apply profile-aware data and settings.

## Data and gameplay subsystems

### World model

- `haven.Glob` – world state container.
- `haven.OCache` – object cache for `Gob` instances.
- `haven.Gob` – world object.
- `haven.MCache` – map tile cache.

Nurgling extends world interaction primarily in `nurgling.NMapView` via:

- overlay toggles (`toggleol(...)`)
- selection modes (area/gob selection, measure tool)
- route and portal visuals
- chunk navigation manager integration

### Inventory / items

- `haven.Inventory` → **`nurgling.NInventory`**
- `haven.GItem` → **`nurgling.NGItem`**
- `haven.WItem` → **`nurgling.NWItem`**

Nurgling adds:

- slot numbering overlay
- search/filtering integration
- additional item interaction verbs (`transfer-same`, `drop-same`)
- quality parsing and content parsing from tooltip payload

### Automation

- `nurgling.actions.Action` – interface for a unit of automation with `Results run(NGameUI gui)`.
- `nurgling.actions.bots.*` – the bulk of automation content.
- `nurgling.tasks.NTask` – polling tasks scheduled into `NCore`.
- `nurgling.NCore` – owns the tasks queue and checks tasks on every tick.

### Configuration / profiles

- `nurgling.NConfig` – global and per-profile keys.
- `nurgling.profiles.ConfigFactory` – resolves a profile-aware `NConfig` instance once the character `genus` is known.

### Database-backed services

When DB is enabled (`NConfig.Key.ndbenable`), `NCore` instantiates `nurgling.db.DatabaseManager` and starts sync services (areas/routes and other data).

## Threading model (practical)

The Haven client is not generally thread-safe across all UI/world state.

- Most UI and world state should be accessed from the **UI tick thread**.
- Many bots are run in their own Java threads (e.g., `new Thread(() -> bot.run(gui))`).
- To safely “wait for” UI state, bots schedule `NTask` instances via `NUI.core.addTask(task)`, which blocks the bot thread until the task completes.

This is the critical mental model: **bots do their work by repeatedly scheduling tasks that poll state on the UI thread**.
