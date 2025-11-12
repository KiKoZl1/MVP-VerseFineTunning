
# 06 — Events, Signals & SFX (Verse)

**Goal:** Broadcast gameplay changes as events/payloads; SFX/UI subscribe and react—no hard coupling.

---

## Right: Payload-Driven SFX

```verse
equipped_changed_payload := struct{ Slot:int, NewId:item_id, PrevId:?item_id }

sfx_system<public> := class:
    OnEquippedChanged<public>(p: equipped_changed_payload):void=
        if (p.NewId?):
            Play("equip")
        else:
            Play("unequip")
```

Emitter side (core):

```verse
Equip<public>(req: equip_request)<transacts>:tuple(logic, ?fail_reason)=
    # ... mutate state
    Sfx.OnEquippedChanged(equipped_changed_payload{ Slot := req.Slot, NewId := req.ItemID, PrevId := prev? })
    return tuple{true, false}
```

---

## Wrong: UI/SFX Reach into Core

```verse
hud_overlay<public> := class:
    Core : inventory_core = inventory_core{}

    OnClickEquip<public>():void=
        # ❌ Calls deep internals and expects specific structure
        set Core.State.Grid[0].Id = "Sword"
```

**Why wrong:** Violates encapsulation; no payloads; brittle.

---

## Checklist

- Define small DTO payloads for key domain events.
- SFX/UI subscribe to events or are called with payloads.
- No reverse dependencies from SFX/UI into core details.
