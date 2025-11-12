
# 05 — UI Overlays & Theme Composition (Verse)

**Goal:** Decouple UI shell/layout from business logic; drive UI through data and theme tokens.

---

## Right: UI Shell Delegates to Theme + Manager

```verse
ui_theme<public> := class:
    InventoryBGImage : ui_image
    InventoryBGSize  : vector2
    # ... all UI tokens centralised

inventory_ui_shell<public> := class:
    Manager : inventory_ui_manager = inventory_ui_manager{}
    Theme   : ui_theme = ui_theme{}

    Build<public>():void=
        bg := MakeImage(Theme.InventoryBGImage, Theme.InventoryBGSize)
        Manager.Attach(bg)
```

**Why right:** Theme tokens are centralized; shell only composes primitives; no core logic calls here.

---

## Wrong: UI Directly Touches Core State

```verse
inventory_ui<public> := class:
    Core : inventory_core = inventory_core{}

    Build<public>():void=
        # ❌ reads and mutates core state during widget build
        Core.Add(add_request{ ItemID := "A", Quantity := 10 })
```

**Why wrong:** UI should not mutate domain state during build; use commands/events from input handlers.

---

## Checklist

- `ui_theme` provides tokens (images, sizes, spacing).
- `*_ui_shell` builds once; `*_ui_manager` updates views via handlers.
- No business-rule mutation inside UI layout code.
