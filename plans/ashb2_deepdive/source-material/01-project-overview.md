# 01 — Project Overview

*ASHB2 is a from-scratch C++17 "society in a box": psychologically rich individual agents whose moment-to-moment decisions, relationships, and deaths emergently produce tribes, religions, economies, wars, and history — visualized live and narrated, after the fact, by an AI historian.*

## What it is

ASHB2 is an **agent-based civilization & life simulator**. It models individual "entities" (people) with deep psychology — Big Five personality, attachment style, continuously-ticking needs, an appraisal-driven emotion system, grief, memory, and goals — and lets their interactions aggregate *upward* into tribes, religions, tech, diplomacy, and macro-history. The whole thing is **algorithmic and deterministic**: there is **no LLM inside the simulation**. The only AI touchpoint is *offline* — a Python pipeline that turns the run's event logs into a narrative "post-mortem" chronicle (see [10-infrastructure-and-postmortem.md](10-infrastructure-and-postmortem.md)).

It renders live through **Dear ImGui** (a pannable social graph, tribe views, entity stat panels, a color-coded History & Report window) with a second, currently **dormant** SDL2 backend, and plots with **ImPlot**.

## By the numbers

| Metric | Value |
|--------|-------|
| Language / standard | C++17 |
| Build | CMake + MinGW (also a 50 KB `Makefile`); OpenMP optional |
| Tracked C++ in `src/` | **40,645 LoC** across **101 files** (49 `.cpp` + 52 `.h`) |
| Hand-written simulation code | **~28,900 LoC** (excludes 11,775 LoC vendored ImPlot) |
| Named subsystems (header inventory) | **~40** |
| Largest hand-written file | `src/implem_free_will.cpp` — **4,030 LoC** (the decision engine) |
| Next largest | `main.cpp` 2,067 · `CivilizationEngine.cpp` 1,972 · `UI.cpp` 1,101 · `Entity.cpp` 892 · `PlanningSystem.cpp` 798 |
| Vendored libraries | GLFW, Dear ImGui, SDL2, ImPlot, `BetterRand`, `fastDiv` |
| Render backends | **1 live** (GLFW+ImGui "statistics" mode); a second SDL2 path exists but is dead-wired |
| LLM in the simulation | **0** — offline post-mortem only |
| Determinism | master-seeded; deterministic replay is **single-thread-only** (one shared global RNG + construction-order counter) |
| Post-mortem generator | `postmortem_generate.py` (~31 KB) + `src/AI_SUMMARY_PROMPT.md` |

> **Provenance note (honest):** this repository's own git history is only three import commits (`begin`, `fix Imgui`, `add lib`) — it is a **received snapshot**, not the original development history. The real trail lives in the artifacts: a French-language `TODO`, dated design plans and post-mortems under `plans/`, `data/saves/` checkpoints, and a `claude --resume <uuid>` reference in `question.md`. Everything here reads as an actively-developed, Claude-assisted **solo project** by a francophone developer. Contributor/commit statistics are therefore not meaningful and are omitted deliberately.

## The two-layer bet

The architecture's core thesis (elaborated in [02-architecture.md](02-architecture.md)) is a **micro → macro** stack:

1. **Micro — the individual.** Each `Entity` is a deeply modeled person: Big Five + attachment drive behavior; needs tick continuously (hunger, fatigue, hygiene, stress, loneliness, boredom, mental health, happiness, health) with a food store and metabolism (*eat or starve*); a `FreeWillSystem` scores candidate actions each tick and picks one; relationships grow from proximity and decay over time; and an inner life adds grief (Kübler-Ross), formative memories that permanently shift personality, a Tree-of-Thoughts planner, semantic memory, and generated first-person inner monologue. See [03](03-entity-and-psychology.md), [04](04-free-will-decision-engine.md), [05](05-cognition-memory-planning.md), [06](06-relationships-and-social-order.md).

2. **Macro — the civilization.** A `CivilizationEngine` aggregates entities into tribes (with cultural values, leaders, a granary + division of labor), religions (founded from a leader's personality, with schisms), a prerequisite-gated tech tree, diplomacy (alliances, treaties, ethnic/hate wars), and kinship/social-order (family registry with incest avoidance, social classes, debt→slavery, inheritance). See [07](07-civilization-engine.md).

Underneath both sits a **procedurally generated planet** — noise-based terrain, biomes, isolated regions, per-region language drift, and a Malthusian carrying-capacity loop that makes climate genuinely gate survival. See [08-world-and-environment.md](08-world-and-environment.md).

## Where it sits in the landscape

ASHB2 occupies a distinctive corner of the emergent-simulation space:

- **vs. Stanford's Generative Agents / "Smallville" (and petri-dish-of-madness):** the *algorithmic inverse*. Those drive agent cognition with an LLM (memory stream → retrieval → reflection → planning in natural language). ASHB2 hand-codes the equivalent machinery in C++ — cheap, deterministic, and able to run *thousands* of agents where LLM sims run tens. See the full contrast in [11-comparison-petri-dish.md](11-comparison-petri-dish.md).
- **vs. Dwarf Fortress / RimWorld:** shares the "emergent story generator" DNA, but ASHB2 leads with *psychology* (Big Five, attachment, appraisal emotion, grief) as the engine of history rather than with logistics or combat.
- **vs. The Sims:** shares needs-driven agents, but pushes far past them into personality-driven decision arbitration and civilization-scale emergence.

Its signature move is the combination almost no other project attempts at once: **deep individual psychology + civilization-scale emergence + deterministic procedural world + an AI narrator** — all in dependency-light hand-written C++.

## What's impressive, and what's honest

- **Impressive:** the breadth of hand-modeled psychology; the fact that jealousy dynamics alone produce a coherent, *observable* social pattern (~89% of deaths in recent runs are jealousy-driven "crimes of passion" — see [06](06-relationships-and-social-order.md)); a seeded procedural planet that yields *wildly different histories per seed*; and a genuinely delightful log→narrative post-mortem pipeline.
- **Honest limitations (developed across the series):** the decision engine is a **weighted sum of hand-tuned modifiers** with no principled arbitration theory ([04](04-free-will-decision-engine.md)); timescales, trait correlations, and appraisal rules are **hand-guessed constants** ([03](03-entity-and-psychology.md)); a **striking number of powerful systems are written but never called** — `EmotionalComplexitySystem`, a Maslow needs ladder, `SocialDynamics` (Sternberg triangular-theory relationships), force-directed movement, the `SpatialMesh` quadtree, the SDL backend, and `EnvironmentModel`/`CulturalTransmissionSystem` are all compiled-but-dormant (see the [02](02-architecture.md) dead-code inventory); deterministic replay works but is **single-thread-only**, and the design docs still warn about a self-seeding gap the code has already closed ([10](10-infrastructure-and-postmortem.md)); and there is **no behavioral ground-truth validation** — nothing checks that the emergent patterns are *realistic* ([10](10-infrastructure-and-postmortem.md)). The original author catalogs several of these himself in `question.md`; the [14-expansion-roadmap.md](14-expansion-roadmap.md) turns them into a concrete plan.

## How to read this series

Start here → [02-architecture.md](02-architecture.md) for the system map. Then read the micro stack ([03](03-entity-and-psychology.md)→[06](06-relationships-and-social-order.md)), the macro/world stack ([07](07-civilization-engine.md)→[08](08-world-and-environment.md)), and the runtime/infra ([09](09-simulation-loop-and-rendering.md)→[10](10-infrastructure-and-postmortem.md)). The strategic payoff is [11](11-comparison-petri-dish.md)→[13](13-frontier-assessment.md) (what petri-dish-of-madness can steal) and [14](14-expansion-roadmap.md) (how the original author can expand ASHB2).
