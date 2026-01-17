
# Automation framework

## Two-layer model

Nurgling automation generally operates in two layers:

1. **Actions/Bots** (high-level intent)
   - implement `nurgling.actions.Action`
   - run in their own thread (frequently created by UI button clicks)

2. **Tasks** (UI-thread polling / waiting)
   - extend `nurgling.tasks.NTask`
   - scheduled into `NCore` via `NUI.core.addTask(task)`
   - `addTask(...)` blocks the caller thread until the task completes

## `nurgling.NCore`

NCore is attached to `ui.root` and ticked every frame.

Core responsibilities relevant to automation:

- owns a concurrent task queue
- each tick, calls `task.check()`; if `true`, notifies and removes the task
- manages auto-actions like AutoDrink and AutoSaveTableware (threaded)
- controls bot mode flags
- manages DB lifecycle and related sync services

### Task scheduling contract

`addTask(task)` behavior (simplified):

- if `task.check()` is already true, returns immediately
- otherwise, enqueue task, then `wait()` on the task object
- when UI tick loop later sees `task.check() == true`, it calls `notify()` on the task

This means:

- tasks must be written to be safe to call repeatedly from the UI tick
- tasks should generally be short and side-effect free unless explicitly intended

## `Action` interface

`nurgling.actions.Action` is intentionally minimal:

- `Results run(NGameUI gui) throws InterruptedException`

Most bots/actions do:

- query/locate UI state (inventory, windows, gobs)
- send actions to server via `wdgmsg` wrappers (`NUtils.*`)
- block on tasks while waiting for state transitions

## Scenarios

There is an additional orchestration layer in:

- `nurgling.scenarios.*`
- `nurgling.actions.bots.ScenarioRunner`

Scenarios allow chaining multiple bot steps with settings.

## Safety and robustness patterns

When adding new automation:

- use stable lookup methods (area IDs, window captions, resource names)
- always handle `InterruptedException`
- avoid long UI thread work; do heavy loops in bot thread and do small UI-state checks via tasks

## Recommended extension patterns

- New “unit of behavior”: implement `Action`
- New UI polling primitive: implement `NTask`
- New menu button: register in `BotRegistry` and provide icon assets
