# Forager bot: recording paths and harvesting

This page documents **how the Forager bot works** and how a user records and executes a forage route.

## Components

- UI window: `nurgling.widgets.bots.Forager`
- Bot logic: `nurgling.actions.bots.Forager`
- Preset storage: `nurgling.conf.NForagerProp` (per user + character)
- Path format: `nurgling.routes.ForagerPath` with `ForagerWaypoint` and `ForagerSection`
- Minimap hook: `nurgling.widgets.NMapWnd` (captures minimap clicks while recording)

## Conceptual model

A Forager “route” in Nurgling2 is:
- A list of **minimap waypoints** (segment id + tile coordinate) saved to JSON.
- A generated list of **sections** (~50 world units each) derived from those waypoints.
- A list of **actions** (pattern → interaction) executed while walking those sections.

Forager is intentionally “corridor-based”: it does not need perfect waypoints, because it scans around each section center and opportunistically harvests targets.

## Recording a foraging path

### Where paths are stored
Paths are saved as JSON in a directory relative to the client cache:

- `.../forager_paths/<pathName>.json`

### Recording workflow (user steps)
1. Start the Forager UI from the bots menu (**Resources → Forager**).
2. Select or create a **Preset**.
3. Select or create a **Path**.
4. Toggle **Record** on.
5. Left-click the minimap to add waypoints.
   - Do not hold Ctrl/Shift/Alt; plain clicks are required.
6. Toggle **Record** off to stop.
   - The path JSON is written to `forager_paths` and the preset is updated to reference the file.

### Critical limitation: waypoints are segment-scoped
`ForagerWaypoint.toWorldCoord(...)` only converts a waypoint if its `seg` matches your current minimap segment (`sessloc.seg.id`).

Practical consequences:
- A single Forager path generally **cannot cross minimap “segments”** (e.g., transitions that produce a different `seg.id`).
- If you record a path in one segment and attempt to execute it while in another, Forager will fail to resolve the section coordinates.

If you need multi-segment travel, you typically split your run into multiple Forager paths and orchestrate them with other navigation steps.

### Path segmentation
A recorded path is converted into “sections” for execution:

- Each segment between waypoints is subdivided into sections up to ~50 world units long.
- Each section has a start, end, and a derived center.
- The bot uses these sections to decide where to walk and where to scan for harvestable objects.

## Configuring harvest actions

Forager uses an ordered list of `ForagerAction` entries.

### Object pattern
The “Object Pattern” is a substring matched against the gob’s **resource name** (e.g., `gfx/terobjs/herbs/blueberry`).

In practice:
- Use short, distinctive substrings like `blueberry`, `chantrelle`, `rustroot`, `flotsam`.
- If you find a pattern is matching too much, refine it by using a longer substring (e.g., `gfx/terobjs/herbs/blueberry` instead of `blueberry`).

### Action types
- `PICK`
  - Runs the flower menu option `Pick`.
- `FLOWER_ACTION`
  - Runs a user-specified flower menu option string.
  - You must enter the menu label exactly (as it appears in the flower menu), because selection is string-based.
  - Examples you might use depending on the object: `Harvest`, `Take`, `Collect`, `Chip`, `Chop`, `Gather`.
- `CHAT_NOTIFY`
  - Not a harvest action; emits a notification when the target pattern is seen.

### Ordering and duplicates
Actions are executed in list order. In particular:
- If you configure multiple actions that can target the same gob, earlier actions may change availability for later actions.
- Forager maintains a “processed gob” set to avoid repeated interaction with the same object id.

## Execution algorithm (what happens when you run it)

At runtime, Forager:

1. Validates that the preset has a loaded path with generated sections.
2. Walks to the first section, then iterates sections sequentially.
3. At each section:
   - Moves to the section end (or to a nearby target gob close to the end).
   - For each configured action, scans for matching gobs within a radius around the section center.
   - Visits each matching gob and executes the configured action.
   - Marks gobs as processed to avoid repeated interactions.

Implementation details worth knowing:
- Scanning is performed using `Finder.findGobs(...)` against a square search region around the section center.
- Movement is done through `nurgling.actions.PathFinder`.
- Interactions are executed through `nurgling.actions.SelectFlowerAction` (flower-menu driven).

## Safety / stop conditions

Forager can stop (or take a safety action) when:
- Other players (“borkas”) are detected (alarm state)
- Dangerous animals are within configured radius
- Inventory is near full (free slots ≤ 4) or hands are occupied

Configurable responses include:
- do nothing
- logout
- travel hearth

Additional flags:
- `ignoreBats` avoids treating bats as dangerous animals.
- `waterMode` changes movement offset logic intended for use from water/boat contexts.

## Scenario integration (Go To Area → Forager)

Forager supports scenario use via a `Map<String,Object>` constructor parameter:
- `presetName` – which preset to run.

### Recommended scenario pattern
If you want a repeatable “foraging run” that starts from a known location:

1. Create an `NArea` at the start of your forage loop (e.g., `ForageStart`).
2. In the Scenario editor, add steps:
   - **Go To Area** → `ForageStart`
   - **Forager** → select your Forager preset

Notes:
- `GotoArea` uses **ChunkNav** (`nurgling.navigation.ChunkNavManager`). It is a different system from the legacy “routes” system documented in `src/nurgling/docs/routes/routes.md`.
- Forager still requires that the recorded forage path waypoints belong to the current minimap segment. The most reliable approach is to place `ForageStart` inside the same segment where you recorded the path.

