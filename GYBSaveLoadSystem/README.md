# GYBSaveLoadSystem

A save and load plugin for Unreal Engine 5. Drop it into your project, mark your variables with `SaveGame`, and call `GYB_SaveGame`. That's the whole integration.

No boilerplate. No subclassing. No engine modification.

> **Supported engine versions:** UE 5.3, UE 5.4, UE 5.5  
> **Marketplace:** [Get it on FAB](https://www.fab.com/sellers/Get%20Your%20Basics)

---

## What it does

GYBSaveLoadSystem serializes any actor or GameInstance variable marked with the `SaveGame` flag and writes it to a save slot. On load, it restores every value and respawns any actors that were created at runtime.

You don't write serialization code. The plugin handles it.

**Supported types:**
- Primitives — `bool`, `int32`, `int64`, `float`, `double`, `byte`
- Strings — `FString`, `FName`, `FText`
- Math — `FVector`, `FRotator`, `FTransform`, `FLinearColor`, `FColor`
- Containers — `TArray`, `TMap`, `TSet` of any supported type
- Custom structs — any `USTRUCT` with `SaveGame` properties
- Actor references — resolved automatically on load

---

## Requirements

- Unreal Engine 5.3 or newer
- No third-party dependencies
- Works with Blueprint-only projects — no C++ required

---

## Installation

1. Copy the `GYBSaveLoadSystem` folder into your project's `Plugins/` directory.
2. Open your project. When UE5 asks to rebuild missing modules, click **Yes**.
3. Enable the plugin: **Edit → Plugins → GYBSaveLoadSystem → Enabled**.
4. Restart the editor.

That's it. The plugin is ready to use.

---

## Quick integration

**Step 1 — Add the component**

Select your actor in the editor. In the Details panel, click **Add Component** and search for `GYBSaveableComponent`. Add it.

**Step 2 — Mark your variables**

Open the actor's Blueprint. For each variable you want saved, enable the **SaveGame** flag in its details.

![SaveGame flag location](../shared/images/savegame-flag.png)

**Step 3 — Give the actor a unique ID**

Select the `GYBSaveableComponent` in the actor. Set the `UniqueID` field to something that identifies this actor — for example, `PlayerCharacter` or `MainDoor`.

> Every actor that uses the plugin needs a unique ID. If two actors share the same ID, the plugin auto-appends a suffix (`Enemy`, `Enemy_1`, `Enemy_2`), but it's better to set them explicitly.

**Step 4 — Save and load**

Call these nodes from any Blueprint:

```
GYB_SaveGame(SlotIndex, SlotName)  →  saves to the given slot
GYB_LoadGame(SlotIndex)            →  loads from the given slot
```

Both return an `EGYBSaveResult` you can branch on to handle errors.

---

## Blueprint API

| Node | Description |
|---|---|
| `GYB_SaveGame` | Saves all registered actors and the GameInstance to a slot |
| `GYB_LoadGame` | Loads a slot and restores all actor data |
| `GYB_DeleteSave` | Deletes a save slot |
| `GYB_GetAllSlots` | Returns info for all existing slots |
| `GYB_GetSlotInfo` | Returns info for a single slot |
| `GYB_DoesSaveExist` | Returns true if a slot has save data |

Full reference with parameters and examples: [Blueprint Node Reference](docs/blueprint-reference.md)

---

## Documentation

- [Quickstart — from zero to first save](quickstart.md)
- [Blueprint Node Reference](blueprint-reference.md)
- [Integration Guide](integration-guide.md)
- [FAQ and Troubleshooting](faq.md)

---

## Project Settings

The plugin adds a settings page under **Edit → Project Settings → GYBSaveLoadSystem**.

| Setting | Default | Description |
|---|---|---|
| Max Registered Objects | 500 | Maximum number of actors the plugin tracks at once |
| Max Save Slots | 10 | Maximum number of save slots available |

---

## Support

Found a bug or have a question? Join GetYourBasics Discord and ask!.

https://discord.gg/MaVR5MRe8

For general feedback or feature requests, leave a review on the FAB listing.
