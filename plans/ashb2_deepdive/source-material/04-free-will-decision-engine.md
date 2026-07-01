# 04 — The Free-Will Decision Engine

*Each tick, every entity turns a bag of drives, memories, personality knobs and social pressure into exactly one chosen action — by summing a pile of weighted modifiers, gambling on the total with a roulette wheel, and letting a reflex layer veto the whole thing. This is the crux subsystem, and its arbitration has no principled theory behind it.*

`FreeWillSystem` is where ASHB2's psychology (see `03-entity-and-psychology.md`) becomes behaviour, and where behaviour becomes civilization (`07-civilization-engine.md`). It is the largest hand-written subsystem in the repo: `src/implem_free_will.cpp` is **4030 lines**, `src/header/FreeWillSystem.h` **345 lines**. One `FreeWillSystem` instance is owned per `Entity` (it holds that agent's Q-table and habit list), and it is driven once per living entity per tick from the main loop (`src/main.cpp:1187`).

This document traces the actual execution path, enumerates the real action set and scoring terms, and then argues — as the author himself does in `question.md` Gap 1 — that the arbitration is the engine's single biggest weakness.

---

## 1. By the numbers

| Quantity | Value | Source |
|---|---|---|
| LoC (`implem_free_will.cpp`) | 4030 | `wc -l` |
| Registered action types | **~66** (68 `Action` objects, 2 commented out: `Anxiety`, `Exercise`) | `initializeActions()` `:667-1198` |
| `availableActions.push_back` sites | 70 (2 are duplicate re-pushes of `CreativeActivity`) | `:667-1198` |
| Additive scoring terms (weighted sum) | **6** | `:1631-1632` |
| Multiplicative modifiers on top | **8** | `:1634-1641` |
| Post-hoc ad-hoc boosts/knobs | ~9 (rarity, jitter, self-concept, RL-Q, social-decay, loneliness, desire, anger, crisis) | `:1648-1824` |
| `rarityMultiplier` assignments | ~78 (a ~55-entry hand-coded per-action table) | `:1650-1754` |
| Float literals (magic-number proxy) | **1742** | `grep -oE '[0-9]+\.[0-9]+f'` |
| Reflex survival gates | 4 (danger, starvation, exhaustion, collapse) | `reflexLayer()` `:1213-1287` |
| Side-drive injectors | 3 (romantic, hostile, side-social) | `main.cpp:1280-1324` |

Note the additive weights `0.20 + 0.25 + 0.10 + 0.10 + 0.15 + 0.15 = 0.95` — they don't even sum to 1.0. That is the tell: these are dials someone turned until the demographics "looked right," not a normalized model.

---

## 2. The three entry points (and which one actually runs)

`chooseAction()` (`:1555`) is the top-level call. It tries three strategies in strict order:

```cpp
if (Action* reflex = reflexLayer(entity, neighbors, context)) return reflex;   // 1. subsumption veto
Action* cp = cognitiveChooseAction(entity, neighbors, context);                // 2. "cognitive" pipeline
if (cp != nullptr) return cp;
Action* habitualAction = checkHabitTrigger(context);                           // 3. legacy fallback
... // giant weighted-sum loop
```

The important, under-documented fact: **`cognitiveChooseAction()` almost always returns non-null**, so the entire legacy path below it — including the habit layer and the *even larger* weighted-sum loop at `:1594-1892` — is **dead code in practice**. The two scoring loops (`:1594` legacy and `:3524` cognitive) are ~90% duplicated but have *drifted*: `socialInfluence` is weighted `0.15` in one and `0.19` in the other, and the two per-action rarity tables disagree (e.g. `Betray` is `0.15` legacy vs `0.25` cognitive). This is a genuine maintenance hazard — the "real" arbitration is the cognitive loop, and the legacy loop is a decaying fossil that only fires if cognitive produces zero candidates.

The rest of this document therefore documents `cognitiveChooseAction()` as the live path, and treats the legacy loop as its near-twin.

---

## 3. The reflex layer (the one principled piece)

`reflexLayer()` (`:1213-1287`) is explicit Brooks-style subsumption: a fast reactive layer that runs *before* any deliberation and can seize control from every higher system. Four hard-thresholded gates, in priority order:

1. **Imminent danger** — `context.env.safetyLevel < 22` OR a neighbor with an anger-link at this entity > 55. Fight-or-flight: a healthy (`health>55`), non-neurotic, angry-or-accompanied agent returns `DefendTribe`/`Duel`; otherwise `Flee`.
2. **Starvation** — `entityHunger >= 70`: eat from `foodStore`, else `Hunt`/`Farm`/`Gather` (best source the agent's health can still work).
3. **Total exhaustion** — `fatigueLevel >= 92` → `Sleep`.
4. **Health collapse** — `entityHealth < 12` → `Rest`.

When any fires it bumps `reflexOverrideCount`, writes `lastReflexReason` + the entity's `innerMonologue`, and **deliberation is skipped entirely**. This is the only place in the engine where one system genuinely *vetoes* the others rather than merely out-weighting them — and, tellingly, it is the part the author is happiest with.

---

## 4. The scoring pipeline (weighted sum → roulette)

For every candidate action, `cognitiveChooseAction()` (`:3524-3631`) builds a scalar `score`. The structure is: **additive core, then multiplicative modifiers, then a cascade of ad-hoc knobs.**

### The additive core (6 terms, `:3544`)

```cpp
float score = requirementFitness * 0.20f + needSatisfaction * 0.25f + memoryBias * 0.10f +
              varietyBonus * 0.10f + socialInfluence * 0.19f;   // + lifeMemoryBias*0.15 in legacy
```

| Term | Fn (`:line`) | Where it comes from |
|---|---|---|
| `requirementFitness` | `calculateRequirementFitness` `:50` | How well current stats meet the action's `StatRequirement`s. Splits stats into *excess* (stress/anger — fires when high) vs *deficiency* (health/hygiene — fires when low). |
| `needSatisfaction` | `calculateNeedSatisfaction` `:106` | `baseSatisfaction` scaled by a Maslow `hierarchyMultiplier`. **Reads `entity->needs`, which `03` shows is largely un-initialized** — so this highest-weighted term is fragile/partly vestigial for non-social actions. |
| `memoryBias` | `calculateMemoryBias` `:217` | Recency-weighted average `outcomeSuccess` of the last 50 executions of this action (recent ×1.5). A crude Rescorla-Wagner-ish reinforcement signal. |
| `varietyBonus` | `calculateVarietyBonus` `:242` | `pow(0.8, recentCount)` — anti-repetition; social/romance actions decay slower (`0.92`). |
| `socialInfluence` | `calculateSocialInfluence` `:1352` | Neighbors' anger/happiness/stress + relationship weights nudge matching action classes. This is "social pressure" — but it is just *another additive term*, not a controller. |
| `lifeMemoryBias` | `calculateLifeMemoryBias` `:191` | Exp-decayed autobiographical memories (loss, trauma, first-love) bias whole categories (see `05-cognition-memory-planning.md`). Legacy loop only. |

### The multiplicative modifiers (8 terms, `:3546-3552`)

`score` is then multiplied by, in order: `contextualWeight` (`:257` — time-of-day/public/work gating), `personalityModifier` (`:564` — Big Five + attachment style shaping, e.g. social ×`0.8→2.2` by extraversion), `valueSatisfaction` (`:527` — ValueSystem drives), `griefModifier` (`:420` — grief suppresses socializing, boosts substances/therapy), `pheromoneInfluence` (`:450` — season + neighbor pheromone type), `envModifier` (`:487` — crowd/noise/weather), `normModifier` (`:1518` — conformity vs rebellion against the group's prevailing action), and (legacy only) `semanticMemoryBias` (`:3695`).

### The ad-hoc cascade (`:3560-3626`)

After the "principled" part, a long tail of hand-tuning: the **`rarityMultiplier`** table (a ~55-line `if/else if` chain of per-action balance constants — `Murder ×0.02`, `Flirt ×3.2` when neighbors present, `Gossip ×3.7`…), a `selfConcept` nudge, a `0.93–1.07` random jitter, the **RL Q-value** (`score *= 0.6 + 0.8*(q/100)`, ±40% from the per-entity Q-table keyed on a ~4-char state signature like `"Ma_p"`, `:1539`), and urgency boosts for loneliness/desire/anger.

### Selection is roulette, not argmax (`:3657-3671`)

The top ≤7 candidates are kept, then one is drawn **proportional to score** (fitness-proportionate / roulette-wheel), not winner-take-all. A final flourish: if the entity has conflicting core values, there's a `conflictLevel` chance to **swap the top two candidates** (`:3673-3685`) — "chose against dominant preference."

```mermaid
flowchart TD
    T[Tick: chooseAction entity] --> R{reflexLayer<br/>4 survival gates}
    R -- fires --> RX[Override action<br/>Flee/Eat/Sleep/Defend] --> OUT[Chosen action]
    R -- passes --> C[cognitiveChooseAction]
    C --> PL[Planner: getPlannedAction<br/>Tree-of-Thoughts]
    subgraph SCORE[per candidate action]
      A1[requirementFitness ×0.20]
      A2[needSatisfaction ×0.25]
      A3[memoryBias ×0.10]
      A4[varietyBonus ×0.10]
      A5[socialInfluence ×0.19]
      SUM[+ = additive core]
      A1 --> SUM
      A2 --> SUM
      A3 --> SUM
      A4 --> SUM
      A5 --> SUM
      SUM --> MUL[× 8 modifiers<br/>context·personality·value·grief<br/>pheromone·env·norm·semantic]
      MUL --> ADH[× rarity × jitter × RL-Q<br/>× urgency boosts<br/>+1.5× if matches plan]
    end
    C --> SCORE
    PL -.1.5× soft nudge.-> ADH
    ADH --> ROU[Roulette-wheel select<br/>top ≤7 candidates]
    ROU --> OUT
    OUT --> SIDE[main.cpp side-drives:<br/>romantic 45% · hostile 40% · side-social]
    SIDE --> EXE[executeAction ×1–3]
```

---

## 5. Requirement / context gating (soft, not hard)

`Action` (`FreeWillSystem.h:46`) carries `requirements` (a vector of `StatRequirement{name, requiredValue, weight}`) and `statChanges`. Crucially, **requirements do not hard-gate** — `Murder` "requires" anger 80 / stress 70, but a calm entity still *scores* Murder; it's just suppressed by low `requirementFitness` × `rarityMultiplier 0.02`. The only true hard gates are: social actions with **empty `neighbors`** are skipped/zeroed (`:3528`, `:3571`), the reflex thresholds, and pointed-action target existence. Everything else is a soft weighting. This is why the author leans on the rarity table so heavily — it is doing the gating that the requirement system doesn't.

The **planner** (`getPlannedAction`, `:3732`) is the closest thing to a "deliberated plan": it pulls the next step from the entity's Tree-of-Thoughts planner (`05`). Its influence on the final choice is a **single `×1.5` multiplier** (`:3555`) — a soft nudge that any strong drive or rarity swing can outvote. There is no sense in which a plan is *committed to*.

---

## 6. Side-drives: actions the scorer never chose

The most surprising design choice lives in `main.cpp`, not the engine. After the chosen action executes, up to **three more actions are injected per tick without scoring**:

- **Side-social** (`main.cpp:1280`): `ChooseSpecificSocialAction()` fires alongside every pointed action.
- **Romantic side-drive** (`main.cpp:1293`): 45% chance → `TriggerDesireLinkedAction()` returns a *random* pick from a curated `desireLinkedAction` pool (Socialize, Desire, Flirt, Date, couple, breeding, Marry, Reconcile…), executed on the most-desired nearby mate, then `pointedAssimilation` builds the desire/couple link.
- **Hostile side-drive** (`main.cpp:1314`): gated on a real grievance (`anger>35` OR existing anger-links OR a low-agreeableness roll), then 40% → `TriggerHatredLinkedAction()` picks randomly from the `hatredLinkedAction` pool (AngerConnection, Murder, Discrimination). Murder is deliberately excluded from *driving* assimilation to avoid carnage.

So an entity can act 1–3 times per tick, and its romances and rivalries (`06-relationships-and-social-order.md`) are built largely by these unscored random draws — a pragmatic hack to stop the scorer's friendship bias from producing an all-platonic world. It works, but it means the "decision engine" is not actually the sole author of behaviour.

---

## 7. The full action set

Enumerated from `initializeActions()` (`:667-1198`), grouped by intent:

- **Baseline social / romance**: Socialize, Desire, GoodConnection, DigitalSocial, Gossip, Flirt, Date, couple, breeding, Marry, Reconcile, BreakUp, Apologize, HelpSupport, Celebrate, TellStory, Trade.
- **Hostile / antisocial**: AngerConnection, Insult, Manipulate, Jealousy, Betray, Discrimination, IgnoreAvoid, Murder, Duel, Raid, ChallengeLeader.
- **Self-care / coping / health**: EatMeal, Sleep, Rest, Take Shower, Prayer, SeekTherapy, SetBoundaries, Mourn, DrinkAlcohol, Smoke, Scrolling, Procrastinate, WatchEntertainment, Gaming.
- **Self-harm / collapse**: SelfHarm, Suicide, QuitGiveUp.
- **Achievement / cognition**: Read, LearnSkill, CreativeActivity, Work on Project, Basic Manual Work, Invent, Specialize, Explore, Build.
- **Survival / subsistence**: Hunt, Gather, Farm, Flee.
- **Civilization-scale** (feed `07`): LeadGroup, Preach, PerformRitual, TeachSkill, FulfillDuty, DeclareWar, Negotiate, DefendTribe.

`DeclareWar`/`ChallengeLeader`/`Preach`/`Invent`/`TeachSkill` are the hooks by which individual choices become tribes, religions, wars and tech (`07`).

---

## 8. The arbitration critique (central thesis)

The author's own framing, from `question.md` Gap 1: *"there's no principled theory of how a mind actually arbitrates between a drive, a habit, a deliberated plan, and social pressure."* The code confirms this precisely. Look at how each of those four is handled:

- **Drive** → additive terms (`requirementFitness`, `needSatisfaction`) + urgency boosts.
- **Habit** → a whole `Habit`/`checkHabitTrigger` subsystem (`:3251`) that **never runs** because it sits below the cognitive path.
- **Deliberated plan** → a `×1.5` multiplier.
- **Social pressure** → more additive/multiplicative terms (`socialInfluence`, `normModifier`).

All four are **flattened into one scalar and gambled on with a roulette wheel.** There is exactly one exception — the reflex layer — which is the only system that can *veto*. Everything above the reflex is a weighted sum with ~1742 hand-tuned constants and a ~55-entry rarity table doing the real balancing. Nothing decides *which system is in charge*; they all vote at once, additively, forever.

The author's intuition (Gap 1 answer) is the frontier direction: a **context-gated hierarchy** where one system can override the others, with an MBTI/Jungian cognitive-stack (the `JungianStack` already minted on every entity, `03`) selecting who's "in charge" for a given context — Brooks-style **subsumption generalized above the reflex line**. Concretely that means: environment sets the feasible set (hard gates), then a gate picks a *controller* (drive vs plan vs social vs habit) based on state + personality, and that controller mostly *wins* rather than merely adding a few points. The current engine already proves the veto pattern works (reflex); the open problem is extending it upward instead of collapsing everything into one number. That single change — the author is right — could restructure the whole decision core.

Three candid gaps worth naming:

1. **Two drifting copies of the scorer.** `chooseAction`'s legacy loop and `cognitiveChooseAction` duplicate ~90% of logic with divergent weights; only the cognitive one runs. Delete or unify.
2. **The highest-weighted additive term may be near-dead.** `needSatisfaction` (weight `0.25`) reads `entity->needs`, which `03` shows is largely un-populated at runtime — so for non-social actions the biggest dial can contribute ~0.
3. **Magic-number opacity.** ~55 per-action rarity constants + 1742 float literals, none calibrated. Rebalancing one action ripples unpredictably; there is no test asserting the emergent action distribution.

---

## Ideas worth stealing for petri-dish-of-madness

- **Deterministic action-scoring scaffold as a cheap prior beneath an LLM planner.** This weighted-sum engine is a fast, fully-deterministic "System 1" that always returns a plausible action for `<1µs`. Use it as a fallback/prior: let the LLM planner override when it has an opinion, but fall back to the scorer when the LLM is slow, budget-capped, or produces an illegal action. The scorer becomes a cheap safety net and a source of grounded candidate actions to feed the LLM.
- **The reflex/subsumption veto layer.** A hard, pre-LLM reactive layer (starving → eat, dying → flee) that can short-circuit the expensive planner entirely is exactly the right pattern for an agent sim: it guarantees survival-critical behaviour is instant and un-hallucinatable. Keep this even with an LLM in the loop.
- **Side-drives as unscored behavioural texture.** Injecting occasional random-but-pool-constrained romantic/hostile acts *outside* the main decision loop is a clever cheap trick to defeat a scorer's systematic biases (here: all-platonic worlds). An LLM sim could use the same idea — periodic "impulse" draws from curated pools — to add lifelike noise without paying for a full deliberation.
- **A discretized state-signature Q-table for near-free per-agent learning.** `rlStateSignature` compresses an agent's situation to a ~4-char bucket string, keeping a per-entity Q-table tiny while still letting agents learn context-dependent preferences. A dirt-cheap memory/learning layer to run *alongside* an LLM that has no persistent weights.
