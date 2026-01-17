
# Nurgling2 (Nurgling client) – Developer Wiki

This documentation set is intended to give you an engineering-level mental model of **how the Nurgling2 custom Haven & Hearth client is organized**, how it extends the upstream **`haven`** codebase, and where to add or modify features—especially UI controls that trigger automation.

> Scope note: this is a source-oriented “system map”. It is not a user manual. For user-facing feature docs that ship with the project, see **[User docs](user_docs/welcome.md)**.

## Repository layout (high level)

- `src/haven/...`  
  Upstream (or upstream-derived) Haven client code. In this project, several factories in `haven` are deliberately modified to instantiate Nurgling extensions.

- `src/nurgling/...`  
  The Nurgling feature layer: UI overlays, custom widgets/windows, automation actions/bots, area/route systems, configuration, services, and DB integration.

- `src/nurgling/actions/...`  
  Action framework (automation entrypoints). Most bots implement `nurgling.actions.Action`.

- `src/nurgling/tasks/...`  
  “UI-thread polling” tasks that bots/actions enqueue into `NCore`.

## The most important design pattern: **factory overrides in `haven`**

The Haven client uses the `@RName("...")` registry to build widgets based on messages from the server. Nurgling modifies **the `haven` factories** so that core widgets are constructed as Nurgling subclasses.

Key overrides you will encounter immediately:

- `gameui` → `nurgling.NGameUI` (primary in-game UI)
- `mapview` → `nurgling.NMapView` (world view and interaction)
- `inv` → `nurgling.NInventory` (inventory widget)
- `item` → `nurgling.NGItem` (item data/model)
- `sm` → `nurgling.NFlowerMenu` (flower menu)

Full `@RName` mapping table extracted from the codebase:

| RName | Factory creates | Defined in |
|---|---|---|
| `acnt` | `AlignPanel` | `haven/Widget.java` |
| `av` | `` | `haven/Avaview.java` |
| `battr` | `BAttrWnd` | `haven/BAttrWnd.java` |
| `btn` | `Button` | `haven/Button.java` |
| `buddy` | `NBuddyWnd` | `haven/BuddyWnd.java` |
| `buff` | `Buff` | `haven/Buff.java` |
| `ccnt` | `` | `haven/Widget.java` |
| `charlist` | `NCharlist` | `haven/Charlist.java` |
| `chat` | `Chatwindow` | `haven/Chatwindow.java` |
| `chk` | `` | `haven/CheckBox.java` |
| `chr` | `CharWnd` | `haven/CharWnd.java` |
| `cnt` | `Widget` | `haven/Widget.java` |
| `credo` | `TabProxy` | `haven/SkillWnd.java` |
| `dynres` | `DynresWindow` | `haven/DynresWindow.java` |
| `epry` | `NEquipory` | `haven/Equipory.java` |
| `expls` | `TabProxy` | `haven/SkillWnd.java` |
| `fcnt` | `` | `haven/Widget.java` |
| `fmg` | `FightWnd` | `haven/FightWnd.java` |
| `frv` | `Fightview` | `haven/Fightview.java` |
| `fsess` | `NFightsess` | `haven/Fightsess.java` |
| `gameui` | `NGameUI` | `haven/GameUI.java` |
| `give` | `GiveButton` | `haven/GiveButton.java` |
| `grp` | `GroupSelector` | `haven/BuddyWnd.java` |
| `hr` | `HRuler` | `haven/HRuler.java` |
| `ibtn` | `IButton` | `haven/IButton.java` |
| `ichk` | `` | `haven/ICheckBox.java` |
| `im` | `` | `haven/IMeter.java` |
| `img` | `` | `haven/Img.java` |
| `inv` | `NInventory` | `haven/Inventory.java` |
| `isbox` | `NISBox` | `haven/ISBox.java` |
| `item` | `NGItem` | `haven/GItem.java` |
| `lbl` | `Label` | `haven/Label.java` |
| `linpack` | `` | `haven/PackCont.java` |
| `log` | `Textlog` | `haven/Textlog.java` |
| `ltbtn` | `` | `haven/Button.java` |
| `make` | `NMakewindow` | `haven/Makewindow.java` |
| `mapview` | `NMapView` | `haven/MapView.java` |
| `mchat` | `` | `haven/ChatUI.java` |
| `pchat` | `` | `haven/ChatUI.java` |
| `pmchat` | `` | `haven/ChatUI.java` |
| `prog` | `` | `haven/Progress.java` |
| `pv` | `Partyview` | `haven/Partyview.java` |
| `quest` | `DefaultBox` | `haven/QuestWnd.java` |
| `quests` | `QuestWnd` | `haven/QuestWnd.java` |
| `sattr` | `SAttrWnd` | `haven/SAttrWnd.java` |
| `schan` | `` | `haven/ChatUI.java` |
| `scm` | `MenuGrid` | `haven/MenuGrid.java` |
| `scr` | `Scrollport` | `haven/Scrollport.java` |
| `sess` | `SessWidget` | `haven/SessWidget.java` |
| `skill` | `TabProxy` | `haven/SkillWnd.java` |
| `sm` | `NFlowerMenu` | `haven/FlowerMenu.java` |
| `speedget` | `Speedget` | `haven/Speedget.java` |
| `text` | `TextEntry` | `haven/TextEntry.java` |
| `vm` | `VMeter` | `haven/VMeter.java` |
| `wnd` | `Window` | `haven/Window.java` |
| `wound` | `WoundBox` | `haven/WoundWnd.java` |
| `wounds` | `WoundWnd` | `haven/WoundWnd.java` |

## Where to start reading (recommended)

If you only read five files first, read these:

1. `haven/GLPanel.java` – creates `NUI` (the global UI instance) and swaps UIs on reconnect.
2. `nurgling/NUI.java` – extends `UI`; adds `NCore`, session info, inspect mode, and various global UI behaviors.
3. `haven/GameUI.java` (factory + base constructor) + `nurgling/NGameUI.java` – the core in-game UI.
4. `haven/MapView.java` + `nurgling/NMapView.java` – world rendering, click handling, overlays, selection modes.
5. `nurgling/NCore.java` – automation “kernel”: tasks, periodic services, DB lifecycle, mapv4 client.

## Documentation map

- [Architecture](Architecture.md)
- [UI framework](UIFramework.md)
- [Widgets and server protocol](WidgetProtocol.md)
- [Windows, menus, and draggable panels](WindowsMenus.md)
- [Inventory and item model](Inventory.md)
- [World interaction](WorldInteraction.md)
- [Automation framework](Automation.md)
- [How to add UI for automation](AddingUI.md)
- [How to add a bot to the Bots Menu](BotRegistry.md)
