---
id: 33
title: 33. Architecture Blueprint Template — Plan Before Verse Coding
tags: [architecture, planning, blueprint, verse, template]
summary: Repeatable worksheet that guides an LLM through goals, events, APIs, and acceptance criteria before it writes Verse code.
---

# Architecture Blueprint (UEFN / Verse) — Planning Template

## Purpose
Give the LLM a **repeatable planning template** to design UEFN/Verse systems *before* writing code. This file teaches structure, vocabulary, and deliverables so the model can propose complete, testable plans.

## When to Use
- Starting a new feature (e.g., Inventory UI, Loot V2, Player System V2).
- Refactoring legacy logic into Devices/Managers/Components.
- Scoping a vertical slice with clear acceptance criteria.

## Inputs (Checklist)
- Goal statement in one sentence.
- Constraints (platform, Verse limits, performance, multiplayer).
- Dependencies (devices, managers, data assets).
- Non‑functional requirements (NFRs): performance, reliability, observability.
- Risks & unknowns with proposed mitigation.
- Test protocol (Play test steps, debug devices).

## Outputs (Checklist)
- **System Map** (devices ↔ managers ↔ components ↔ events).
- **Event Catalog** (payloads, subscriptions, cancelables).
- **Public API** for each device/manager (names, params, effects).
- **Data shapes** (structs, enums, persistables).
- **Initialization order** and async flows (`spawn`, `race`).
- **Acceptance criteria** and **Done Definition**.
- **Telemetry/Logs** to verify behavior in‑game.

## Canonical System Map (Skeleton)
```
[Player] ─┬─> PlayerManagerV2 (Events: OnPlayerInitialized/Removed, OnXPChanged, OnCurrencyChanged)
          ├─> InventoryDevice (AddItem, OnEquippedChanged)
          ├─> LootGeneratorV2 (GenerateLootByName/Instance)
          ├─> ItemSpawner (SpawnItem/FromLoot, Collector)
          └─> EffectsManager (OnDamagingEvent/OnDamagedEvent)
```

## Verse Patterns (Ready-to-Adapt Stubs)
```verse
# === Device Registration & Typed Getter ===
my_device := class(creative_device):
    OnBegin<override>()<suspends>:void=
        SetDevice("MyDevice", Self)
        spawn. Initialize()

GetMyDevice<decides>():my_device=
    if (Dev := GetDevice("MyDevice")?, Typed := my_device[Dev]):
        return Typed
    return my_device{} # fallback for debug only

# === Generic Event Manager Skeleton ===
payload_example := class:
    Value : int

event_subs := class(generic_subscriptions(payload_example)) {}
event_sig  := class(generic_event(payload_example)) {}

event_manager := class:
    OnExample : event_sig = event_sig{}
    GetSubs():event_subs= event_subs{ OnExample }
```

## Planning Algorithm (Runbook)
1) **Define the Goal** and boundaries (what is explicitly not included).  
2) **List Events** the system must emit/consume (who listens, when, with what payload).  
3) **Design Data Shapes** (enums, structs) and persistence strategy (weak_map / persistable).  
4) **Sketch APIs** for each device/manager (inputs/outputs, `<transacts>` where needed).  
5) **Bootstrap Order** (who registers first, who subscribes before first signal).  
6) **Failure Paths & Retries** (guards for optionals, set‑checks on maps/arrays).  
7) **Observability** (logs, string helpers for enums, debug device buttons).  
8) **Acceptance Criteria** (functional + NFRs + negative tests).  
9) **Risks & Mitigations** (fall back to legacy, feature flags, toggles).  
10) **Play Test Script** (exact actions to reproduce and verify).

## Definition of Done (DoD)
- All public APIs documented with parameter semantics and failure modes.  
- Event flows proven with logs and a debug device.  
- No prints of raw enums/agents; only helper‑stringified values.  
- Guards for all optional/map/array accesses.  
- Async tasks cancel on shutdown or event signal.  
- A Play test passes end‑to‑end with a checklist.

## Anti‑Patterns (Reject During Review)
- Ad‑hoc events per device (use shared manager + generic events).  
- Direct access to another system’s internal state (use typed getters).  
- `<transacts>` sprayed everywhere (mark only functions used in failure context).  
- Map/array access without `if (Index < Arr.Length, Value := Arr[Index])`.  
- Local `manager{}` instances that other systems cannot see (register in `OnBegin`).

## Example: Mini Plan (Inventory Auto‑Equip)
**Goal:** Auto‑equip the first weapon added when the player has no weapon.  
**Events:** `OnItemCollected` → InventoryDevice → `OnEquippedChanged`.  
**APIs:** `InventoryDevice.AddItem(Player, ItemID, Quantity, ?AutoEquip=true)` returns `{Added, Remaining, Partial}`.  
**Acceptance:** First weapon equips; hotbar selects its slot; logs show equip change; no crash on full category.  
**NFR:** <2ms per add; no extra allocations on hot path.

## Prompting Hints for the LLM
- “Propose the **System Map**, **Events**, **APIs**, **Data Shapes**, **Init Order**, and **Acceptance Criteria** for <FEATURE> following this template.”  
- “List **risks** and **negative tests** first, then write the plan.”

## Finetuning Notes (LLM)
- Emphasize the **order of thinking** (goals → events → data → APIs → acceptance) so completions stay structured.  
- Show how to mix prose checklists with Verse snippets for reusable planning artifacts.  
- Keep terminology neutral (Device, Manager, Payload) to avoid overfitting to a single game.  
- Reinforce logging, guards, and debug hooks as non-negotiable acceptance items.
