
# How to add UI for automation

This page focuses on practical patterns already in the codebase.

## Option A: Add a new standalone window

1. Create a new widget class.

A minimal example using `haven.Window`:

```java
package nurgling.widgets;

import haven.*;

public class MyAutomationWindow extends Window {
    public MyAutomationWindow() {
        super(UI.scale(320, 200), "My Automation", "My Automation");

        // Example controls
        add(new Label("Automation Controls"), UI.scale(10), UI.scale(10));

        add(new Button(UI.scale(120), "Run") {
            @Override
            public void click() {
                // Kick off a bot thread
                new Thread(() -> {
                    try {
                        new mypackage.MyBot().run(nurgling.NUtils.getGameUI());
                    } catch (InterruptedException e) {
                        // Respect interruption
                    }
                }).start();
            }
        }, UI.scale(10), UI.scale(40));

        pack();
    }
}
```

2. Attach the window somewhere stable.

Common patterns:

- In `NGameUI.attached()` / heavy widget init: create it once and hide by default.
- Wrap it in `NDraggableWidget` to support drag-mode placement:

```java
add(new NDraggableWidget(new MyAutomationWindow(), "MyAutomation", UI.scale(340, 240)));
```

3. Add a way to open/toggle it.

- Add a button to an existing panel
- Add a menu entry
- Bind a hotkey

## Option B: Add a button into an existing window titlebar

Use the pattern in `NGameUI` that injects an inventory sort button:

- find the target window
- subclass `Window.DefaultDeco`
- call `window.chdeco(new YourDeco(window, window.large))`

This approach is usually safer than editing upstream windows.

## Option C: Add a button/icon to the Bots menu

If your automation fits the “bot” model, the Bots menu is the quickest integration.

See [How to add a bot](BotRegistry.md).

## UI-to-bot communication patterns

### Pattern 1: Direct thread start

Simple but adequate for many bots:

- button click starts `new Thread(() -> action.run(gui))`

### Pattern 2: Settings-driven bot instantiation

If you want settings, follow the `BotDescriptor.instantiate(Map<String,Object>)` convention:

- implement a `public YourBot(Map<String,Object> settings)` constructor
- store settings
- use them in `run(...)`

### Pattern 3: Use scenarios

If your bot should be composable, register it as a scenario step (see `BotDescriptor.allowedAsStepInScenario`).

## Don’t fight the threading model

- Keep UI reads/waits in tasks (`NTask`) scheduled through `NCore.addTask`.
- Avoid doing long computations in `task.check()`.
- Prefer existing helpers in `NInventory` and `NUtils`.
