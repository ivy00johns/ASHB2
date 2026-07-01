# 14 — Expansion Roadmap (for the ASHB2 author)

> **STATUS — PROPOSED.** This is a forward plan written *for the original developer*, in the
> phased style of your own `plans/alternate-earth.md`. It is grounded in a full read of the
> live code (docs [03](03-entity-and-psychology.md)–[10](10-infrastructure-and-postmortem.md)) and in your own gap analysis in `question.md`.
> Nothing here is built yet. Phases are ordered so each one compiles, runs, and is a safe
> commit point on its own.

**The one thing to internalize first:** *you have already written far more than you run.* The
single most consistent finding across every subsystem is a large, well-engineered **aspirational
architecture that never executes** ([02](02-architecture.md) dead-code inventory). Your highest-leverage,
lowest-risk expansion is not new code — it's **wiring (or deleting) the code you already wrote.**
That theme drives Phase 0, and it recurs throughout.

**Build context (verified):** CMake + MinGW, C++17, OpenMP optional. New `.cpp` → the
`add_executable(app …)` list in `CMakeLists.txt`; new source dirs → `include_directories(...)`.
No new external deps needed for any phase below.

**Guiding principle:** each phase leaves the sim in a runnable, committable state. Front-load the
work that *unlocks capability you already paid for* (Phase 0) and the work that fixes your own
named #1 weakness (Phase 1).

---

## Phase 0 — Triage the dead code: wire it or delete it

**Why first:** Every dormant subsystem is either free capability you're not using or dead weight
that makes the codebase lie about what it does. Deciding *wire vs. delete* for each, one commit at
a time, is pure upside and shrinks the aspirational/real gap that otherwise keeps growing.

**The dormant inventory (confirmed):**

| Subsystem | Files | Recommendation |
|-----------|-------|----------------|
| `EmotionalComplexitySystem` (OCC appraisal→emotion, mood, contagion) | `EmotionalComplexity.cpp` (603) | **WIRE** — it's your real emotion model (Phase 2) |
| Maslow `HierarchicalNeed` ladder | `Entity.h` | WIRE or delete (needs already work via drives) |
| `SocialDynamics` (Sternberg triangular-theory, groups, gossip, reputation) | `SocialDynamics.cpp` (405/624 w/ header) | **WIRE** — richer relationships than the 4 scalars ([06](06-relationships-and-social-order.md)) |
| `LifeCourseSystem` + `AgingEffects` curves | `LifeCourse.cpp` (582) | WIRE the aging curves (Phase 2); delete the rest |
| `CognitiveArchitecture`, `PersonaSystem` | `.cpp`/`.h` | delete unless Phase 1 uses them |
| ~85% of `LearningAdaptation` (habits, skills, cultural transmission) | `LearningAdaptation.cpp` (517) | keep the Q-table; delete or wire the rest |
| Force-directed movement + `Movement` class | `movement.cpp` (125), `Movement.h` | **DECIDE** — wire for spatial realism, or delete and own "motion = clustering + migration" ([09](09-simulation-loop-and-rendering.md)) |
| `SpatialMesh` quadtree / `getCloseEntityGroups` | `SpatialMesh.cpp` (188) | wire if you wire movement; else delete |
| SDL backend (mode 2, commented out) | `SDLEngine.cpp` (117) | delete unless you actually want the second UI |
| `CulturalTransmissionSystem` / `WorldEnvironment` | `environment/EnvironmentModel.cpp` (700) | WIRE transmission (drives culture drift); you already salvaged `SeasonalConfig` |
| `checkSchisms` (declared, never defined); `loseTechnology` entity-strip (`(void)lostId`) | `CivilizationEngine.cpp` | implement (Phase 4) |

**Verify:** after each commit, `grep -rn "ClassName\b" src/` shows a real call-site, or the files
are gone. The build stays green. Nothing in `src/` is compiled-but-never-constructed.

**Anti-patterns:** don't "keep it just in case" — dead code that looks live is worse than no code.
If you're not going to wire it this quarter, delete it; git remembers.

---

## Phase 1 — Give the decision core a real arbitration theory (your Gap 1)

**Why:** This is *your own* #1 named weakness. `FreeWillSystem` currently arbitrates by summing
`calculateNeedSatisfaction + calculateMemoryBias + calculatePersonalityModifier +
calculateGriefModifier + Q-value + calculateSemanticMemoryBias(:3695) + social influence + habit`
with hand-tuned weights, plus a reflex layer on top ([04](04-free-will-decision-engine.md)). A weighted sum has no theory
of *which system is in charge* — and your own answer in `question.md` is right: it should be a
**context-gated hierarchy where one system can veto the others.**

**Implement** — a two-stage arbiter, not a flat sum:
- **Stage A — reflexes/subsumption (already exists):** keep the hard survival vetoes (starving→eat,
  dying→flee). Formalize them as the top of a subsumption stack that short-circuits everything below.
- **Stage B — context gate picks the "governing" system:** before scoring, decide *what kind of
  moment* this is (crisis / social / deliberative / habitual) and let that gate **which term
  dominates** — e.g. crisis → drives win; social scene → relationship+norm win; calm → planner wins;
  low-salience → habit/Q-table wins. Implement the gate as a small function of visceral state +
  salience + your **JungianStack** (which is already computed but currently only flavors the inner
  monologue — `JungianType.cpp` — so this *also* wires a dead system). The cognitive stack is a
  natural "who's driving" selector.
- Keep the weighted sum *inside* the winning context as the tie-breaker among that context's actions.

**Where:** `implem_free_will.cpp` scoring entry (wrap the summation), `FreeWillSystem.h`;
`JungianType.cpp` for the stack→context mapping.

**Verify:** log the winning context each tick. The same entity in a famine vs. at a dance should
show *different governing systems*, not just different sums. Ablate the gate → behavior collapses
back to today's sum (proves the gate is doing work).

**Anti-patterns:** don't just add "context" as one more weighted term — that's still a sum. The gate
must be able to **override**, i.e. zero out competing terms.

---

## Phase 2 — Calibrate the psychology with real numbers (Gaps 2–5)

**Why:** Your defaults are `N(50,20)` independent draws and guessed decay constants; the appraisal
rule is a hand-coded if/else emitting only **6 of 14** emotions; and the whole
`EmotionalComplexitySystem` that would do this properly is **dead** ([03](03-entity-and-psychology.md)). This is where
"everyone is bland and average" and "fast/slow states bleed together" come from.

**Implement:**
- **Wire `EmotionalComplexitySystem`** as the live affect path (replaces the coarse PAD-over-floats).
  Its `generateEmotion(relevance, desirability, coping, control, normCompatibility)` becomes the real
  appraisal→emotion table — extend it to the full Ekman/OCC set (Gap 4).
- **Timescale hierarchy (Gap 2):** make emotion (seconds), mood (~100 ticks), allostatic load (slow),
  and trait drift (rare) genuinely separate rates, not shared magic numbers. Wire the dead
  `AgingEffects`/time-horizon curves from `LifeCourse.cpp` for the slow end.
- **Trait correlations (Gap 3):** sample Big Five from a correlated multivariate draw (real facet
  correlations) and a non-uniform population distribution instead of five independent `N(50,20)`.
- **Body→mind coupling (Gap 5):** make visceral state (hunger/fatigue/stress) bias the Phase-1
  context gate and time-discounting, via an explicit somatic-marker term.
- **Fix the formative-memory bug:** there are two rival `updatePersonalityFromExperience`
  implementations; only the inline one (`:1289`) runs and `isFormative` never triggers a distinct
  mutation. Make formative events a *real*, rarer, larger trait shift than ordinary nudges.

**Where:** `EmotionalComplexity.cpp`, `Entity.cpp`/`Entity.h`, `LearningAdaptation.cpp:423`,
`implem_free_will.cpp:1289`.

**Verify:** a run's founder Big-Five scatter shows realistic correlation + spread (not a blob at 50).
Emotions cover the full set. A repeated trauma shifts a trait; a passing annoyance does not.

**Anti-patterns:** don't hand-tune new magic numbers — pull the correlations/rates from the
literature you already gesture at in `question.md`, and write them down as named constants with
citations.

---

## Phase 3 — Tame the jealousy runaway (balance)

**Why:** The relationship system is a **murder epidemic**: uncapped anger + 22px couple orbits +
packed geography + no cooldown → ~89% of deaths are crimes of passion, median death age 15–16
([06](06-relationships-and-social-order.md)). The ingredients are realistic; the absence of governors is not.

**Implement:**
- Cap and decay `anger`; add a per-entity kill cooldown; gate lethal outcomes on more than
  `anger≥70 && within 90px` (`implem_free_will.cpp:3780–4030`).
- Add **non-violent jealousy outlets** (withdrawal, breakup, rumor, rivalry) that fire far more often
  than murder.
- **Fix the incest gap:** the action-driven `breeding` branch (`:2385`) has no `wouldBeIncest` gate —
  only the auto-conception path (`:3959`) does. Route both through the same kinship check, and make
  `shareParent` handle unknown-parentage siblings.

**Verify:** re-run a standard seed; violent-death share drops from ~89% to a chosen target band and
median lifespan rises. Post-mortem ([10](10-infrastructure-and-postmortem.md)) shows a plausible cause-of-death mix. No
incestuous births in the log.

**Anti-patterns:** don't fix it by globally suppressing emotion — keep the drama, add the brakes.

---

## Phase 4 — Make the micro layer actually *cause* the macro layer

**Why:** Today the free-will actions `Preach / Invent / DeclareWar / ChallengeLeader / TeachSkill`
are **narrative tells, not triggers** — they mutate the actor's own emotions/logs, while
`CivilizationEngine` runs its *own* probabilistic rolls off the same traits ([07](07-civilization-engine.md)). So the
post-mortem's action counts *correlate with* but don't *cause* civ outcomes. That's a hidden
disconnect between the two layers your README sells as one causal stack.

**Implement:**
- Make individual actions write real civ events: a `Preach` that lands seeds or grows a religion; a
  `DeclareWar` that actually shifts `relations`; a `ChallengeLeader` that can depose.
- **Implement `checkSchisms`** (declared, never defined; `parentReligionId` is always −1) so religions
  actually fracture from doctrinal + personality divergence.
- **Make `loseTechnology` real** — its entity-strip is a no-op (`(void)lostId`); a dark age should
  genuinely remove low-`knowerCount` innovations *and* touch the structured tech tree.

**Where:** `CivilizationEngine.cpp` (`foundReligion`, `updateInnovations`, diplomacy), the action
handlers in `implem_free_will.cpp`.

**Verify:** disable the engine's autonomous rolls → civ still evolves *from actions alone* (proves
causation). A logged schism produces a `parentReligionId ≥ 0` child faith. A population crash shows
a measurable tech regression, then re-invention.

**Anti-patterns:** don't double-drive outcomes (both actions *and* autonomous rolls) — pick actions
as the cause and let the engine aggregate, or you'll never be able to reason about balance.

---

## Phase 5 — Validation & ground truth (your Gap 6)

**Why:** Nothing checks that emergent behavior is *realistic*. `ValidationFramework` (600 LoC) is a
stats toolkit with `// Placeholder`/`Dummy` fitness, and it's **unwired** anyway ([10](10-infrastructure-and-postmortem.md)). This
is the question almost nobody asks and where your combined design pays off.

**Implement:** wire the framework to check 3–4 population-level patterns you *didn't* code directly —
distribution of relationship lengths, share of ever-isolated agents, life-outcome spread, grief
resolution time — against target ranges from real demography/psychology. Emit a pass/fail panel and a
line in the post-mortem.

**Verify:** a deliberately broken run (e.g. Phase-3 disabled) fails the checks; a tuned run passes.
The checks run every N ticks without tanking FPS.

**Anti-patterns:** don't validate what you coded (that's circular) — validate *emergent* patterns you
never explicitly authored.

---

## Phase 6 — Reproducibility & scale

**Why:** Replay is real but **single-thread-only** (one shared global RNG + construction-order
counter), and learned Q-values are **discarded on save** ([05](05-cognition-memory-planning.md)/[10](10-infrastructure-and-postmortem.md)). Both cap how far the sim
can grow.

**Implement:** give each entity its own seeded sub-stream (derived from master + entity id) so
ordering/threading no longer affects results; serialize Q-values in `SaveLoad.cpp` (160 LoC); add a
**headless tick-and-dump harness** (run N generations with no window, emit the logs) so long runs and
CI don't need a desktop GLFW window.

**Verify:** same seed, different thread counts → identical history signature. Save/reload preserves
learned preferences. A headless run produces a valid post-mortem with no display.

**Anti-patterns:** don't reintroduce `random_device`/`time(0)` anywhere in sim logic.

---

## Phase 7 — New frontier (optional): a Tier-2 LLM in the loop

**Why:** You already have the *entire* offline LLM path — `AI_SUMMARY_PROMPT.md` +
`postmortem_generate.py` turn logs into narrative. The natural frontier is moving a *sliver* of that
capability **into** the loop: let the algorithmic scorer run everything for free, and call an LLM only
for the rare high-stakes/novel decision (a leader's speech, a schism's doctrine, a first-contact).
This is exactly the two-tier "System 1 + System 2" brain that the sister project
[petri-dish-of-madness] arrived at from the other side ([12](12-convergence-analysis.md)).

**Implement (spike):** add an optional `llmDecide(entity, context)` hook invoked only when the
scorer flags a tick as high-salience; feed it the entity's numeric psychology + retrieved memories;
have it return one of the scorer's *legal* candidate actions (grounded, not free-form). Off by
default; token-budgeted.

**Verify:** with the hook off, runs are byte-identical to Phase 6. With it on, flagged moments show
richer, legible reasoning without changing per-tick cost for the other 99% of ticks.

**Anti-patterns:** don't route every tick through an LLM — that throws away ASHB2's whole
advantage (free, deterministic, thousands of agents). The LLM is a garnish, not the engine.

---

## Suggested order & checkpoints

`Phase 0` (triage) unblocks everything and is pure cleanup — do it first and commit per subsystem.
Then `Phase 1` (your #1 gap) → `Phase 2` (make minds non-bland) → `Phase 3` (stop the murder
epidemic) are the "make it feel real" core. `Phase 4` (micro causes macro) makes the two-layer claim
*true*. `Phase 5–6` are rigor. `Phase 7` is the optional leap. Each phase is a safe commit point.

## Files touched (summary)

- **Wire (don't rewrite):** `EmotionalComplexity.cpp`, `SocialDynamics.cpp`, `JungianType.cpp`,
  `environment/EnvironmentModel.cpp` (CulturalTransmission), `validation/ValidationFramework.cpp`,
  aging curves in `LifeCourse.cpp`.
- **Edit:** `implem_free_will.cpp` (arbitration gate, jealousy governors, incest gate, action→civ
  triggers), `CivilizationEngine.cpp` (`checkSchisms`, `loseTechnology`), `Entity.cpp`/`Entity.h`
  (correlated traits, timescales), `SaveLoad.cpp` (persist Q-values), `WorldSeed.*` (per-entity
  streams), `main.cpp` (headless harness).
- **Delete (unless wired):** `CognitiveArchitecture.*`, `PersonaSystem.*`, `SDLEngine.cpp` (mode 2),
  dead portions of `LearningAdaptation.cpp`, and `movement.cpp`/`SpatialMesh.cpp` if you commit to
  "motion = clustering + migration."

> The throughline: ASHB2's ceiling isn't limited by ideas — you've had more than enough, and written
> most of them. It's limited by how much of what you wrote is actually *running, causal, and
> calibrated*. Every phase above converts latent code into live behavior.
