# Extending: adding a new bot

## 1) Implement an `Action`

Create a class under `nurgling.actions.bots.*` implementing `nurgling.actions.Action`:

```java
public class MyBot implements Action {
  @Override
  public Results run(NGameUI gui) throws InterruptedException {
    // ... bot logic ...
    return Results.SUCCESS();
  }
}
```

Important:
- Use `InterruptedException` as your stop signal.
- Avoid blocking UI thread; bots run on their own thread already.

## 2) Optional: support scenario configuration

Add a constructor:

```java
public MyBot(Map<String,Object> settings) {
  // read settings
}
```

Then add editor UI to `StepSettingsPanel`.

## 3) Register in `BotRegistry`

Add a `BotDescriptor` in `nurgling.actions.bots.registry.BotRegistry`:
- choose an `id`
- set category (`BotType`)
- set icon folder under `nurgling/bots/icons/<iconPath>/[u|d|h]`

## 4) Add a bot UI window (optional)

If your bot needs a UI:
- create a `Window` class under `nurgling.widgets.bots.*`
- have the bot open the window and read settings from it (or from a `*Prop` persisted config)

