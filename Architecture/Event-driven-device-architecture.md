---
id: 35
title: 35. Event-Driven Device Architecture — Patterns & Anti-Patterns
tags: [architecture, events, devices, verse, patterns]
summary: Defines how devices, managers, and generic events collaborate so Verse systems stay decoupled, observable, and safe.
---

# Event‑Driven Device Architecture — Patterns & Anti‑Patterns (UEFN / Verse)

## Purpose
Codify the architecture patterns we expect across the project so the LLM produces **consistent, decoupled** Verse systems.

## Core Patterns
### 1) Device + Manager
- **Device (creative_device):** lifecycle, registration, editor‑facing @editable fields, spawning/cleanup.
- **Manager (plain class):** domain logic and events; owned by a device or made global via typed getters.

### 2) Global Access via Typed Helpers
```verse
GetInventoryDevice<decides>():inventory_device=
    if (Dev := GetDevice("InventoryDevice")?, Typed := inventory_device[Dev]):
        return Typed
    return inventory_device{} # debug fallback only
```

### 3) Generic Events (Publish/Subscribe)
- Build with `generic_subscriptions`, `generic_event`, `generic_cancelable`.
- Define payload classes per domain (`item_spawned_payload`, `inventory_changed_payload`).  
- **Managers** host events and **devices** signal/subscribe. Never instantiate ad‑hoc local managers.

### 4) Initialization Order
- Subscribe **before** first signal.
- `OnBegin` registers devices and spawns async initializers.  
- `spawn { ... }` for background loops; cancel with event signals or session end.

### 5) `<transacts>` Usage
- Only mark functions actually used in failure contexts (`if (...)`, `and`, `or`).  
- Void helpers should **not** receive `<transacts>`.  
- Invoke `<transacts>` functions with `[]`, not parenthesis.

### 6) Optionals, Maps & Arrays (Guarded Access)
```verse
# Map write with success check
if (set PlayerMap[Player] = State):
    {}

# Array read with bounds guard
if (Idx < Arr.Length, Val := Arr[Idx]):
    {}
```

### 7) Logging & String Helpers
- Never print enums or agents directly; provide `EnumToString`/`GetSpawnContextName` helpers.  
- Prefer compact, structured logs (prefix with subsystem tag).

## Anti‑Patterns (Reject)
- Local `manager{}` that no other system can see (missing `SetDevice`).  
- Events emitted from random devices without a shared manager.  
- Unchecked `Map[Key]` or `Array[Index]` accesses.  
- “God device” that owns unrelated domains.

## Example — Event Manager + Device Pair
```verse
# Payloads
item_spawned_payload := class:
    ItemID : int
    Quantity : int
    Owner : ?player = false

# Subscriptions and event signal
item_event_subs := class(generic_subscriptions(item_spawned_payload)) {}
item_event := class(generic_event(item_spawned_payload)) {}

# Manager
item_event_manager := class:
    OnItemSpawned : item_event = item_event{}

# Device hosting the manager
item_events_device := class(creative_device):
    Manager : item_event_manager = item_event_manager{}

    OnBegin<override>()<suspends>:void=
        SetDevice("ItemEvents", Self)

GetItemEvents<decides>():item_event_manager=
    if (Dev := GetDevice("ItemEvents")?, D := item_events_device[Dev]):
        return D.Manager
    return item_event_manager{} # debug only
```

## Bootstrapping Checklist
- [ ] Register devices with stable names in `OnBegin`.  
- [ ] Provide typed getters for every global dependency.  
- [ ] Subscribe to events before first spawn/signal.  
- [ ] Validate all optional/map/array operations.  
- [ ] Keep managers pure; push I/O to devices.  
- [ ] Expose a debug device to drive critical flows.

## Review Questions
- What happens on failure paths? Where do we log and recover?  
- Can listeners subscribe late and still get the needed state?  
- Do we have cancellation for `spawn`/`race` blocks?  
- Is persistence state owned by a single authoritative device?

## Negative Test Ideas
- Emit events with missing/invalid payloads (should log, not crash).  
- Remove a device mid‑play; ensure getters fail soft and logs guide recovery.  
- Flood events (100/min) and verify no heap growth.

## Finetuning Notes (LLM)
- Model should associate **SetDevice + typed getter** with every shared manager.  
- Reinforce guarded map/array access inside publish/subscribe flows.  
- Encourage compact, structured logs for every signal, so completions naturally log context.  
- Include both positive skeletons and anti-pattern callouts to sharpen classifier behavior during training.
