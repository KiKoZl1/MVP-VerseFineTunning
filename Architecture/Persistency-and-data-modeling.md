---
id: 36
title: 36. Persistency & Data Modeling — Schemas, Versioning, and Migrations (Generic)
tags: [architecture, persistence, data-modeling, versioning, migrations, verse]
summary: Generic patterns for modeling data, persisting snapshots, and evolving schemas safely; includes versioning, migrations, and acceptance checks.
---

# Persistency & Data Modeling (Generic)

Design **data shapes** and **persistency** so they are stable, evolvable, and testable. Keep schemas **small**, names **clear**, and upgrades **explicit**.

---

## 1) Goals & Constraints (Template)
- **Goals:** durable state across sessions; fast load; predictable upgrades.
- **Constraints:** Verse runtime limits; multiplayer concurrency; data size budgets.
- **NFRs:** reliability, observability, zero-crash on unknown/old fields.

---

## 2) Schema Guidelines
- Use **unit-aware names** (e.g., `Amount`, `Seconds`).
- Prefer **flat records** for hot paths; nest only when necessary.
- Mark **optional** fields with safe defaults during hydrate.
- Keep **IDs** stable (`EntityId`, `OwnerId`).

---

## 3) Versioning & Migrations
Add a `SchemaVersion` and upgrade older snapshots **before** use.

```verse
snapshot_v1 := class:
    SchemaVersion : int = 1
    Currency : int
    Level : int

snapshot_v2 := class:
    SchemaVersion : int = 2
    Currency : int
    Level : int
    PremiumTokens : int = 0  # new optional field

UpgradeToV2(Old : snapshot_v1) : snapshot_v2 =
    return snapshot_v2{ Currency := Old.Currency, Level := Old.Level, PremiumTokens := 0 }
```

- Maintain **one upgrader per version bump**. Chain them if multiple steps.

---

## 4) Hydration & Defaults
```verse
HydrateV2(Raw : snapshot_v2) : snapshot_v2 =
    # normalize ranges and fill missing/invalid values
    C := (Raw.Currency < 0) ? 0 : Raw.Currency
    L := (Raw.Level < 1) ? 1 : Raw.Level
    P := (Raw.PremiumTokens < 0) ? 0 : Raw.PremiumTokens
    return snapshot_v2{ Currency := C, Level := L, PremiumTokens := P }
```

- Always **sanitize** inputs from storage or network.

---

## 5) Save/Load Facade (Generic)
```verse
SaveState(Id : string, Data : snapshot_v2) : logic =
    Print("[Persist] save id={Id} v={Data.SchemaVersion}")
    return true  # implement device call

LoadState(Id : string) : ?snapshot_v2 =
    Print("[Persist] load id={Id}")
    # implement device call; return optional
    return false
```

- Keep storage device/API calls behind a **facade** with logs.

---

## 6) Acceptance Checklist
- [ ] Unknown versions **log & fail soft** (no crash).  
- [ ] Missing fields hydrate with **safe defaults**.  
- [ ] Negative values are clamped or rejected.  
- [ ] Migrations are **idempotent** and covered by a test script.

---

## 7) Anti-Patterns
- Saving raw nested graphs with circular refs.  
- Using enums without string helpers.  
- No version field or “magic” inference.

---

## 8) Finetuning Notes (LLM)
- Keep examples **generic** (no domain names).  
- Show **before/after** for migrations.  
- Prefer facades for I/O; pure functions for upgrades/defaults.
