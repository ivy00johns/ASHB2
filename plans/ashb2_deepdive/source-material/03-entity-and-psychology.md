# 03 — Entity & Psychology

*The micro-layer of state: what one simulated person **is**, how their body ticks toward starvation, and how a hand-wired psychology tries (and partly fails) to turn appraisals into feelings.*

Everything in ASHB2 — tribes, religions, wars — is an emergent statistic over thousands of `Entity` objects. This document traces that object: the fields it carries, how its needs tick and couple to survival, how personality and attachment are minted at birth, and the timescale hierarchy that is *supposed* to separate a passing mood from a lifelong trait. Read alongside `04-free-will-decision-engine.md` (how these states become actions), `05-cognition-memory-planning.md` (life memories → beliefs), and `08-world-and-environment.md` (where food comes from).

---

## 1. What the Entity holds

`Entity` (`src/header/Entity.h:244`) is a wide, flat struct — no inheritance, ~90 public fields. It bundles several loosely-coupled sub-systems:

- **Scalar needs/stats** (the "legacy unipolar" floats, `Entity.h:248-327`): `entityHealth`, `entityHapiness`, `entityStress`, `entityMentalHealth`, `entityLoneliness`, `entityBoredom`, `entityGeneralAnger`, `entityHygiene`, `entityHunger`, `fatigueLevel`. Each is a `[0,100]` float that only ever climbs toward deprivation; only the high pole bites.
- **Subsistence**: `entityHunger` (0 sated → 100 starving) and `foodStore` (`Entity.h:327-328`) — days of rations. This is the eat-or-die loop.
- **Personality** (`struct Personality`, `Entity.h:134`): the Big Five — extraversion, agreeableness, conscientiousness, neuroticism, openness. Default ctor sets **all five to `50.0f`** (`Entity.h:141`).
- **`ValueSystem`** (`Entity.h:151`): five "soul" values — familyOrientation, achievementDrive, spiritualNeed, hedonism, collectivism (all default `50.0f`).
- **`DevelopmentalHistory dv`**: childhood trauma/nurture scores + `AttachmentStyle` (SECURE / ANXIOUS / AVOIDANT / DISORGANIZED, `Entity.h:50`).
- **`DriveSet drives`** (`Entity.h:296`, see §3): the *bipolar* homeostatic model that layers a lethal floor + allostatic load + habituation on top of the legacy floats.
- **`JungianStack cognition`**: a Beebe 8-function stack derived from the Big Five — the "cognitive spine" the author intends as the arbitration mechanism (see `04`).
- **Relationships**: parallel vectors `list_entityPointedDesire/Anger/Couple/Social`, plus `reputationMap`, `list_MentalModelOfOther` (see `06`).
- **Memory & cognition**: `lifeMemories`, `semanticMemory`, `planner`, `coreBeliefs`, `workingMemory`, `pad` (PAD affect), `bodyLanguage` (see `05`).
- **Identity/geography**: `tribeId`, `religionId`, `familyId`, `socialClass`, `auctoritas`, `posX/posY`.

The struct is a bag of subsystems assembled over many phases (the `// Phase 1-5` comments at `Entity.h:453`), and — as we'll see — several of them are wired to nothing.

---

## 2. How needs tick and couple to survival

The real per-tick metabolism lives **not** in `Entity.cpp` but in the main loop, `src/main.cpp:955-1090`. The eat-or-starve core (`main.cpp:963-987`):

```cpp
float burn = 0.08f * deltaTime * coldFactor;      // calories/tick, colder = more
if (entity->foodStore > burn) {                    // fed: draw down the larder
    entity->foodStore -= burn;
    entity->entityHunger = clamp(entity->entityHunger - 0.8f, 0, 100);
} else {                                            // larder empty: hunger climbs
    entity->foodStore = 0.0f;
    entity->entityHunger = clamp(entity->entityHunger + 0.45f * coldFactor, 0, 100);
}
if (entity->entityHunger > 80.0f) {                 // past 80 it eats health
    float starve = (entity->entityHunger - 80.0f) / 20.0f;
    entity->entityHealth -= starve * 0.18f;         // grind to death over many ticks
}
```

Food is **produced** by hunting/gathering/farming actions in the free-will layer (`implem_free_will.cpp:2899-2947`) and **consumed** here — a genuine closed loop tying psychology to `08`'s harvest/season system. Parallel cascades follow the same shape: fatigue >80 frays mind and lightly chips health (`main.cpp:991-1000`); hygiene <25 raises stress; loneliness >80 erodes mental health; boredom >60/85 saps happiness then mental health. Every threshold and coefficient (`0.08f`, `0.45f`, `0.18f`, the `>80` cliff) is a **hand-tuned magic number** — there is no calibration against real metabolic or affective data.

A **second, richer needs model exists but is dead code.** `initializeHierarchicalNeeds()` (`Entity.cpp:604`) builds a proper Maslow ladder of 11 `HierarchicalNeed`s with per-tier decay rates (PHYSIOLOGICAL 0.25 → SELF_ACTUALIZATION 0.02). It is read by `PlanningSystem.cpp`, but **is never called from the spawn or tick path I could trace** — so `entity->needs` is empty at runtime and the Maslow layer is vestigial.

---

## 3. Personality & attachment: generated, and mostly independent

An entity is *minted* in the spawn block (`main.cpp:1808-1900`), not in a constructor:

1. **Big Five** — `generateRandomPersonality()` (`main.cpp:170`) draws each of the five traits **independently** from `N(50, 20)`, clamped to `[0,100]`:
   ```cpp
   std::normal_distribution<float> dist(50.0f, 20.0f);
   return Personality(clamp(dist(gen)), clamp(dist(gen)), ...);  // 5 independent rolls
   ```
   This is exactly **Gap 3** in `question.md`: real Big Five facets correlate (e.g. neuroticism ↔ low conscientiousness), but here they are five orthogonal Gaussians. The spread (σ=20) avoids everyone being *literally* 50, but there is no covariance structure and the population mean is pinned at bland-average 50.
2. **Values** *are* correlated with personality (`main.cpp:1823-1827`): `familyOrientation = N(50,18) + 0.3·(agreeableness−50)`, hedonism keys off extraversion minus conscientiousness, etc. A nice exception to Gap 3 — but only for the value layer.
3. **Childhood → attachment** (`main.cpp:1830-1844`): a `traumaRoll`/`nurtureRoll` pair deterministically selects an `AttachmentStyle`, then **permanently shifts** four traits (`main.cpp:1847-1850`): trauma pushes neuroticism up, agreeableness/extraversion down; nurture pushes openness up. This injects *some* structured covariance after the independent draw.
4. **Starting stats** (`main.cpp:1859-1879`) are personality-tinted: neuroticism→stress, conscientiousness→hygiene, extraversion→−loneliness. "No two people start at zero."

Attachment style, once set, **never changes** — there is no adult attachment revision anywhere. It only *modulates grief recovery* (`Entity.cpp:314-318`): avoidant recovers 2× faster, anxious 2× slower.

Note a subtle bug-shaped smell: the default `Personality()` ctor and dozens of sub-structs (`SelfConcept`, `MoodState`, `EmotionalRegulation`, `AffectiveStyle`, `PerceivedReputation`) all default to `50.0f`. Any entity created **outside** the spawn block (e.g. via the bare `Entity(int id)` ctor) is a perfectly bland 50-across person until something overwrites it.

---

## 4. The timescale hierarchy (and where it leaks)

The author's intended hierarchy — **Gap 2** — is emotion (seconds) → mood (~100 ticks) → allostatic load (slow) → personality drift (rare) → fixed-for-life. Here is what the code *actually* wires:

```mermaid
flowchart TD
    subgraph FAST["seconds — EmotionalEpisode (UNWIRED)"]
      E["generateEmotion: appraisal→Ekman emotion<br/>peak 3-10 ticks, offset 15-60"]
    end
    subgraph MOOD["~100 ticks — MoodState (UNWIRED)"]
      M["regress to baseline 0.98-0.99/tick<br/>reset at maxDuration=100"]
    end
    subgraph AFFECT["per-tick — PAD (WIRED)"]
      P["computePAD(happiness,stress,anger,...)<br/>→ bodyLanguage"]
    end
    subgraph SLOW["slow — Drive load/tolerance (WIRED)"]
      L["allostatic load ±0.01-0.02/tick<br/>habituation tolGain 0.015-0.02"]
    end
    subgraph DRIFT["rare — personality drift (WIRED)"]
      D["updatePersonalityFromExperience<br/>only when |reward|>70, changeRate 0.01"]
    end
    subgraph FIXED["fixed for life"]
      F["Big Five baseline · attachment style · sex"]
    end
    E --> M --> AFFECT
    AFFECT --> SLOW --> DRIFT --> FIXED
    L -. mindLoad ·0.05 .-> AFFECT
    D -. clamps traits .-> FIXED
```

Reading the layers bottom-up:

- **Emotion (seconds)** — `EmotionalEpisode` has a proper onset→peak→decay lifecycle (`EmotionalComplexity.cpp:9-32`), peaks 3-10 ticks, offsets 20-60. **But the entire `EmotionalComplexitySystem` is never instantiated** — grep finds zero callers of `generateEmotion`, `updateEmotions`, or `updateMood` outside the class itself. The fast/mood layers are dead.
- **Mood (~100 ticks)** — `MoodState::update` (`EmotionalComplexity.cpp:40`) regresses toward baseline (`×0.99 + 50×0.01`) and hard-resets at `maxDuration=100`. Also dead (lives inside the unwired system).
- **What actually runs as "affect"** is `PAD` — `updatePAD()` (`Entity.cpp:796`) recomputes pleasure/arousal/dominance *every tick* directly from the legacy floats, then derives `bodyLanguage`. It is stateless-per-tick, so there is **no genuine emotion→mood→affect timescale separation in the live sim** — exactly the "fast/slow states bleed into each other" that Gap 2 names.
- **Allostatic (slow)** — the one place slow accumulation is real: `Drive.load` accrues `loadGain 0.02`/tick in the danger zone and sheds `loadRelief 0.01`/tick in comfort (`Drive.h:69-70`); `tolerance` (habituation) grows/decays on novelty axes. Ticked every frame via `dr.tick()` at `main.cpp:1056`, feeding a hedonic-treadmill happiness drag (`main.cpp:1070`).
- **Drift (rare)** — `updatePersonalityFromExperience` (`LearningAdaptation.cpp:423`) is a keyword table: a positive "social" outcome adds `+2.0` extraversion, "trauma" adds `+3.0` neuroticism, etc. It only fires when `|reward| > 70` (`LearningAdaptation.cpp:305`), accumulates into a `PersonalityChangeTracker`, and applies at `changeRate 0.01` (`LearningAdaptation.cpp:234`). Big one-off events also poke traits directly (child death → `neuroticism += 8`, `main.cpp:207`).
- **Fixed** — Big Five baseline, attachment, sex, birth year.

Every rate here is a guess — `0.01` change-rate, the `|reward|>70` threshold, grief recovery `0.0008`/tick (`Entity.cpp:313`), the `0.98/0.99` mood decays. This is precisely the "calibrate with real numbers instead of 0.7f guesses" ask.

### Grief — a Kübler-Ross sub-timescale

`GriefState` (`Entity.h:95`) carries `stagesRemaining` (5, Kübler-Ross) and `intensity`. `tickGrief` (`Entity.cpp:311`) bleeds intensity slowly; crossing each `stageThreshold = stagesRemaining/5 − 0.1` decrements a stage. Attachment style scales recovery. It is the cleanest, best-wired psychological timescale in the codebase.

---

## 5. The appraisal → emotion rule (Gap 4)

`EmotionalComplexitySystem::generateEmotion` (`EmotionalComplexity.cpp:209`) is the OCC/Scherer-style mapping. It takes five appraisals — relevance, desirability, coping, control, normCompatibility, plus `causingEntityId` — and runs a **hand-coded if/else cascade** (`EmotionalComplexity.cpp:229-253`):

| Appraisal pattern | Emotion |
|---|---|
| desirability > 60, control > 50 | JOY |
| desirability > 60, control ≤ 50 | SURPRISE |
| desirability < 40, control < 30, relevance > 70 | FEAR |
| desirability < 40, control < 30, relevance ≤ 70 | SADNESS |
| desirability < 40, control ≥ 30, **someone to blame** | ANGER |
| desirability < 40, control ≥ 30, no agent | SADNESS |
| neutral desirability, normCompatibility < 30 | DISGUST |
| else | SURPRISE |

This confirms Gap 4 on two counts. First, it **is** hand-coded — no principled dimensional model. Second, it is impoverished: of the **14** `BasicEmotion` values, the primary rule can only ever emit **6** (JOY, SURPRISE, FEAR, SADNESS, ANGER, DISGUST). SHAME, GUILT, PRIDE, ENVY, GRATITUDE, HOPE, DESPAIR, CONTEMPT are unreachable from appraisal — only GUILT and PRIDE ever appear, and only via `generateMetaEmotions` (`EmotionalComplexity.cpp:503`, "guilt about anger", "pride about joy"). The anger-vs-sadness fork hinges entirely on `causingEntityId >= 0` — a blame proxy so crude it can't distinguish anger from indignation from contempt. And, again: **none of it runs**, because the system is never instantiated.

---

## 6. Body → mind coupling (Gap 5)

The visceral→psychological coupling *is* wired, but only body → **stats**, via the tick cascades in §2: starvation drags happiness/stress, fatigue frays mental health, drive `mindLoad` erodes mental health at `×0.05` (`main.cpp:1064`), pleasure `tolerance` drags happiness at `×0.15` (the hedonic treadmill). What Gap 5 actually asks — does a hungry/tired body distort **judgment and risk-taking**? — is only partly present, and lives in the decision layer (`04`): the free-will reflex path fires `EatMeal` when `hunger ≥ 70` (`implem_free_will.cpp:1260`). That is a hard threshold reflex, not a graded somatic-marker bias on time-discounting or perception. The coupling *strengths* (`0.05`, `0.15`, `0.18`) are, once more, hand-guessed.

---

## By the numbers

| Thing | Count / value | Where |
|---|---|---|
| Big Five traits | 5, independent `N(50,20)` draws | `main.cpp:170` |
| Attachment styles | 4 (secure/anxious/avoidant/disorganized) | `Entity.h:50` |
| ValueSystem values | 5 (Fam/Ach/Hed/Col/Spi) | `Entity.h:151` |
| Legacy scalar needs/stats | ~10 floats (`[0,100]`) | `Entity.h:248-327` |
| Bipolar drives (wired) | 5 (hunger, fatigue, stimulation, social, pleasure) | `Drive.cpp:120` |
| Maslow HierarchicalNeeds (unwired) | 11 across 5 tiers | `Entity.cpp:604` |
| `BasicEmotion` types | 14 defined; **6** reachable by primary appraisal | `EmotionalComplexity.h:11` |
| Grief stages (Kübler-Ross) | 5 | `Entity.h:95` |
| Personality-drift trigger | only when `|reward| > 70`, rate `0.01` | `LearningAdaptation.cpp:305,234` |
| `Entity.h` | 515 LoC | core data model |
| `Entity.cpp` | 892 LoC | ticks, grief, spawn helpers, PAD |
| `EmotionalComplexity.cpp` | 603 LoC | appraisal→emotion (**unwired**) |
| `EmotionalComplexity.h` | 194 LoC | emotion/mood/regulation structs |
| `Drive.h` | 126 LoC | bipolar homeostatic model |

---

## Honest limitations

- **Two whole subsystems are dead code.** `EmotionalComplexitySystem` (emotion + mood + regulation + contagion, ~800 LoC across the .h/.cpp) and the Maslow `HierarchicalNeed` ladder are defined, thoughtful, and **never instantiated or ticked**. The live "psychology" is the coarse legacy floats + per-tick PAD + the drive layer.
- **No true timescale separation at runtime.** With emotion/mood unwired, "affect" is recomputed statelessly every tick from the same floats — the fast/slow layering exists on paper (`question.md` Gap 2) but not in execution.
- **Traits are independent Gaussians.** No facet correlation (Gap 3); the population is centered on 50 with childhood trauma the only source of structured covariance.
- **Everything is hand-tuned magic numbers** — decay rates, thresholds, drift deltas, coupling strengths. There is no validation layer (Gap 6) checking that emergent life-outcome distributions look human.

---

## Ideas worth stealing for petri-dish-of-madness

- **The bipolar Drive with lethal floor *and* ceiling + allostatic load + habituation** (`Drive.h`) is the single best idea here: cheap, deterministic, and it produces "short-term fine, sustained kills" texture plus a hedonic treadmill for free. Portable as a per-agent state vector an LLM reads as context ("you are chronically over-stimulated and worn down") rather than recomputes.
- **Appraisal→emotion as a tiny decision table.** Even the impoverished 6-way rule (§5) is a deterministic, auditable way to label an event's emotion from `(desirability, control, relevance, blame)`. In an LLM sim, compute the appraisal numerically, pick the emotion from the table, and hand the *label* to the model — far cheaper and more consistent than asking the LLM "how do you feel?".
- **Childhood → attachment → lifelong trait shift**: a one-time deterministic character-minting pipeline (`main.cpp:1830-1850`) that yields diverse, backstory-consistent agents. Trivially portable as a spawn-time prompt-fact generator.
- **Kübler-Ross grief as a decaying multi-stage state** with attachment-scaled recovery — a clean template for any long-tail emotional aftermath (loss, betrayal, exile) that should color behavior for a bounded, self-resolving window.
