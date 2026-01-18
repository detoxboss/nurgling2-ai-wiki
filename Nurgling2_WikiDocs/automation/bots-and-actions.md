# Bots and Actions framework

## The `Action` interface

Bots in Nurgling2 are implemented as `nurgling.actions.Action`.

Key ideas:
- A bot is a long-running (or short-running) task that executes in its own thread.
- Bots are expected to be interruptible; interruption is represented as `InterruptedException`.

Many bots are composition-heavy and use:
- `nurgling.actions.PathFinder` – movement/pathing
- `nurgling.tools.Finder` – scanning for gobs/items
- `nurgling.actions.SelectFlowerAction` – selecting a flower menu action on a gob
- various `nurgling.tasks.*` – waiting for UI state transitions (e.g., flower menu closed)

## Bot registration

Bots are registered in `nurgling.actions.bots.registry.BotRegistry` and become available via:
- `nurgling.widgets.NBotsMenu` (interactive use)
- `nurgling.scenarios` (scenario steps)

## Scenario instantiation contract

If you want your bot to be usable in scenarios with configuration:

- Provide a constructor `public MyBot(Map<String, Object> settings)`.
- Read scenario parameters from that map.

Example (conceptual):

```java
public MyBot(Map<String,Object> settings) {
    this.areaId = (Integer) settings.get("areaId");
}
```

Then you add an editor for these settings in `nurgling.widgets.StepSettingsPanel`.

