# 13 — Frontier Assessment

*What in ASHB2 is genuinely novel, what is table stakes, and — for petri-dish-of-madness specifically — a ranked backlog of what's worth stealing, with effort and impact called out.*

## What's genuinely novel in ASHB2

Not the individual parts — needs, Big Five, tribes, and tech-trees all exist elsewhere — but three combinations that are rare-to-unique:

1. **Deterministic personality-physics *at civilization scale*.** ASHB2 runs a psychologically-detailed agent model for *thousands* of agents with **zero** stochastic external calls, and lets macro-history fall out of the trait distribution. Most psychology-rich sims (The Sims) don't scale to civilizations; most civ sims (Civ, most ABMs) don't model individual Big-Five psychology. Doing both at once, for free, is the novel move.
2. **Geography-seeded divergence with a `DivergenceConfig` dial.** A single master seed produces a planet whose *isolated regions* diverge into different religions, tech orders, and languages — and a tiny knob-set (`butterfly`/`innovationLuck`/`catastropheRate`/`migrationPressure`, [08](08-world-and-environment.md)) tunes *how wild* the run is, with a hashable "history signature" to prove runs actually diverged. That's a clean, verifiable answer to "make it different every time."
3. **The log→AI post-mortem chronicle** ([10](10-infrastructure-and-postmortem.md)). ASHB2 treats its event log as a first-class product and refines it into a narrated history via a "data historian" prompt. This is the most *portable* idea in the codebase and — notably — petri-dish reinvented it independently (EM-094), which is strong evidence it's a real primitive of the genre.

## What's table stakes

Present in ASHB2 and petri-dish both, and in most of the field — necessary, not differentiating: continuous needs/drives; proximity-based relationships with decay; kinship/lineage/inheritance and life-stages; a reflex/reactive fast path; factions/tribes; a memory of past events. If petri-dish is choosing what to take from ASHB2, these are **not** the reasons — it already has them.

## What NOT to take

Being honest saves petri-dish from importing ASHB2's mistakes:

- **The weighted-sum arbitration as-is.** ASHB2's decision core sums hand-tuned modifiers with no principled theory ([04](04-free-will-decision-engine.md)); its own author flags this in `question.md`. Take the *scaffold* (scored candidates + reflex veto), not the magic weights.
- **The uncapped jealousy loop.** ASHB2's relationship system produces a **murder epidemic** — ~89% of deaths are crimes of passion, median death age 15 ([06](06-relationships-and-social-order.md)). Emergent drama, yes; realistic society, no. Take the mechanism *with governors* (aggression caps, cooldowns, non-violent outlets).
- **The 1-ply "Tree-of-Thoughts."** It's a template selector, not search ([05](05-cognition-memory-planning.md)); petri-dish's LLM planning already exceeds it.
- **The "write it, don't wire it" habit.** ASHB2's defining flaw is a large *aspirational* architecture that never runs ([02](02-architecture.md)). petri-dish's contract/wave discipline is the antidote — don't regress it.

## Ranked backlog for petri-dish

Ordered by (impact ÷ effort). "Where" points at the petri-dish surface to change; "from" points at the ASHB2 reference.

| Rank | Steal | Impact | Effort | Where in petri-dish | From ASHB2 |
|-----:|-------|:------:|:------:|---------------------|------------|
| 1 | **Log-schema-as-contract → richer AI chronicle** | High | **Low** | `chronicle/ChronicleView.tsx`, `POST /api/chronicle/build`; formalize the event-log schema (`contracts/event-log.md`) | [10](10-infrastructure-and-postmortem.md): `AI_SUMMARY_PROMPT.md` 16-section historian + 6-tier death dedup |
| 2 | **Numeric Big-Five + attachment vector** | High | Low–Med | `config/personas.yaml`, `agents/runtime.py` context assembly | [03](03-entity-and-psychology.md): OCEAN + attachment minting |
| 3 | **Scorer-as-Tier-1 (zero-LLM brain)** | High | Med | beneath `providers/router.py` / `engine/loop.py` reflex path | [04](04-free-will-decision-engine.md): `FreeWillSystem` scoring + reflex veto |
| 4 | **Appraisal→emotion table** | Med–High | Low | derive a `mood`/emotion label from event appraisal before prompting | [03](03-entity-and-psychology.md): OCC-style `generateEmotion` |
| 5 | **Tech-tree for the economy** | High | Med | `engine/world.py` economy/skills; add prereq-gated nodes | [07](07-civilization-engine.md): 14-node tree + 31-item innovation catalog |
| 6 | **Carrying-capacity → collapse loop** | High | Med | tie population to a food/resource cap in `engine/world.py` | [07](07-civilization-engine.md)/[08](08-world-and-environment.md): Malthus + `loseTechnology` |
| 7 | **Procedural planet (biomes + regions)** | High | **High** | under the `CityGenerator`; new geography layer for multi-city (EM-109/117) | [08](08-world-and-environment.md): `world/{Noise,Planet}` fBm + flood-fill |
| 8 | **Per-region language drift (Lexicon)** | Med | Med | name/culture generation per city | [08](08-world-and-environment.md): `world/Lexicon` phoneme drift + creolization |
| 9 | **Per-agent Q-table learning** | Med | Med | `engine/world.py` skills; add a persisted value table | [05](05-cognition-memory-planning.md): `rlStateSignature` Q-table (but **persist** it) |
| 10 | **Kübler-Ross grief arc** | Med | Low | agent affect on bereavement events | [03](03-entity-and-psychology.md): decaying multi-stage grief |
| 11 | **`DivergenceConfig` wildness dial + history signature** | Med | Low | run config + a run-fingerprint in the chronicle | [08](08-world-and-environment.md): butterfly/catastrophe knobs + hash |

**Reading the table:** rows 1–4 are the high-leverage, low-risk wins — they upgrade petri-dish's *existing* surfaces (chronicle, persona, reflex path, mood) with proven, cheap mechanisms and don't fight its LLM-native design. Rows 5–8 are the substrate petri-dish's *planned* Wave O / multi-city work still needs; ASHB2 is a working reference implementation for exactly those. Rows 9–11 are polish.

## The single highest-value move

If only one thing ships: **#1 — formalize the event log as a contract and feed it to the chronicle the way ASHB2 does.** petri-dish is already LLM-native and already has a chronicle builder, so this is nearly free, and it's the feature that makes every *other* emergent system legible and shareable. ASHB2's `AI_SUMMARY_PROMPT.md` (a 16-section "data historian" spec) and its 6-tier death-dedup priority are a ready-made blueprint. It's the clearest example in this whole deep-dive of a low-effort, high-impact steal.

## A note on scope

This backlog is a **static report** (per the deep-dive's chosen output — no living-plan intake). If petri-dish's `BUILD-PLAN.md` should track these as real work items later, run the `plan-intake` skill on this document to convert rows 1–11 into ledger entries in petri-dish's own format. For ASHB2's *own* forward plan (the other audience for this deep-dive), see [14-expansion-roadmap.md](14-expansion-roadmap.md).
