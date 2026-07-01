# 15 — Expansion Ideas (idea-level, for the ASHB2 author)

> **STATUS — IDEAS, NOT A PLAN.** This is the *blue-sky companion* to [14-expansion-roadmap.md](14-expansion-roadmap.md).
> Doc 14 is about **depth** — finishing and calibrating the machine you already built (wire the
> dead code, give the decision core a real arbiter, tame the jealousy runaway). This doc is about
> **scope** — where the sim could grow *into*. Deliberately no file paths, no verify steps, no
> code. Just directions, why each one fits, and what new thing it would make happen.

**Read doc 14 first.** Almost everything here assumes the foundation is solid — a world where
micro actually causes macro ([07](07-civilization-engine.md)) and minds aren't bland ([03](03-entity-and-psychology.md)). Building these ideas *on top of*
the current disconnects would just add more surface that looks alive but isn't. Depth before scope.

---

## Anchor every new idea on your three superpowers

The deep-dive found ASHB2 has three things that are genuinely rare in the field ([13](13-frontier-assessment.md)). Ideas
that ride these compound your existing advantage; ideas that ignore them are just features anyone
could build. Every entry below is tagged with which one(s) it exploits:

- **`[Det]` Deterministic personality-physics at civilization scale.** Thousands of psychologically
  detailed agents, **zero** external calls, macro-history falling out of the trait distribution. Nobody
  else gets to treat a whole civilization's history as a *pure function of a seed.*
- **`[Div]` Seeded divergence with a wildness dial + a hashable history signature.** Isolated regions
  drift into different faiths, tech orders, and languages; a knob-set tunes how wild; a hash proves two
  runs actually diverged ([08](08-world-and-environment.md)).
- **`[Chr]` The log→AI post-mortem chronicle.** You already treat the event log as a first-class
  product and refine it into narrated history ([10](10-infrastructure-and-postmortem.md)). That pipeline is a launch pad, not an endpoint.

Ambition tags: **`weekend`** (a focused sprint) · **`project`** (a real chunk of work) ·
**`moonshot`** (redefines what ASHB2 *is*).

---

## North star: the reproducible history generator

**The single most distinctive product ASHB2 could become is a deterministic *world-and-history
generator* — feed it a seed, get back a coherent fictional civilization: a map, its peoples,
their religions and languages, a timeline of wars and golden ages, and the notable figures who
shaped it — all reproducible from that seed, and all shareable as an artifact.** `[Det][Div][Chr]` · **moonshot**

This isn't a new feature; it's a *reframing* of what you already have. Most generative-history tools
are either shallow (name generators, random tables) or non-reproducible (LLM one-shots you can't
re-derive). ASHB2 is the rare thing that is **both deep and reproducible**: a real psychological
substrate producing real emergent history, from a number. That combination is your moat. Every idea
below is, in some sense, a facet of turning ASHB2 from "a sim you watch" into "a world you can
generate, share, and return to." Pick the north star and the rest of this doc becomes a backlog.

---

## Theme 1 — Deepen the world so more *history* emerges

New **kinds** of emergent phenomena — not fixing existing systems (that's doc 14), but adding
substrates that let genuinely new stories happen.

- **Material culture & heirlooms → archaeology.** `[Det]` · **project**
  Right now "culture" is a vector of values; it has no *things*. Give the world persistent objects — a
  founder's axe, a holy relic, a monument raised at a battle site — that carry meaning, get inherited,
  get looted, get buried. Suddenly a run leaves *artifacts*, and a late-era tribe can dig up an
  early-era one and misread it. That's archaeology, mythmaking, and forgery as emergent behavior — and
  it's a natural home for the dead `CulturalTransmission` machinery.

- **Writing, and the death of knowledge.** `[Det][Chr]` · **project**
  Today knowledge effectively dies with its knowers (the `loseTechnology` / low-`knowerCount` finding,
  [07](07-civilization-engine.md)). Make that a *feature* by adding its cure: the invention of **writing** as a tech that lets
  culture survive the death of any individual. Now oral vs. literate societies diverge hard — literate
  ones accumulate and recover from dark ages; oral ones repeatedly reinvent and forget. The chronicle
  practically writes itself: "the Second Library burned, and with it the memory of iron."

- **Institutions that outlive people.** `[Det]` · **project**
  You have leaders, classes, and patron-client ties — all *personal*. The frontier is the **impersonal**:
  a council, a code of law, a priesthood, a treasury — structures that persist across the death of the
  individual holding the office. This is the leap from "a strong chief" to "the State," and it's exactly
  the kind of macro-permanence the sim currently lacks. Succession crises, coups, and constitutional
  drift become possible only once the office is separable from the officeholder.

- **A memetic layer parallel to religion.** `[Div]` · **project**
  Religions already found, spread, and (should) schism. Generalize the mechanism into **ideas that
  aren't religions** — political movements, philosophies, taboos, fashions, conspiracy theories — that
  compete, mutate, and ride migrations. A society's "vibe" becomes a population of memes under
  selection, which is a second, faster inheritance channel layered on the genetic one. Great fuel for
  the divergence dial: two isolated regions evolving incompatible ideologies.

- **Catastrophe as an era-maker.** `[Det][Chr]` · **weekend→project**
  You have disease, drought, and blight, but they're grind, not history. Let a rare catastrophe be
  *era-defining*: a plague that halves a region and permanently shifts the survivors' religion and labor
  structure; a volcanic winter that triggers a mass migration. The chronicle already loves a death
  event — give it civilization-scale ones with named aftermaths ("the Long Winter," "the Great Dying").

- **Trade networks & economic geography.** `[Div]` · **project**
  Biomes already differ; let that difference *flow*. Goods move between regions, specialization emerges
  (this valley smiths, that coast fishes), trade routes become cultural conduits that carry memes and
  disease alongside grain, and a resource-rich region can suffer a "resource curse." Geography stops
  being just a divergence source and becomes a connective tissue that ties the diverged pieces back
  together — which is where the *interesting* history (empire, dependency, war over routes) lives.

---

## Theme 2 — Make the generated world legible and shareable

ASHB2 generates rich history and then mostly *keeps it to itself*. These ideas are about **surfacing**
the world as something a human can read, love, and pass around — the highest-leverage payoff of a
deterministic sim, because the artifact is stable.

- **The auto-generated Codex.** `[Chr]` · **project**
  Every run already contains an encyclopedia's worth of content: every religion, tribe, war, tech, and
  notable figure. Turn that into a **browsable in-world codex** — one entry per entity of history, each
  cross-linked, each with the chronicle's prose. A run stops being a log you scroll and becomes a
  *wiki of a world that never existed.* This is the most direct extension of your existing chronicle
  pipeline and probably the highest joy-per-effort idea in this doc.

- **Readable culture: names, places, a tiny conlang.** `[Div]` · **weekend→project**
  The `Lexicon` already drifts phonemes per region ([08](08-world-and-environment.md)) — but the output isn't *read*. Grow it into
  place-names on the map, personal names that mark which culture someone is from, and a small,
  self-consistent naming grammar per region so the map itself *reads* as a real world. When two tribes'
  languages visibly creolize at their border, the divergence engine becomes something you can see at a
  glance instead of proving with a hash.

- **Myths, scripture, and the products of belief.** `[Chr][Div]` · **project**
  A religion currently has a name and a moral code. Give it its **artifacts of belief**: a generated
  creation myth, a founding legend, taboos with in-world justifications, maybe a short "scripture."
  This is the perfect job for the offline LLM path you already own — feed it the religion's founder
  psychology, its schism history, and its catastrophes, and let it write the holy book. Deterministic
  substrate, generative surface: exactly the two-tier split ([12](12-convergence-analysis.md)) played to its strength.

- **"Follow one soul" biography mode.** `[Det][Chr]` · **weekend**
  A camera/reader mode that follows a single agent's entire life as a readable story — their loves,
  griefs, the war they fought in, the tech they carried, the descendants they left. Because the run is
  deterministic, that biography is stable and re-derivable. It's also the most *emotionally* legible
  view of a sim that currently only shows aggregate carnage; one life is a story, 89% crime-of-passion
  deaths is a statistic.

- **The history signature as a shareable fingerprint.** `[Div]` · **weekend**
  You already hash the run into a divergence signature. Make it a **social object**: a short,
  human-shareable world-fingerprint ("seed 4471 → a sea-empire that outlawed its own founder's faith"),
  a gallery of famous seeds, a "seed of the week." This costs almost nothing and turns reproducibility
  from an internal correctness property into a *community feature*.

---

## Theme 3 — Turn determinism into the killer feature

Most sims can't do these at all, because they aren't reproducible. ASHB2 can — this is where `[Det]`
stops being a nice property and becomes the whole point.

- **Counterfactual history: branch and diff.** `[Det]` · **project**
  Fork a save at tick N, change **one** thing (kill this leader, gift that tech, spare this famine), run
  both forward, and **diff the two histories.** Because everything downstream is deterministic, the
  divergence you see is *caused* by your one change — a clean natural experiment. "What if the prophet
  had died young?" becomes a runnable, rigorous question. Almost nothing else in the genre can offer a
  controlled counterfactual; your determinism makes it trivial.

- **God-mode intervention.** `[Det]` · **weekend→project**
  Let a human reach in and nudge the world — drop a drought, gift or delete a technology, assassinate a
  leader, force a migration — then watch the ripple. Pairs naturally with counterfactual mode (intervene
  on one branch, leave the control branch alone). Turns the sim from a thing you watch into a thing you
  *play*, without giving up reproducibility as long as interventions are logged into the seed-derived
  event stream.

- **A scenario library.** `[Det][Div]` · **weekend**
  Authored, reproducible starting conditions — "two tribes, one island, one water source"; "a lone
  seed population after a near-extinction" — saved as named scenarios anyone can load and get the *same*
  world from. This is how you make ASHB2 teachable and comparable: everyone can run the identical setup
  and discuss the identical history. It also gives *you* a regression suite of interesting worlds.

- **An experiment harness for emergent questions.** `[Det][Div]` · **project**
  Sweep the `DivergenceConfig` knobs (and any others) across hundreds of headless runs and measure what
  emerges: *what conditions produce matriarchy vs. patriarchy? monotheism vs. pantheon? empire vs.
  perpetual tribes?* You already have the wildness dial and the history signature; this idea just points
  them at a question and aggregates. It's the bridge from "a toy that makes stories" to "a testbed that
  answers questions," and it's where your combined psychology-plus-history design pays off most.

---

## Theme 4 — Grow into new audiences and scope

Positioning ideas — less about new mechanics, more about **who ASHB2 is for** and how big it can get.

- **A worldbuilding engine for writers, GMs, and game designers.** `[Det][Div][Chr]` · **moonshot**
  This is the north star as a *product*. A novelist or tabletop GM needs a deep, self-consistent,
  *reproducible* world with real history, named figures, and believable religions — and ASHB2 generates
  exactly that from a seed. Export a world as a shareable pack (map + peoples + timeline + codex), and
  you've built the tool that generic name-generators and non-reproducible LLM one-shots can't. This is
  the most credible path from "impressive personal project" to "thing other people depend on."

- **An ALife / computational-social-science testbed.** `[Det][Div]` · **project→moonshot**
  The experiment harness (Theme 3), taken seriously, is a research instrument: reproducible runs,
  controlled interventions, measurable emergent patterns validated against real demography and
  anthropology (Zipf's law for settlement sizes, war-frequency distributions, kinship structures). The
  validation work in doc 14's Phase 5 is the on-ramp; this is the destination. It's also the framing
  most likely to get the project *cited* rather than just starred.

- **Data-driven modding.** `[Div]` · **project**
  Expose the trait, tech, religion, and biome tables as **data** rather than C++, so others can add a
  tech branch, a new value axis, or a starting scenario without touching the engine. This is the
  difference between a project one person maintains and a world other people *tinker with* — and it
  multiplies the divergence space for free (every mod is new history).

- **Multi-scale zoom (LOD for civilizations).** `[Det]` · **moonshot**
  To reach *continents* and *millennia*, you can't simulate every agent in full forever. The frontier is
  **level-of-detail for people**: distant/old populations simulated statistically (as distributions and
  aggregate flows), nearby/present ones in full psychological detail, with seamless promotion/demotion
  between the two as the camera and the era move. This is the hard, ambitious way to make the "society
  in a box" a *world* in a box — and it's the kind of problem that would define the project's second act.

---

## If you only pick three

Ordered for compounding value on top of a doc-14 foundation:

| Pick | Idea | Why this one | Ambition |
|-----:|------|--------------|:--------:|
| 1 | **The auto-generated Codex** (Theme 2) | Highest joy-per-effort; directly extends the chronicle you already own; makes *every other* emergent system finally legible and shareable. | project |
| 2 | **Counterfactual branch-and-diff** (Theme 3) | Converts your rarest asset — determinism — into an experience nothing else in the genre can offer. Cheap given you already have saves + a signature. | project |
| 3 | **The reproducible worldbuilding engine** (Theme 4 / north star) | The idea that gives all the others a *purpose* and the project an audience beyond yourself. Everything above is a component of it. | moonshot |

Codex makes the world **readable**, counterfactuals make it **explorable**, and the worldbuilding
engine makes it **useful to someone else** — read, play, share. That's a coherent second act.

---

## The through-line

Doc 14's message was *converge the aspiration and the reality* — run what you wrote. This doc's
message is the mirror image: **once the machine is real, your unfair advantage isn't more mechanics —
it's that your worlds are reproducible.** A deterministic civilization is a *seed you can share, branch,
and return to.* Almost every idea here is a way of cashing that in: a codex of a world that never
existed, a controlled experiment on a dead prophet, a fictional history a stranger can regenerate
byte-for-byte. Build the depth first (doc 14); then spend the determinism (doc 15).

> Note for the petri-dish reader: these are ASHB2's frontiers, not petri-dish's — petri-dish is
> LLM-native and *non*-reproducible by design, so the determinism-powered ideas (Themes 3–4) are
> exactly the ones it **can't** copy. What petri-dish *can* take stays in [13](13-frontier-assessment.md)'s ranked backlog. The
> most interesting cross-pollination is the shared two-tier vision ([12](12-convergence-analysis.md)): ASHB2's Theme-2 "generative
> surface on a deterministic substrate" is the same bet petri-dish makes from the opposite shore.
