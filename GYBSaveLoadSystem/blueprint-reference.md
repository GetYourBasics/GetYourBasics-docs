# Blueprint Node Reference

Complete reference for all Blueprint nodes exposed by GYBSaveLoadSystem. Every node is available from any Blueprint — no casting, no getting a reference to a subsystem.

---

## GYB_SaveGame

Saves all registered actors and the GameInstance to a slot.

**Inputs**

| Parameter | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of the slot to save to. Must be between 0 and Max Save Slots - 1. |
| `SlotName` | `FString` | Display name for the slot. Shown in your UI — does not affect the save file. |

**Output**

| Value | Description |
|---|---|
| `Success` | Save completed without errors. |
| `UnknownError` | Something went wrong internally. Check the Output Log for details. |

**Example**

```
[Keyboard Event: S]
      ↓
[GYB_SaveGame] → SlotIndex: 0, SlotName: "Slot 1"
      ↓
[Switch on EGYBSaveResult]
   Success      → Print "Game saved"
   UnknownError → Print "Save failed — check logs"
```

> **Note:** SlotIndex 0 and SlotIndex 1 are different slots. Saving to an existing slot overwrites it without warning — there is no confirmation step built into the node.

---

## GYB_LoadGame

Loads a save slot and restores all actor data.

**Input**

| Parameter | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of the slot to load from. |

**Output**

| Value | Description |
|---|---|
| `Success` | Load completed. All actor data has been restored. |
| `SlotEmpty` | No save file found at this index. |
| `FileCorrupted` | Save file exists but could not be deserialized. |
| `UnknownError` | Something went wrong internally. Check the Output Log for details. |

**Example**

```
[Keyboard Event: L]
      ↓
[GYB_LoadGame] → SlotIndex: 0
      ↓
[Switch on EGYBSaveResult]
   Success        → continue
   SlotEmpty      → Print "No save found"
   FileCorrupted  → Print "Save data is corrupted"
   UnknownError   → Print "Load failed — check logs"
```

> **Important:** The level does not reload on load. Data is restored directly onto existing actors. If your actors have logic that depends on restored values, implement it in the `OnDataRestored` event — it fires on every actor right after its data is applied. See the [Integration Guide](integration-guide.md) for details.

---

## GYB_DeleteSave

Deletes a save slot permanently.

**Input**

| Parameter | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of the slot to delete. |

**Output**

| Value | Description |
|---|---|
| `Success` | Slot deleted successfully. |
| `SlotEmpty` | No save file found at this index. Nothing was deleted. |
| `UnknownError` | Something went wrong internally. Check the Output Log for details. |

**Example**

```
[Button: Delete Slot]
      ↓
[GYB_DeleteSave] → SlotIndex: SelectedSlotIndex
      ↓
[Switch on EGYBSaveResult]
   Success   → refresh slot list in UI
   SlotEmpty → Print "Slot is already empty"
```

> **Warning:** This operation is permanent. There is no undo. Always confirm with the player before calling this node.

---

## GYB_GetAllSlots

Returns info for all save slots, including empty ones.

**Inputs**

None.

**Output**

| Parameter | Type | Description |
|---|---|---|
| `Slots` | `TArray<FGYBSlotInfo>` | Array of slot info structs, one per slot up to Max Save Slots. |

**FGYBSlotInfo fields**

| Field | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of this slot. |
| `SlotName` | `FString` | Display name set when saving. |
| `SaveDateTime` | `FDateTime` | Date and time of the save. |
| `LevelName` | `FString` | Name of the level at save time. |
| `PlayTimeSeconds` | `float` | Total play time in seconds at save time. |
| `SaveVersion` | `int32` | Plugin version used to create this save. |
| `bIsEmpty` | `bool` | True if this slot has no save data. |

**Example**

```
[Event BeginPlay]
      ↓
[GYB_GetAllSlots] → Slots
      ↓
[For Each Loop] → iterate slots
      ↓
[Branch] → bIsEmpty?
   False → add slot to UI list with SlotName and SaveDateTime
   True  → show empty slot placeholder
```

> Use this node to populate a save slot selection UI. All slots are returned regardless of whether they have data — check `bIsEmpty` to distinguish them.

---

## GYB_GetSlotInfo

Returns info for a single save slot.

**Input**

| Parameter | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of the slot to query. |

**Output**

| Parameter | Type | Description |
|---|---|---|
| `SlotInfo` | `FGYBSlotInfo` | Info struct for the requested slot. See field list above. |
| `bSuccess` | `bool` | False if the slot index is out of range or the slot is empty. |

**Example**

```
[GYB_GetSlotInfo] → SlotIndex: 0
      ↓
[Branch] → bSuccess?
   True  → display SlotInfo.SlotName and SlotInfo.SaveDateTime
   False → show "No save data"
```

> Use this when you only need info for one specific slot — for example, showing a "Continue" button with the last save date.

---

## GYB_DoesSaveExist

Returns whether a save slot has data.

**Input**

| Parameter | Type | Description |
|---|---|---|
| `SlotIndex` | `int32` | Index of the slot to check. |

**Output**

| Parameter | Type | Description |
|---|---|---|
| `bExists` | `bool` | True if the slot has save data. False if it's empty or out of range. |

**Example**

```
[Event BeginPlay]
      ↓
[GYB_DoesSaveExist] → SlotIndex: 0
      ↓
[Branch] → bExists?
   True  → show "Continue" button
   False → hide "Continue" button
```

> This is a lightweight check — use it to show or hide UI elements without loading the full slot info.

---

## EGYBSaveResult

The return enum used by `GYB_SaveGame`, `GYB_LoadGame`, and `GYB_DeleteSave`.

| Value | Meaning |
|---|---|
| `Success` | Operation completed without errors. |
| `SlotEmpty` | The requested slot has no save data. |
| `FileCorrupted` | Save file exists but could not be read. |
| `UnknownError` | Internal error. Check the Output Log for details. |

Always handle all possible values. Don't assume `Success`.

---

## OnDataRestored

An event you can implement on any actor that uses `GYBSaveableComponent`. It fires automatically after the actor's data has been restored from a save.

This is not a node you call — it's an event you implement.

**When to use it**

Use `OnDataRestored` for any logic that needs to run after variables are restored — updating a mesh, playing an animation, recalculating derived state. Anything you would normally do in `BeginPlay` based on a variable's value should go here instead.

**Example**

A door actor has a `bIsOpen` boolean marked with SaveGame. On load, `bIsOpen` is restored to its saved value. But the door mesh position isn't a SaveGame variable — it's driven by code. So the door needs to update its mesh after the load.

```
[OnDataRestored]
      ↓
[Branch] → bIsOpen?
   True  → Set door mesh to open position
   False → Set door mesh to closed position
```

Without this, the door variable would be correct but the mesh would still show the default position.

> See [Integration Guide — OnDataRestored](integration-guide.md#ondatarestored) for more examples.
