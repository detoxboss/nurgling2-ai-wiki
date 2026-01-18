# Extending: adding a new window or UI control

This page describes a practical approach for adding a new window (e.g., to control an automation script).

## 1) Create a window class

Create a class under `nurgling.widgets.*` (or `nurgling.widgets.bots.*` if bot-specific) extending `haven.Window`.

Common controls:
- `Label`, `Button` / `IButton`
- `TextEntry`
- `Dropbox` / `NDropbox`
- `CheckBox`

## 2) Wire it into `NGameUI`

You need an entry point so the user can open the window.

Common patterns in this codebase:
- expose it through the bots menu (if it’s tied to a bot)
- open it from a settings panel/tab
- add a dedicated HUD button

## 3) Persist settings

If your window has configurable state, persist it via:

- a feature-specific `*Prop` class implementing `JConf`
- stored inside `NConfig` under a dedicated `NConfig.Key`

Then:
- load current settings when the window opens
- save settings when the window closes or on every change

## 4) Keep UI and bot logic separated

Recommended:
- UI window collects user intent and settings.
- Bot logic lives in `nurgling.actions.*` and is invoked by the bots framework.

This separation makes it easier to run the same bot headlessly (e.g., through scenarios).

