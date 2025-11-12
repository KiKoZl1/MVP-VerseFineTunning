
# 02 — State & Transactions (Verse)

**Goal:** Model state explicitly and mutate it through transactional functions to keep invariants enforced and roll‑up side effects.

---

## Key Ideas

- **Single state owner**: a `*_state_manager` class encapsulates all storage and provides safe mutators.
- **<transacts>** on mutators: make side effects and atomic updates explicit.
- **Pure helpers**: read-only computations should be side-effect free for testability.
- **Return structured results**: `tuple(logic, ?reason)` beats `logic` only; prefer typed `fail_reason` enums.

---

## Right: Single Owner + Transactional Mutations

```verse
inventory_state_manager<public> := class:
    Grid<private> : []item_stack = array{}

    GetGrid<public>():[]item_stack = Grid

    WithItem<public>(id: item_id, qty: int)<transacts>:inventory_state_manager=
        var g := Grid
        idx := FindIndex(id, g)
        if (idx >= 0):
            set g[idx].Qty += qty
        else:
            set g += array{ item_stack{ Id := id, Qty := qty } }
        return inventory_state_manager{ Grid := g }

    FindIndex<private>(id: item_id, g: []item_stack):int=
        for (i -> s : g): if (s.Id = id) then return i
        return -1
```

---

## Wrong: Multiple Writers, Hidden Side Effects

```verse
some_feature<public> := class:
    OtherSystem: inventory_state_manager = inventory_state_manager{}

    DoThing<public>():void=
        # ❌ Directly mutates state of another system, bypassing invariants
        set OtherSystem.Grid[0].Qty = 999
```

**Why wrong:** State is mutated from outside the owner; invariants can’t be enforced.

---

## Checklist

- Exactly one class owns persistent state for a domain.
- All mutations pass through `<transacts>` functions.
- Read functions are pure and cheap.
