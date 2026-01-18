# Architecture overview

## Package layout

Nurgling2 is a custom Haven & Hearth client built on top of the upstream client codebase. The two dominant namespaces in the source tree are:

- `haven.*` – upstream client engine and gameplay UI.
- `nurgling.*` – the Nurgling2 “mod layer”: extended widgets, overlays, automation/bots, configuration, and domain-specific tooling.

In practice, **most extension work happens by subclassing and/or composing `haven` widgets**, then routing behavior through the Nurgling-side managers and config.

## Key entry points and “where to look”

### UI root and session state
- `nurgling.NUI extends haven.UI`
  - Owns Nurgling-side runtime state (`NCore`, `NSessInfo`, data tables).
  - Loads/saves selected UI settings (e.g., opacity).

### Main in-game UI
- `nurgling.NGameUI extends haven.GameUI`
  - Central hub for game widgets (inventory, map, windows) and Nurgling-side panels (bots, scenarios, recent actions).
  - Typically where you anchor new windows/panels so they are discoverable and persisted.

### World rendering and interaction
- `nurgling.NMapView extends haven.MapView`
  - The main 3D world view. Used by automation and navigation (pathfinding, click injection, etc.).
- `nurgling.widgets.NMapWnd extends haven.MapWnd`
  - Map window/minimap integration. Also intercepts clicks for some mod features (notably Forager path recording).

### Objects in the world (“gobs”)
- `haven.Gob` – upstream world object container.
- `nurgling.NGob` – a Nurgling companion attached to each Gob (via `gob.ngob`) that caches:
  - `name` (resource name like `gfx/terobjs/herbs/blueberry`)
  - hitboxes and overlays
  - various heuristics used by bots and HUD features

### Configuration persistence
- `nurgling.NConfig` – persistent configuration store; holds many per-user and per-character settings.
- Several feature-specific “prop” classes implement `JConf` serialization patterns (e.g., `nurgling.conf.NForagerProp`).

### Automation framework
- `nurgling.actions.Action` – the core “bot task” interface; each bot is an `Action`.
- `nurgling.actions.bots.registry.BotRegistry` – the registry mapping bot IDs to their `Action` classes and icons.
- `nurgling.widgets.NBotsMenu` – the in-game bots menu that launches actions.
- `nurgling.scenarios.*` – “Scenario” automation: ordered steps of bot actions.

## Extension strategy

When you add new functionality, you typically choose one (or more) of these integration points:

1. **New UI widget/window** – subclass `haven.Window` or `haven.Widget` in `nurgling.widgets.*`.
2. **New bot/action** – implement `nurgling.actions.Action` (or reuse existing building blocks), then register it in `BotRegistry`.
3. **Scenario step support** – if the bot needs scenario-time configuration, accept a `Map<String,Object>` constructor and add UI in `StepSettingsPanel`.
4. **Map/minimap interaction** – hook into `NMapWnd` / `NMapView` input handlers (only if needed).

