# Scenario system

Scenarios are user-defined automation sequences: an ordered list of bot steps.

## Key classes

- `nurgling.scenarios.Scenario`
  - Holds scenario ID, name, and a list of `BotStep` entries.

- `nurgling.scenarios.BotStep`
  - Holds a bot ID plus a `Map<String,Object>` settings payload (serialized to JSON).

- `nurgling.actions.bots.ScenarioRunner`
  - Executes the scenario by instantiating each bot via `BotRegistry` and calling `run`.

## UI

- Settings panel: `nurgling.widgets.nsettings.ScenarioPanel`
  - Lets users create/edit scenarios.
- Step editor: `nurgling.widgets.StepSettingsPanel`
  - Provides per-step settings UIs for selected bots.

## Bots with scenario settings

In the current codebase, the step editor has explicit settings support for at least:

- `goto_area` – parameter: `areaId`
- `forager` – parameter: `presetName`
- `equipment_bot` – parameter: `presetId`
- `wait_bot` – parameter: `waitDurationMs`

To add settings support for a new bot, extend `StepSettingsPanel` with the required controls and write into `step.setSetting(key, value)`.

