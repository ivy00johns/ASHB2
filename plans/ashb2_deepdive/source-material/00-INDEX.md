# ASHB2 — Deep Dive Index

*A structured technical reference on **ASHB2**, a from-scratch C++17 agent-based civilization simulator, produced to answer: how does it line up with **petri-dish-of-madness**, what has already converged, and what else can be learned — for two audiences at once.*

## Generation metadata

| Field | Value |
|-------|-------|
| Subject | `ASHB2` (`/Users/johns/Projects/ASHB2`) — C++17 "society in a box" |
| Reference project | `petri-dish-of-madness` (`/Users/johns/Projects/petri-dish-of-madness`) |
| Method | `repo-deep-dive` skill — parallel subagents reading live code + a research pass |
| Grounding docs | `README.md`, `plans/alternate-earth.md`, `question.md` (author gap analysis), two auto-generated post-mortems in `plans/` |
| Generated | 2026-07-01 |
| Series size | 16 documents (~2,150 lines) |
| Honesty note | Live code was the ground truth; where the plans/README oversold what runs, the code won — those discrepancies are flagged throughout. |

## Two audiences

This deep dive is deliberately useful to **two** readers:

- **The petri-dish-of-madness owner** — "how does ASHB2 line up, what's already integrated, what else can we learn." Served by [11-comparison-petri-dish.md](11-comparison-petri-dish.md), [12-convergence-analysis.md](12-convergence-analysis.md), [13-frontier-assessment.md](13-frontier-assessment.md).
- **The original ASHB2 developer** — who asked for ideas on how to expand it. Served by two companion docs: [14-expansion-roadmap.md](14-expansion-roadmap.md), a phased *implementation* plan in the author's own `alternate-earth.md` style (depth — finish the machine), and [15-expansion-ideas.md](15-expansion-ideas.md), an *idea-level* menu of new frontiers (scope — where it could go).

## The document series

| # | Document | Purpose | Lines |
|---|----------|---------|------:|
| 00 | **INDEX** (this file) | TOC, metadata, headline findings, reading guide | — |
| 01 | [project-overview](01-project-overview.md) | What ASHB2 is, by the numbers, landscape position | 57 |
| 02 | [architecture](02-architecture.md) | Micro→macro system map (mermaid), key decisions, the dead-code pattern | 110 |
| 03 | [entity-and-psychology](03-entity-and-psychology.md) | Big Five + attachment, needs/metabolism, emotion, grief | 180 |
| 04 | [free-will-decision-engine](04-free-will-decision-engine.md) | The 4k-LoC action-arbitration core (weighted sum + reflex veto + Q-table) | 186 |
| 05 | [cognition-memory-planning](05-cognition-memory-planning.md) | Semantic memory, the "Tree-of-Thoughts" planner, learning, life-course | 151 |
| 06 | [relationships-and-social-order](06-relationships-and-social-order.md) | Dyadic bonds, the jealousy→murder runaway, kinship, class | 158 |
| 07 | [civilization-engine](07-civilization-engine.md) | Tribes, religion, tech-tree, diplomacy, economy, collapse loop | 131 |
| 08 | [world-and-environment](08-world-and-environment.md) | Procedural planet (fBm biomes, regions, language drift), Malthus | 112 |
| 09 | [simulation-loop-and-rendering](09-simulation-loop-and-rendering.md) | The per-tick pipeline, dormant force-movement, dual (one live) render backends | 134 |
| 10 | [infrastructure-and-postmortem](10-infrastructure-and-postmortem.md) | Determinism, logging, persistence, and the log→AI chronicle pipeline | 171 |
| 11 | [comparison-petri-dish](11-comparison-petri-dish.md) | ASHB2 vs. petri-dish head-to-head + capability matrix | 72 |
| 12 | [convergence-analysis](12-convergence-analysis.md) | The two-tier "hybrid brain"; what each lacks | 61 |
| 13 | [frontier-assessment](13-frontier-assessment.md) | What's novel / table stakes; ranked "what to steal" backlog | 52 |
| 14 | [expansion-roadmap](14-expansion-roadmap.md) | **For the ASHB2 author** — a phased *implementation* plan (depth: finish the machine) | 254 |
| 15 | [expansion-ideas](15-expansion-ideas.md) | **For the ASHB2 author** — an *idea-level* menu of new frontiers (scope: where it could go) | 236 |

## ASHB2 by the numbers

| Metric | Value |
|--------|-------|
| Hand-written simulation code | **~28,900 LoC** (40,645 total in `src/` minus 11,775 vendored ImPlot) |
| Source files | 101 (49 `.cpp` + 52 `.h`) · ~40 named subsystems |
| Largest hand-written file | `implem_free_will.cpp` — 4,030 LoC |
| Agents per run | thousands (deterministic, zero API calls) |
| Recent-run signature | ~89% of deaths are jealousy "crimes of passion"; median death age ~15 |
| LLM touchpoints in-sim | **0** (only an offline post-mortem narrator) |

## Headline findings

1. **Ambitious-but-dead code is the defining trait.** A large, well-engineered architecture never runs: the `EmotionalComplexitySystem`, `SocialDynamics` (Sternberg theory), `LifeCourseSystem`, force-directed movement, the `SpatialMesh` quadtree, the SDL backend, and `CulturalTransmissionSystem` are all compiled-but-dormant. ASHB2's *aspirational* architecture is materially larger than its *live* one ([02](02-architecture.md)).
2. **The decision core is a weighted sum with no arbitration theory** — the author's own #1 named gap. It sums hand-tuned modifiers + a reflex veto; there's no principled account of which system is "in charge" ([04](04-free-will-decision-engine.md), `question.md`).
3. **The relationship system is a runaway.** Uncapped jealousy → a murder epidemic (~89% of deaths). Great emergent drama, poor realism — it needs governors ([06](06-relationships-and-social-order.md)).
4. **Micro doesn't causally drive macro (yet).** Individual actions like `Preach`/`DeclareWar` are narrative tells; the civ engine rolls its own probabilities off the same traits, so they correlate but don't cause ([07](07-civilization-engine.md)).
5. **The log→AI post-mortem chronicle is the most portable idea** — and petri-dish reinvented it independently, evidence it's a real primitive of the genre ([10](10-infrastructure-and-postmortem.md), [13](13-frontier-assessment.md)).
6. **ASHB2 and petri-dish converged, they didn't copy.** Shared machinery (memory, planning, crime, lineage, chronicle) arrived via the generative-agents lineage, not a port. So ASHB2's genuine value to petri-dish is (a) a proven reference for petri-dish's *unshipped* macro/world plans and (b) a few substrates petri-dish lacks entirely: a zero-LLM brain, structured psychology, a procedural planet, a tech-tree, and a collapse-driven history arc ([11](11-comparison-petri-dish.md)–[13](13-frontier-assessment.md)).

## Reading guide

- **Just want the strategic answer** ("what can we learn"): read [11](11-comparison-petri-dish.md) → [13](13-frontier-assessment.md). The ranked backlog is the table in [13](13-frontier-assessment.md).
- **Passing this back to the ASHB2 author:** [01](01-project-overview.md) for orientation, then [14-expansion-roadmap.md](14-expansion-roadmap.md) (what to *finish*) and [15-expansion-ideas.md](15-expansion-ideas.md) (where to *go next*).
- **Want to understand the machine:** [02](02-architecture.md) for the map, then the micro stack [03](03-entity-and-psychology.md)→[06](06-relationships-and-social-order.md), the macro/world stack [07](07-civilization-engine.md)→[08](08-world-and-environment.md), and runtime/infra [09](09-simulation-loop-and-rendering.md)→[10](10-infrastructure-and-postmortem.md).

> **Scope note:** this is a **static** reference series (output target: `ASHB2/plans/ashb2_deepdive/`, no living-plan intake). To turn [13](13-frontier-assessment.md)'s backlog into tracked work items in petri-dish's `BUILD-PLAN.md`, run the `plan-intake` skill on that document later.
