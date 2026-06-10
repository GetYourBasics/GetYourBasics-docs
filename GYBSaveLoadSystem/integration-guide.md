# Integration Guide

This guide covers everything beyond the basic quickstart. If you haven't done the quickstart yet, [start there](quickstart.md) first.

---

## Saving the GameInstance

The GameInstance is saved automatically — you don't need to add a `GYBSaveableComponent` to it. Just mark the variables you want saved with the `SaveGame` flag and the plugin handles the rest.

**Example**

Your GameInstance has a `TotalPlayTime` float and a `PlayerLevel` integer. Mark both with SaveGame. Every time you call `GYB_SaveGame`, those values are included automatically.

> The GameInstance is a single object — there's no UniqueID needed. The plugin always knows where to find it.

---

## Runtime-spawned actors

Level-placed actors are straightforward — they exist when the level loads, so the plugin finds them automatically. Runtime-spawned actors (enemies, pickups, projectiles spawned during gameplay) need a bit more consideration.

The plugin detects whether an actor was spawned at runtime and saves its class path and transform alongside its data. On load, it respawns the actor first, then restores its data.

**This works automatically as long as:**
- The actor has a `GYBSaveableComponent`
- The actor has a unique `UniqueID` set on the component

**What you need to do**

Nothing extra. Spawn the actor normally, make sure it has the component and a UniqueID, and the plugin takes care of the rest.

> **Warning:** If you spawn multiple actors of the same class with the same UniqueID, the plugin auto-generates suffixes — `Enemy`, `Enemy_1`, `Enemy_2`. This works, but it means the IDs are assigned by order of registration, which can lead to mismatches if the spawn order changes between saves. Set explicit UniqueIDs on runtime actors when possible.

---

## Actor references

You can save a reference from one actor to another. Mark the variable with `SaveGame` as usual — the plugin serializes it as the target actor's UniqueID and resolves it back to the actual actor on load.

**Example**

A `TriggerZone` actor has a variable `LinkedDoor` of type `ADoor*`, marked with SaveGame. When saved, the plugin stores the UniqueID of the door. On load, it finds the door by that ID and restores the reference.

**Requirements**

- The referenced actor must have a `GYBSaveableComponent`
- The referenced actor must have a UniqueID set

If the referenced actor has no `GYBSaveableComponent`, the plugin logs a warning and sets the reference to `nullptr` on load.

> **Two-phase loading:** All runtime actors are spawned first, then all data is restored. This guarantees that actor references can always be resolved — the target actor always exists by the time the plugin tries to find it.

---

## OnDataRestored

`OnDataRestored` fires on every registered actor and the GameInstance right after their data has been applied. It's the right place for any logic that depends on restored values.

**When you need it**

Any time an actor has state that isn't a SaveGame variable but is driven by one. The classic example is a mesh position driven by a boolean.

**How to implement it**

1. Open the actor's Blueprint.
2. In the Event Graph, right-click and search for `OnDataRestored`.
3. Add the event and implement your logic there.

**Example — Door**

```
[OnDataRestored]
      ↓
[Branch] → bIsOpen?
   True  → Set door mesh to open position
   False → Set door mesh to closed position
```

**Example — Character**

```
[OnDataRestored]
      ↓
[Set Health Bar percent] → CurrentHealth / MaxHealth
      ↓
[Update character material] → based on restored TeamIndex
```

> Don't use `BeginPlay` for logic that depends on saved variables. `BeginPlay` runs before data is restored — at that point the variables still have their default values.

---

## Building a save slot UI

The plugin gives you everything you need to build a save slot selection screen. Here's the recommended approach.

**Populating the slot list**

```
[Event Construct]
      ↓
[GYB_GetAllSlots] → Slots
      ↓
[For Each Loop] → SlotInfo
      ↓
[Branch] → bIsEmpty?
   False → create slot widget, pass SlotInfo
   True  → create empty slot widget with "New Save" label
```

**Displaying slot info**

Each `FGYBSlotInfo` gives you everything you need for a slot entry:

- `SlotName` — the name set when saving
- `SaveDateTime` — format it however you like with UE5's date formatting nodes
- `LevelName` — the level where the save was made
- `PlayTimeSeconds` — convert to hours/minutes for display

**Saving from the UI**

```
[Button: Save]
      ↓
[GYB_SaveGame] → SlotIndex: SelectedIndex, SlotName: InputText
      ↓
[Success] → refresh slot list
```

**Loading from the UI**

```
[Button: Load]
      ↓
[GYB_DoesSaveExist] → SlotIndex: SelectedIndex
      ↓
[Branch] → bExists?
   True  → [GYB_LoadGame] → SlotIndex: SelectedIndex
   False → show "No save data in this slot"
```

**Deleting a slot**

```
[Button: Delete]
      ↓
[Show confirmation dialog]
      ↓
[Confirmed]
      ↓
[GYB_DeleteSave] → SlotIndex: SelectedIndex
      ↓
[Success] → refresh slot list
```

> Always confirm before deleting. There is no undo.

---

## Advanced — Saving custom structs

Any `USTRUCT` with `SaveGame` properties is serialized recursively. You don't need to do anything special — mark the struct variable with `SaveGame` on the actor, and the plugin serializes every `SaveGame` field inside the struct automatically.

**Example**

```cpp
USTRUCT(BlueprintType)
struct FWeaponData
{
    GENERATED_BODY()

    UPROPERTY(SaveGame, BlueprintReadWrite)
    int32 Ammo;

    UPROPERTY(SaveGame, BlueprintReadWrite)
    float Durability;
};
```

Mark the `FWeaponData` variable on your actor with `SaveGame`. Both `Ammo` and `Durability` are saved.

> Nested structs up to 2 levels deep are supported. Deeper nesting is not supported in v1 — the plugin logs a warning if it encounters it.

---

## Advanced — Saving containers

`TArray`, `TMap`, and `TSet` of any supported type are fully serialized. Mark the container variable with `SaveGame` and the entire container is saved and restored.

**Example**

```
Inventory (TArray<FItemData>) — marked SaveGame → entire array saved
UnlockedLevels (TSet<FName>) — marked SaveGame → entire set saved
ActiveBuffs (TMap<FName, float>) — marked SaveGame → entire map saved
```

> `TMap` with actor references as values is not supported in v1.

---

## Project Settings

You can adjust the plugin limits under **Edit → Project Settings → GYBSaveLoadSystem**.

| Setting | Default | When to change it |
|---|---|---|
| `Max Registered Objects` | 500 | Increase if your game has more than 500 saveable actors at once |
| `Max Save Slots` | 10 | Increase or decrease based on how many slots your UI supports |

> Increasing `Max Registered Objects` has no performance cost at save/load time — it only affects the initial allocation. Set it to whatever your game needs.
