
# How to add a bot to the Bots Menu

## What the Bots Menu is

`nurgling.widgets.NBotsMenu` is an icon-based launcher. It groups bots by `BotDescriptor.BotType`.

The menu sources items from:

- `nurgling.actions.bots.registry.BotRegistry.allowedInBotMenu()`

Each item is:

- instantiated from its `Action` class
- executed by clicking its icon

## Step-by-step

### 1) Create an Action

Implement `nurgling.actions.Action`:

```java
package nurgling.actions.bots;

import nurgling.actions.*;
import nurgling.NGameUI;

public class MyExampleBot implements Action {
    @Override
    public Results run(NGameUI gui) throws InterruptedException {
        // Do work...
        return Results.SUCCESS();
    }
}
```

If you want settings:

```java
public MyExampleBot(Map<String,Object> settings) {
   // read settings
}
```

### 2) Add a BotDescriptor entry

Edit `nurgling/actions/bots/registry/BotRegistry.java` and add:

```java
bots.add(new BotDescriptor(
    "my_example_bot",
    BotDescriptor.BotType.UTILS,
    "My Example Bot",
    "Does a thing",
    false,  // allowedAsStepInScenario
    true,   // allowedAsItemInBotMenu
    MyExampleBot.class,
    "my_example_icon_dir",
    false   // disStacks
));
```

Notes:

- `id` should be unique.
- `iconPath` is a directory name under `resources/nurgling/bots/icons/`.

### 3) Add icon assets

Create these files:

- `resources/nurgling/bots/icons/my_example_icon_dir/u` (up)
- `resources/nurgling/bots/icons/my_example_icon_dir/d` (down)
- `resources/nurgling/bots/icons/my_example_icon_dir/h` (hover)

Nurgling commonly stores icons as images loaded by `Resource.loadsimg(...)`.

### 4) Optional: disStacks

Some bots set `disStacks` to disable stacking behavior or certain menu behaviors. Use existing bots as reference.

## Debugging tips

- If your icon does not show, verify resources are on the classpath and the path is correct.
- If clicking does nothing, catch and log exceptions in your Action `run`.
- If your bot blocks forever, verify any `addTask(...)` calls eventually resolve and that tasks are actually being checked in `NCore.tick()`.
