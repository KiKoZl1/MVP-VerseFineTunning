# UEFN Verse Finetuning Corpus

**Goal:** Provide a high-quality, English-only Markdown corpus that teaches an LLM to read and write Verse with **clarity**, **safety**, and **architectural discipline**.  
This repository mixes two complementary tracks:
- **Language & Patterns:** idiomatic Verse constructs, formatting, failure contexts, safety, and reusable utilities.
- **Architecture & Planning:** domain-agnostic blueprints to plan systems before coding (system maps, events, APIs, data shapes, acceptance tests).

## Principles
- **English-only** for all `.md` content. (Chat/discussions can be another language, but docs stay in English.)
- **Domain-agnostic** architecture (no feature-specific names). Teach **best practices**, not a single game.
- **One topic per file**, with **sequential IDs** and no subfolders.
- Each topic unifies material from multiple sources into a **single, coherent** document.
- Examples use **positive vs. negative** snippets and **structured logs** (short, key=value style).
- Prefer **guards** (failure contexts, bounds checks), **pure helpers**, and **facades** for engine/device I/O.

## Repository Layout
- **01–10**: Style & Core Language (naming, formatting, functions, failure checks, encapsulation, events, concurrency, attributes, constructors).  
- **11–17**: Core Patterns & Flow (variables/constants, collections, expressions/types, playspace/players, etc.).  
- **18–26**: Gameplay Utilities & Safety (spawning/teleport, timers/cooldowns, logging, randomness/clamping, teams, collections, movement, rewards, safety patterns).  
- **27–32**: Ecosystem & Integration (math/transforms, class design, modules/imports, devices, UI prompts, testing).  
- **33–49**: Architecture & Planning (planning templates, acceptance criteria, event-driven patterns, persistency & data modeling).

See **SUMMARY.md** for the full index generated from the current tree.

## How to Use
- As a **reference** while coding in Verse.
- As a **finetuning dataset**: sample entire files or chunk into sections/tasks. Keep train/val splits at file boundaries to avoid leakage.
- As a **prompting library**: copy the planning templates when asking the model to propose a system design.

## Authoring Rules (for contributors)
1. **Filenames**: `NN.-topic-name.md` (sequential IDs, dash-separated words). If editing significantly, you may publish a `-REV1` variant to avoid collisions in toolchains.
2. **Front matter**: include `id`, `title`, `tags`, `summary` when applicable.
3. **Structure**: brief intro → small numbered sections → examples (good/poor) → finetuning notes.
4. **Generic first**: in Architecture & Planning, use neutral names like `System`, `Entity`, `Resource`, `Manager`, `Device`. Avoid feature names.
5. **Safety**: guard all map/array/optional access; keep effects last in a block; prefer idempotent helpers; log context with stable keys.

## Quality Checklist (per file)
- [ ] English-only; no ambiguous terms or local slang.  
- [ ] Topic-scope is focused; no sub-topics spillover.  
- [ ] Contains at least one **good** and one **poor** example.  
- [ ] Shows at least one **guard** (failure context or bounds check).  
- [ ] Logs are short and structured (key=value).  
- [ ] Finetuning notes summarise the teaching objective.

## License
- Add your preferred license here (e.g., MIT). Ensure third-party content is **paraphrased** and citations (if any) are handled per your policy.

## Acknowledgements
- Built to help teams learn Verse and plan systems responsibly, with consistent patterns that a model can internalize.
