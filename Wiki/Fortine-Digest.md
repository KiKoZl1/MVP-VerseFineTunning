---
id: digest-fortnite-ui
title: Fortnite UI Device Digest — Buttons, HUD, and Sliders
tags: [fortnite, ui, hud, verse, digest]
summary: Practical guide for `text_button_base`, HUD controllers, and slider widgets with safe Verse patterns and structured logs.
---

# Fortnite UI Device Digest

This digest translates the Fortnite UI device APIs into **actionable Verse snippets** so the model learns how to wire widgets, log state, and guard native calls.

---

## 1) Text Buttons (`text_button_base` family)

- `text_button_base` is the common superclass for `button_loud`, `button_regular`, and `button_quiet`.  
- Use `SetText` during runtime to swap labels and `OnClick` subscriptions to react to input.  
- Always guard subscriptions with a `cancelable` so debug devices can dispose cleanly.

**Good**
```verse
RegisterLoudButton(Button : button_loud, Label : message) : cancelable =
    Button.SetText(Label)
    Print("[UI] bound button label={Label}")
    return Button.OnClick().Subscribe(fun (Msg : widget_message) : void =
        Print("[UI] click player={Msg.Player} source={Msg.Source}")
    )
```

**Poor**
```verse
# No SetText call; subscription never cleaned up
BadBind(Button : button_regular) : void =
    Button.OnClick().Subscribe(fun (_) : void = Print("clicked"))
```

- Set default text before Play so the widget renders even if Verse fails to run.  
- Use translated `message` assets instead of raw strings when localizing.

---

## 2) HUD Controller Access (`fort_hud_controller`)

- `player_ui` exposes `ShowHUDElements`, `HideHUDElements`, and `ResetHUDElementVisibility`.  
- `fort_hud_controller` wraps the same capabilities for a playspace-wide toggle; query it from `fort_playspace`.

```verse
HideCombatHUD(Player : player, Elements : []hud_element_identifier) : void =
    if (UI := GetPlayerUI(Player)):
        Print("[HUD] hide count={Elements.Length}")
        UI.HideHUDElements(Elements)

ToggleRoundInfo(Playspace : fort_playspace, Visible? : logic) : void =
    if (Controller := Playspace.GetHUDController()):
        if (Visible?):
            Controller.ShowElements([creative_hud_identifier_round_info{}])
        else:
            Controller.HideElements([creative_hud_identifier_round_info{}])
```

- Keep the `[]hud_element_identifier` lists short and explicit; mixing unrelated identifiers makes debugging hard.  
- Resets should be logged so QA knows when defaults return.

---

## 3) Sliders (`slider_regular`)

- `slider_regular` exposes editable defaults (`DefaultValue`, `DefaultMinValue`, etc.) but you still need Verse methods to respond to changes.  
- Store the `cancelable` from `OnValueChanged` (once available) or poll via a debug task that reads `GetValue`.

**Good**
```verse
ConfigureVolumeSlider(Slider : slider_regular, TargetVolume : float) : void =
    Slider.SetMinValue(0.0)
    Slider.SetMaxValue(1.0)
    Slider.SetStepSize(0.05)

    Clamped := TargetVolume
    if (Clamped < 0.0):
        set Clamped = 0.0
    else:
        if (Clamped > 1.0):
            set Clamped = 1.0

    Slider.SetValue(Clamped)
    Print("[UI] slider target={Clamped}")
```

**Poor**
```verse
# Values never clamped; assumes defaults already match design
ConfigureVolumeSlider(Slider : slider_regular) : void =
    Slider.SetValue(3.0)
```

- Always clamp user-driven values before applying them to gameplay state.  
- Log both the requested value and the clamped result when debugging.

---

## Finetuning Notes (LLM)

- Tie every native API (`SetText`, `ShowHUDElements`, `SetValue`) to a **guarded Verse snippet**.  
- Reinforce cleanup via `cancelable` returns for widget events.  
- Encourage structured logs with subsystem prefixes (`[UI]`, `[HUD]`).  
- Contrast good vs. poor usage so the model learns to favor guards, clamping, and disposables.
