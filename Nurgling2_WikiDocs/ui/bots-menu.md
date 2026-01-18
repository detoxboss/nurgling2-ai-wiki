# Bots menu and bot UIs

## Bots menu

The bots menu is implemented by `nurgling.widgets.NBotsMenu`.

### What it does
- Provides a tiled icon menu grouped by bot category.
- Launches bots/actions on separate threads.
- Registers launched actions in the “recent actions” panel when available.

### How bots are launched
The menu wraps a bot `Action` in a `Thread` and runs `action.run(gui)`. If an action has supporting/background actions via `action.getSupp()`, those are started too.

Implication for bot authors:
- `Action.run(NGameUI gui)` must be thread-safe with respect to UI reads.
- Your bot should handle `InterruptedException` cleanly as “stop requested”.

## Bot registry

Bots appear in the menu because they are registered in `nurgling.actions.bots.registry.BotRegistry` with:
- an ID
- category (`BotType`)
- display name/description
- icon path
- the implementing `Action` class

Scenarios can also instantiate bots through the same registry (see `scenarios/scenarios.md`).

