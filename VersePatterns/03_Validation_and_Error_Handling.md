
# 03 — Validation & Error Handling (Verse)

**Goal:** Centralize rules and provide consistent, typed failures.

---

## Right: Centralized `*_validation` with Typed Reasons

```verse
fail_reason := enum { None, NoSpace, NotFound, WrongSlot, InvalidQty }

inventory_validation<public> := class:
    CanAdd<public>(st: inventory_state_manager, req: add_request):logic=
        return req.Quantity > 0 and st.HasCapacity(req.ItemID, req.Quantity)

    CanRemove<public>(st: inventory_state_manager, req: remove_request):logic=
        return req.Quantity > 0 and st.HasAtLeast(req.ItemID, req.Quantity)
```

Usage from the core:

```verse
(ok, reason?) := inventory_validation.CanAdd(State, req) ? tuple{true, false} : tuple{false, option{fail_reason.NoSpace}}
```

---

## Wrong: Scattered Inline Checks + Booleans Only

```verse
Add<public>(req: add_request)<transacts>:logic=
    if (req.Quantity <= 0) then return false  # ❌
    if (Weight + req.Weight > 100) then return false  # ❌
    # ...
    return true
```

**Why wrong:** No typed reason, rules duplicated in many sites.

---

## Checklist

- A single `*_validation` module exposes `CanXxx(...)` predicates.
- Public API returns `tuple(logic, ?fail_reason)` or a typed result record.
