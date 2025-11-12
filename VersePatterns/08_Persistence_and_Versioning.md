
# 08 — Persistence & Versioning (Verse)

**Goal:** Save/load deterministic snapshots; support schema evolution safely.

---

## Right: Versioned Snapshot + Idempotent Load

```verse
snapshot_v1 := struct{
    Version:int,
    Grid:[]item_stack,
}

inventory_persistence<public> := class:
    Save<public>(st: inventory_state_manager):snapshot_v1=
        return snapshot_v1{ Version := 1, Grid := st.GetGrid() }

    Load<public>(snap:snapshot_v1)<transacts>:inventory_state_manager=
        if (snap.Version = 1) then
            return inventory_state_manager{ Grid := snap.Grid }
        # future migrations...
        return inventory_state_manager{}
```

---

## Wrong: Persist Runtime Handles / Devices

```verse
Save<public>(st: inventory_state_manager):any=
    # ❌ stores live device refs or UI widgets (non-serializable)
    return tuple{ st.Grid, st.HudWidget }
```

**Why wrong:** Snapshots must be stable data only.

---

## Checklist

- Include an explicit `Version` in snapshots.
- Make load idempotent and safe against missing fields.
- Never persist runtime handles, devices, or UI elements.
