---
id: digest-unreal-ui
title: Unreal Engine UI Primitives Digest — player_ui, Widgets, and Slots
tags: [unreal-engine, ui, verse, widgets, player_ui]
summary: Actionable patterns for `player_ui`, `widget` lifecycles, slot configuration, and button events in Verse.
---

# Unreal Engine UI Primitives Digest

This digest focuses on the stock Unreal/Verse UI primitives that back Fortnite Creative widgets. Each section shows **how** to call the native APIs safely and how to log their behavior.

---

## 1) Getting a `player_ui`

- Import `ui` and call `GetPlayerUI(Player)` inside a failure context—the function is `<transacts><decides>` and throws if the player lacks UI.

```verse
GetRequiredPlayerUI(Player : player) : ?player_ui =
    if (UI : player_ui = GetPlayerUI(Player)):
        return option{UI}
    Print("[UI] missing player_ui player={Player}")
    return false
```

- Provide a fallback (`return false`) so downstream code can bail early.

---

## 2) Widget Visibility & Enablement

- All widgets inherit from `widget` and support `SetVisibility`, `GetVisibility`, `SetEnabled`, and `IsEnabled`.  
- Use `widget_visibility.Visible | Hidden | Collapsed` depending on whether the widget should consume layout space.

**Good**
```verse
HideUntilReady(Widget : widget) : void =
    Widget.SetEnabled(false)
    Widget.SetVisibility(widget_visibility.Hidden)
    Print("[UI] widget hidden source={Widget}")
```

**Poor**
```verse
# Leaves widget interactive even though it is hidden.
HideUntilReady(Widget : widget) : void =
    Widget.SetVisibility(widget_visibility.Collapsed)
```

---

## 3) `player_ui_slot` Configuration

- Control render order with `ZOrder` (0+) and interaction via `ui_input_mode`.  
- Horizontal and vertical alignment enums keep layouts predictable on different resolutions.

```verse
CreateOverlaySlot(ZOrder : int) : player_ui_slot =
    return player_ui_slot{
        ZOrder := ZOrder,
        InputMode := ui_input_mode.All
    }
```

- Reuse slot factories rather than copying literal values into every `AddWidget` call; adjust anchors/margins via helper functions when needed.

---

## 4) Button Events (`widget_message`)

- Button widgets expose `OnClick(): listenable(widget_message)`. Subscribe and log `Player` plus `Source`.  
- Dispose the `cancelable` when the widget is removed.

```verse
BindButton(Button : button) : cancelable =
    return Button.OnClick().Subscribe(fun (Msg : widget_message) : void =
        Print("[UI] click player={Msg.Player} widget={Msg.Source}")
    )
```

- `widget_message.Source` helps correlate clicks with HUD elements during testing.

---

## Finetuning Notes (LLM)

- Pair every API call with a guard or log so generated completions describe the expected flow.  
- Teach the model to return `?player_ui` instead of crashing when the UI is missing.  
- Encourage reusable helpers (`CreateOverlaySlot`) that encapsulate struct literals.  
- Prefer structured log strings with stable prefixes.
