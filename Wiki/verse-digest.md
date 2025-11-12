---
id: digest-verse-runtime
title: Verse Runtime Digest — Simulation, Tags, Arrays, and Events
tags: [verse, runtime, simulation, tags, arrays, events]
summary: Curated reference for core Verse modules such as Simulation tags, array helpers, sleep semantics, and event/cancelable patterns.
---

# Verse Runtime Digest

This digest packages frequently used Verse runtime modules into short patterns so the LLM can recall the API surface without dumping raw SDK text.

---

## 1) Simulation & `Sleep`

- `Sleep(Seconds)` suspends the current task; `0.0` yields to the next tick, `Inf` blocks until canceled, negative values complete immediately.  
- Guard loops so they never `Sleep(0.0)` without additional exit logic.

**Good**
```verse
Ticker(IsActive? : logic) : void =
    loop:
        if (not IsActive?):
            return
        Update()
        Sleep(0.1)
```

**Poor**
```verse
# Sleeps forever; Update never executes.
Ticker() : void =
    Sleep(Inf)
    Update()
```

- Use `GetSimulationElapsedTime()` for profiling logs or timeout logic.

---

## 2) Gameplay Tags (`tag`, `tag_view`)

- Tags are hierarchical strings (e.g., `Ability.Damage.Fire`).  
- `tag_view.Has`, `.HasAny`, and `.HasAll` are `<transacts><decides>` helpers; always call them inside a failure context.

```verse
HasTag?(Tags : tag_view, Query : tag) : logic =
    if (Tags.Has(Query)):
        return true
    Print("[Tags] missing tag={Query}")
    return false
```

- Keep frequently used tag sets in arrays so you can call `HasAny` with a single allocation.

---

## 3) Array Utilities (`Verse` module)

- `Concatenate`, `Slice`, `Insert`, `RemoveElement`, and `ReplaceElement` return **new arrays**—they do not mutate the original input.  
- Guard bounds (`0 <= Index < Length`) before invoking them to avoid runtime failures.

```verse
SafeRemove(Arr : []int, Index : int) : []int =
    if (Index < 0 or Index >= Arr.Length):
        Print("[Array] skip remove index={Index} len={Arr.Length}")
        return Arr
    return Arr.RemoveElement(Index)
```

- Use `Find` to fetch the first matching index; it fails when not found, so wrap it in an `if`.

---

## 4) Events, `cancelable`, and `disposable`

- `event<T>` pairs `Signal` and `Await`, resuming tasks in FIFO order.  
- Subscriptions return a `cancelable`; call `Cancel()` when disposing a device or when the listener is no longer valid.

```verse
payload := class:
    Value : int

counter_event := class(event(payload)) {}

SubscribeOnce(Evt : counter_event) : cancelable =
    return Evt.Subscribe(fun (P : payload) : void =
        Print("[Event] value={P.Value}")
    )
```

- Implement `disposable` when your custom class owns native resources; call `Dispose()` inside `OnEnd` or teardown hooks.

---

## Finetuning Notes (LLM)

- Emphasize **guarded calls** for every `<transacts>` function (tags, arrays).  
- Prefer helper wrappers (`SafeRemove`, `HasTag?`) instead of sprinkling raw SDK calls.  
- Highlight good vs. poor concurrency patterns so the model avoids infinite sleeps or missed cancels.  
- Keep digest sections focused on a single runtime concept to reduce prompt noise.
