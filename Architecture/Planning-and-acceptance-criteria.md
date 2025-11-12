---
id: 34
title: 34. Planning & Acceptance Criteria — From Stories to Verse Tasks
tags: [planning, acceptance, templates, verse, testing]
summary: Walkthrough for turning user stories into Verse-ready tasks with crisp acceptance tests and play scripts.
---

# Planning & Acceptance Criteria — From User Stories to Verse Tasks

## Purpose
Teach the LLM to transform **user stories** into **implementable Verse tasks** with precise acceptance criteria and test scripts.

## User Story Canon
- **Format:** As a <player/designer/system>, I want <capability> so that <value>.  
- **Constraints:** <performance, reliability, persistence, multiplayer>.  
- **Notes:** Include dependencies and events.

## Decomposition Pattern
1) Identify **Domain** (Inventory, Loot, Player, Enemy, Dungeon, UI).  
2) Extract **Events** the story touches.  
3) Define **Data** (structs/enums/persistables).  
4) Specify **Public APIs** and return types.  
5) Write **Acceptance Criteria** (functional + NFR + negative cases).  
6) Provide a **Play Test** procedure and expected logs.

## Acceptance Criteria Library (Reusable Snippets)
### Inventory / Items
- When `ItemSpawner.SpawnFromLoot` fires and the player collects within ownership, **Inventory.AddItem** must add up to capacity, split stacks above `MaxStack`, and set `PartialCollect` correctly.  
- On first weapon added to an empty hotbar, **auto‑equip** and emit `OnEquippedChanged`; hotbar selects that slot.

### Loot V2 (Data‑Driven)
- Given a `loot_table_base` with level bands and pity configuration, **GenerateLoot** returns at least `MinDrops`, applies guaranteed entries, and forces a rare item when `RollsSinceRare >= Threshold`.  
- `loot_delivery_handler` respects `DropMode` (World/Inventory/Hybrid) and uses `item_spawner` layouts.

### Player System V2 (Lifecycle, XP, Currency)
- On `PlayerAdded`, `PlayerManagerV2.EnsurePlayer` loads or initializes snapshot; emits `OnPlayerInitialized` with Level/Currency/Class.  
- On XP gain, `OnPlayerXPChanged` fires with `{Player, NewLevel, CurrentXP, RequiredXP}`; level‑up grants rewards and persists.

### Skills Cooldown Persistence
- Removing a skill from a slot **does not reset** its cooldown; runtime instance is unique per player+skill and keeps ticking.

### Enemy XP Distribution
- On enemy elimination, XP is **proportional to DamagePool shares**; grants at least 1 XP per contributor and logs distribution.

## Non‑Functional Requirements (Examples)
- **Performance:** <2ms per `AddItem` in hot path; zero heap growth under 100 spawns/min.  
- **Reliability:** No silent failures; all guardable accesses are guarded.  
- **Observability:** Every acceptance criterion has a corresponding log line and/or debug device action.  
- **Multiplayer:** Behavior deterministic with 1–4 players.

## Definition of Ready (DoR)
- Story has a measurable goal, dependencies, constraints, and test plan.  
- Unknowns listed with an exploration task.

## Example — Story → Tasks
**Story:** As a player, I want **chest loot** to scale with my level so that low‑level chests don’t drop end‑game gear.  
**Tasks:**
1) Create `chest_loot_table_*` inheriting from `loot_table_base` with level bands.  
2) Update chest device to hold `@editable LootTable : loot_table_base`; build `loot_context_v2` with `PlayerLevel`.  
3) Extend `loot_delivery_handler` tests to verify pity and bands.  
4) Add debug button “Test Chest Tables”.  
**Acceptance:** With level 3 and difficulty Easy, common chest yields only Common/Uncommon pools; pity triggers after N attempts; logs prove band selection.

## Play Test Script (Template)
1) Enter Play; spawn debug devices.  
2) Trigger the feature (button / elimination / pickup).  
3) Observe logs: `[Feature] ...`; cross‑check event order.  
4) Repeat with 1/2/4 players.  
5) Record observations and deltas.

## Finetuning Notes (LLM)
- Demonstrate how to move from **story format** to concrete Verse deliverables (events, APIs, data).  
- Tie every acceptance statement to observable signals/logs so completions naturally include verification hooks.  
- Provide reusable prose checklists; the model should learn to output structured bullets, not free-form paragraphs.  
- Include cross-domain examples (inventory, loot, XP) so the style generalizes.
