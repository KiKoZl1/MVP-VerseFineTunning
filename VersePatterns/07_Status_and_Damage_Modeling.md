
# 07 — Status & Damage Modeling (Verse)

**Goal:** Separate status storage, derived formulas, and damage application so each is testable and reusable.

---

## Right: Interface + Formulas + Applier

```verse
character_status<public> := interface:
    GetHealth<public>():float
    SetHealth<public>(v: float)<transacts>:void
    GetArmor<public>():float

status_formulas<public> := class:
    ResolveDamage<public>(raw: float, armor: float):float=
        return max(0.0, raw - armor)

damage_system<public> := class:
    Apply<public>(st: character_status, raw: float)<transacts>:void=
        dmg := status_formulas.ResolveDamage(raw, st.GetArmor())
        st.SetHealth(st.GetHealth() - dmg)
```

---

## Wrong: Hardcoded Damage Rules Inside Device

```verse
some_device<public> := class:
    OnHit<public>(target:any)<transacts>:void=
        # ❌ magic constants, no formulas layer
        set target.Health = target.Health - 37.5
```

---

## Checklist

- Status storage (manager), formulas (pure), and applier (transactional) are split.
- No magic constants; prefer tunable parameters in one place.
