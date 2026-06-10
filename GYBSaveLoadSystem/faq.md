# FAQ and Troubleshooting

Answers to the most common questions and problems. If your issue isn't here, open an issue on this repository or ask on the [GetYourBasics Discord](https://discord.gg/MaVR5MRe8).

---

## My actor isn't being saved

**Check these in order:**

1. **Does the actor have a `GYBSaveableComponent`?** Open the actor in the Blueprint editor and check the Components panel. If the component isn't there, the plugin doesn't know the actor exists.

2. **Is the UniqueID set?** Select the `GYBSaveableComponent` in the Components panel and check the Details panel. If UniqueID is empty, the actor can't be identified on load.

3. **Are the variables marked with SaveGame?** Select each variable in the My Blueprint panel and check the SaveGame checkbox in Details. This flag is what tells the plugin which variables to serialize.

4. **Is the actor registered before you call `GYB_SaveGame`?** The component registers the actor in `BeginPlay`. If the actor spawns after `GYB_SaveGame` is called, it won't be included in that save.

---

## Actor references are null after loading

This happens when the referenced actor isn't registered with the plugin.

**Requirements for actor references to work:**
- The referenced actor must have a `GYBSaveableComponent`
- The referenced actor must have a UniqueID set
- Both actors must be registered before the load completes

If the referenced actor meets these requirements and the reference is still null, check the Output Log — the plugin logs a warning with the UniqueID it couldn't resolve.

> See [Integration Guide — Actor references](integration-guide.md#actor-references) for how the two-phase loading system works.

---

## The plugin doesn't appear in Project Settings

This usually means the plugin isn't fully enabled.

**Steps to fix:**
1. Go to **Edit → Plugins** and search for `GYBSaveLoadSystem`.
2. Make sure the **Enabled** checkbox is checked.
3. Restart the editor.
4. If it still doesn't appear, close UE5, delete the `Binaries` and `Intermediate` folders from your project root, reopen the project, and let it rebuild.

---

## How do I increase the limit of registered objects?

Go to **Edit → Project Settings → GYBSaveLoadSystem** and increase `Max Registered Objects`. The default is 500.

This limit exists to prevent unbounded memory allocation in projects that spawn large numbers of actors. Increasing it has no performance cost — set it to whatever your game needs.

---

## My variables are restored but the actor looks wrong

The variables are correct but something visual isn't updating — a mesh position, a material, an animation state.

This is the expected behavior. The plugin restores variable values but doesn't re-run `BeginPlay` or any other initialization logic. Visual state driven by code needs to be updated manually after load.

**The fix:** implement `OnDataRestored` on the actor and update your visuals there. It fires automatically after data is applied.

```
[OnDataRestored]
      ↓
[your logic here — update mesh, material, animation, etc.]
```

> See [Integration Guide — OnDataRestored](integration-guide.md#ondatarestored) for examples.

---

## Does the level reload when I call GYB_LoadGame?

No. Data is restored directly onto the actors already in the world — no level reload, no loading screen. Runtime-spawned actors are respawned automatically before their data is applied.

This is intentional. A level reload would force a loading screen and break flows where you want to load mid-game without interrupting the experience.

---

## Can I use this plugin with C++?

The Blueprint API works from C++ as well — call the nodes through the `UGYBSaveSubsystem` directly. A dedicated C++ API (`UGYBSaveLibrary` static functions) is planned for v2.

---

## What UE5 versions are supported?

UE 5.3, 5.4, and 5.5.

---

## Can I add more save slots than the default 10?

Yes. Go to **Edit → Project Settings → GYBSaveLoadSystem** and increase `Max Save Slots`. There's no hard cap — set it to whatever your game needs.

---

## Does this work in multiplayer?

Not in v1. The current system is designed for single-player — one save file, one GameInstance, one set of registered actors. Per-player save support is planned for v2.

---

## What happens if two actors have the same UniqueID?

The plugin detects the collision and auto-generates suffixes — `Enemy`, `Enemy_1`, `Enemy_2` — up to 1000 iterations. It logs a warning in the Output Log when this happens.

This works, but it's not reliable if the spawn order of those actors can change between sessions. If the order changes, the suffixes change, and data gets applied to the wrong actor. Set explicit UniqueIDs on your actors to avoid this entirely.

---

## A variable type I'm using isn't being saved

The plugin supports a specific set of types. If a variable type isn't supported, the plugin logs a warning:

```
Property X of type Y is not supported
```

**Supported types:**
- Primitives — `bool`, `int32`, `int64`, `float`, `double`, `byte`
- Strings — `FString`, `FName`, `FText`
- Math — `FVector`, `FRotator`, `FTransform`, `FLinearColor`, `FColor`
- Containers — `TArray`, `TMap`, `TSet` of any supported type
- Custom structs — any `USTRUCT` with `SaveGame` properties (up to 2 levels deep)
- Actor references — actors with a `GYBSaveableComponent`

If your type isn't on this list, it won't be saved. Support for additional types is planned for future versions.

---

## How do I migrate saves between engine versions?

Save files are binary — they're tied to the plugin version that created them, not the engine version. If you update the plugin to a new version, existing saves remain compatible as long as you haven't removed or renamed any SaveGame variables. Removing a variable that was previously saved is safe — the plugin ignores data for variables that no longer exist.

---

## Something else is wrong — where do I look?

The Output Log is your first stop. The plugin logs warnings and errors with the `GYBSaveLoadSystem` prefix — filter by that to find relevant messages quickly.

If the log doesn't help, open an issue on this repository with:
- Your UE5 version
- What you expected to happen
- What actually happened
- The relevant Output Log entries
