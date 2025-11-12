
# 01 — Module Organization & Access Modifiers (Verse)

**Goal:** Teach modular design in Verse: clear separation of concerns, stable public APIs, and private internals.  
**Why it matters:** A clean module boundary keeps UI, gameplay logic, persistence, validation, and SFX independent and testable.

---

## Key Ideas

- **One responsibility per file**: e.g., `*_core` (business rules), `*_state` (state shape & helpers), `*_validation` (rules), `*_persistence` (save/load), `*_ui` (widgets/layout), `*_sfx` (feedback), `*_types` (DTOs/enums), `Utils*` (generic helpers).
- **Public vs. Private**: Expose only what the outside world must call. Keep helpers `\<private>` to avoid accidental coupling.
- **Stable API surface**: Put all entry points (`Add`, `Remove`, `Equip`, `Drop`, etc.) in one class; keep signatures small, typed, and transactional when they mutate state.
- **No device/UI references in core**: Core modules shouldn’t import UI or devices; invert control via events or payloads.

---

## Right: Thin Public API, Encapsulated Internals

```verse
using { /Verse.org/Simulation }

inventory_core<public> := class:
    State: inventory_state_manager = inventory_state_manager{}

    # Stable public entry points
    Add<public>(req: add_request)<transacts>:tuple(logic, ?fail_reason)=
        if (not inventory_validation.CanAdd(State, req)) then return tuple{false, option{fail_reason.ValidationFailed}}
        newState := State.WithItem(req.ItemID, req.Quantity)
        set State = newState
        return tuple{true, false}

    Remove<public>(req: remove_request)<transacts>:tuple(logic, ?fail_reason)=
        if (not inventory_validation.CanRemove(State, req)) then return tuple{false, option{fail_reason.ValidationFailed}}
        set State = State.WithRemoved(req.ItemID, req.Quantity)
        return tuple{true, false}

    # Internal helpers stay private
    FindStackIndex<private>(id: item_id):int =
        for (i -> stack : State.Grid):
            if (stack.Id = id) then return i
        return -1
```

**Why this is right:** Public API is minimal; validation is delegated; internal iteration details are hidden.

---

## Wrong: Everything Public, Leaky Internals

```verse
using { /Verse.org/Simulation }

inventory_core<public> := class:
    # ❌ Exposes raw state & mutable arrays to everyone
    Grid<public> : []item_stack = array{}

    Add<public>(id: string, qty: int)<transacts>:void=
        # ❌ Inlined rules, no validation boundary, magic types
        for (i -> stack : Grid):
            if (stack.Id = id):
                set stack.Qty += qty
                return
        set Grid += array{ item_stack{ Id := id, Qty := qty } }
```

**Why this is wrong:** No validation layer, public mutable state, magic strings, no failure reporting.

---

## Checklist

- Files reflect responsibilities (`*_core`, `*_state`, `*_validation`, `*_persistence`, `*_ui`, `*_sfx`, `*_types`, `Utils*`).
- Public methods are **nouns + verbs** describing behavior; helpers are private.
- Cross‑module deps flow **from UI/SFX → Core**, never the reverse.
