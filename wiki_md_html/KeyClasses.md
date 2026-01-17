
# Key classes by responsibility

This page is a fast index of “where does X live?”

## Core runtime

- `haven.GLPanel` – render loop; instantiates `nurgling.NUI`
- `nurgling.NUI` – global UI behaviors and session info
- `nurgling.NCore` – automation kernel; task scheduling; DB lifecycle

## In-game UI

- `nurgling.NGameUI` – main in-game UI; heavy widget init; window find helpers
- `nurgling.widgets.NDraggableWidget` – persistent draggable panels
- `nurgling.widgets.NBotsMenu` – bots launcher UI
- `nurgling.widgets.NMenuGridWdg` – menugrid wrapper + toggle buff tracking

## World / map

- `nurgling.NMapView` – world view; overlays; selection modes; navigation manager
- `nurgling.navigation.*` – chunk navigation and pathing abstractions
- `nurgling.areas.*` – area definitions, labels, and area persistence

## Inventory / items

- `nurgling.NInventory` – extended inventory UI; query helpers
- `nurgling.NGItem` – item model; tooltip parsing; cached quality/content
- `nurgling.NWItem` – item widget; search, auto-drop, modifier gestures

## Automation

- `nurgling.actions.Action` – bot interface
- `nurgling.actions.bots.registry.BotRegistry` – menu bot registration
- `nurgling.tasks.*` – blocking wait helpers for bots
