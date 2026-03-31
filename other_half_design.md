# Other Half
### A Companion Mod for Project Zomboid
**Build 42 Target · Design Document v2.0**

> *"Find them. Keep them alive. Feel it when you don't."*

---

## Table of Contents

1. [North Star & Design Philosophy](#01--north-star--design-philosophy)
2. [The Current Landscape](#02--the-current-landscape)
3. [Optional Dependencies — Vanilla+ Philosophy](#03--optional-dependencies--vanilla-philosophy)
4. [System 0 — The Companion Profile](#04--system-0-the-companion-profile)
5. [Finding the Companion](#05--finding-the-companion)
6. [System 1 — The Bond Engine](#06--system-1-the-bond-engine)
7. [System 2 — The Threat AI](#07--system-2-the-threat-ai)
8. [System 3 — The Personality Core](#08--system-3-the-personality-core)
9. [System 4 — Interaction & Communication](#09--system-4-interaction--communication)
10. [System 5 — The Home Base](#10--system-5-the-home-base)
11. [System 6 — Independent Life](#11--system-6-independent-life)
12. [System 7 — Separation & Reunion](#12--system-7-separation--reunion)
13. [System 8 — The Sacrifice Play](#13--system-8-the-sacrifice-play)
14. [System 9 — The Dialogue Engine](#14--system-9-the-dialogue-engine)
15. [System 10 — Romance](#15--system-10-romance)
16. [System 11 — Death & Grief](#16--system-11-death--grief)
17. [Implementation Roadmap](#17--implementation-roadmap)
18. [Technical Architecture](#18--technical-architecture)
19. [Competitive Differentiators](#19--competitive-differentiators)
20. [Closing Statement](#20--closing-statement)

---

## 01 — North Star & Design Philosophy

Every companion mod that has come before this one has the same two failures. The first is **mechanical**: companions aggro hordes, flee into players, waste ammunition, and get stuck on doors. The second is **emotional**: they give you workers, not a person. You never grieve them.

Other Half fixes both. Its governing philosophy is **one companion, completely realized**. Not a squad of helpers. A single human being — with a past, a personality, good days and bad, habits, fears, and a relationship with you that is built slowly and lost completely.

The emotional and mechanical failures of the genre are not separate problems. Players don't invest in companions they can't trust. You cannot grieve someone whose death felt like a bug. The Threat AI must be bulletproof first — because everything else only lands with weight when the player already trusts that the companion's behavior makes sense.

> **The Three Laws of Other Half**
> 1. The companion must never be the cause of the player's death through reckless behavior.
> 2. The companion must feel like a person the player chose to survive with — not an asset they deployed.
> 3. Losing the companion must hurt. That requires first making the player love them.

---

## 02 — The Current Landscape

### The Superb Survivors Problem

Superb Survivors is the longest-running companion mod in the community, and also a cautionary tale. Its core AI failure is well-documented: when NPCs encounter zombies they attack until overwhelmed, then panic-flee directly into the player, dragging a horde behind them. A configured horde cap of 10 can mushroom to 50 because of NPC-triggered migrations.

The mod's own continuation fork explicitly states: *beyond Follow, Barricade, and Pile Corpses, the NPCs cannot be trusted to do much besides dying and attacking.* This is not a criticism of its authors. It is a diagnosis. The architecture is the problem, and architecture can only be fixed by starting over.

### The Emotional Gap

The deeper failure of the existing NPC ecosystem is not technical. These mods were designed around utility — companions as tools that extend the player's capability. No existing mod has been built around the question: **what would it take to make a player feel something when this character dies?**

The answer is not better models or voice acting. It is consistency, memory, and time. A companion who does the same things in response to the same stimuli, every session, across many hours — that companion becomes real. Other Half is built around that principle.

Official NPC survivors are planned for Build 43 with complex AI and narrative stories, but remain unimplemented. This mod owns that emotional territory now.

---

## 03 — Optional Dependencies — Vanilla+ Philosophy

**Core rule:** Other Half must be fully playable, emotionally complete, and mechanically solid with zero optional dependencies installed. Every dependency is an enhancement, never a requirement. Players should never feel they are missing anything by running vanilla.

The mod detects optional dependencies at load time and gracefully activates enhanced behavior when present. Nothing breaks if they are absent — the system falls back silently to vanilla equivalents.

| Mod | Category | What it unlocks |
|---|---|---|
| True Actions: Sitting & Lying | Animation | Companion sits at furniture during downtime, lies on beds during sleep cycles. Enormously increases home-life believability. |
| True Actions: Dancing (B42) | Animation | Companion dances autonomously on good-day rolls, when a working radio is found, and at relationship milestones. 45+ animations. |
| Spongie's Open Jackets | Visual | Companion clothing state reflects wear and environment. Jacket opens as they warm up, layers adjust seasonally. |
| Tomb's Player Body Overhaul | Visual | Higher fidelity character model. Companion appearance feels less plastic, more human. |
| Vanilla Clothing Expansion (VCE) | Visual | 200+ clothing combinations for companion generation. No two companions look alike across playthroughs. |
| Comfy Sleeping | Mechanic | Companion sleep quality tracked, affects disposition. Sleeping near the player applies a small mutual rest quality bonus. |
| True Music | Mechanic | Companion reacts to instruments the player plays. Mood uplift on good-day disposition rolls. |
| Dynamic Moodles (B42) | UI | Companion stress and mood states visible as moodle icons on the companion HUD panel. |

> **Implementation note:** Dependency detection uses a mod ID check at `OnGameStart`. Each optional system is wrapped in a feature flag (`COMPANION_FEATURES.trueActions`, etc.). All feature-gated code paths have a vanilla fallback. No optional dependency is ever listed as `Required` in `mod.info`.

---

## 04 — System 0: The Companion Profile

The Companion Profile is a lightweight pre-game configuration screen added as its own tab in character creation. It is the configuration layer that all other systems draw from. It does not create the companion — it shapes the probability space the world draws from when generating one.

**Core principle:** sliders and toggles, not decisions. Every option has a sensible default. The player who doesn't touch it gets a solid, balanced experience. The player who does gets exactly what they came for. You are not building a second character — you are setting the stage.

---

### Section 1: Survival Stakes

The most impactful single setting. Displayed as a named range with tooltips explaining what each tier means mechanically. No hidden math — players should understand what they are signing up for.

| Tier | Name | What it means |
|---|---|---|
| 1 | **Fragile** | Realistic injury thresholds. Companion can die from sustained danger. One catastrophic event can end the story. |
| 2 | **Vulnerable** | Heightened danger. Death possible but requires sustained failure across multiple systems. |
| 3 | **Balanced** *(default)* | The intended experience. Danger is real, death is possible, but the player has clear warning and time to act. |
| 4 | **Resilient** | Companion can absorb significant punishment. Death requires extraordinary and sustained failure. |
| 5 | **Enduring** | The Hulk in disguise. The companion will not die under any normal circumstances. Stakes are emotional only. |

---

### Section 2: Companion Generation Preferences

These settings shape who is out there waiting to be found. They shift probability, not guarantee outcomes.

**Gender Preference**
A percentage slider from 0% to 100% representing the chance the companion is female. Default: 50/50. Fully adjustable to either extreme. The UI labels the ends clearly without editorial commentary.

**Personality Core Access**
Checkboxes for each of the six dominant traits (Cautious, Reckless, Nurturing, Sardonic, Devout, Pragmatic). Unchecking a trait removes it from the generation pool entirely. All checked by default.

**Skill Predisposition**
A single slider from Domestic (Cook, Nurse, Farmer) to Combat (Ex-Military, Construction, Mechanic). Default center. Shapes which skill background is more likely to appear.

**Sexuality Predisposition**
Shapes what relationship paths are available to develop organically. Options: Straight, Gay, Bisexual, Any (default). Automatically grayed out with a tooltip if the Relationship Type Ceiling is set to Survival Partner or Close Friend.

**Skill Starting Levels**
A global modifier from Low (the companion starts green, growth is part of the story) to High (they arrive experienced and competent from day one).

**Relationship Gain / Loss Rate**
Two independent sliders. Trust gain rate affects how quickly positive actions accumulate bond. Trust loss rate affects how quickly negative actions and neglect erode it. Default: both at center.

---

### Section 3: Relationship Type Ceiling

The most important framing decision the player makes. This gates dialogue branches and autonomous behaviors across the entire playthrough.

| Ceiling | What it means |
|---|---|
| **Survival Partner** | Platonic ceiling. Deep trust and care can develop but romance dialogue, physical proximity escalation, and the Partner bond tier are permanently unavailable. The Joel/Ellie dynamic. |
| **Close Friend** | Emotional ceiling. The relationship can become deeply personal and vulnerable — but stops short of romance. The bond feels chosen and non-negotiable in its own way. |
| **Open** *(default)* | All paths available. Romance can develop if the organic bond conditions are met. Nothing is guaranteed, nothing is forced. The relationship goes where it goes. |

---

### Section 4: Death & Continuation

Configures what happens to the world and the mod if the companion dies. All settings in this section are irrelevant if Survival Stakes is set to Enduring.

| Setting | Description |
|---|---|
| **Death = End of Mod** | Toggle. When on, companion death permanently ends the companion system for that playthrough. No new companion will ever spawn. The Rapport Log and Keepsake item remain. Default: Off. |
| **New Companion Spawn Chance** | 0–100% slider. Probability that a new companion exists somewhere in the world after the mourning period expires. Default: 80%. |
| **Mourning Buffer Period** | Slider in in-game days (0–30). Minimum time before a new companion can appear. Default: 7 days. The gap is the grief. |
| **New Companion Starting Trust** | Slider from Stranger (default, zero trust, full arc ahead) to Cautious Openness (they can tell you've been through something — a slight head start). |
| **New Companion Spawn Zones** | Checkboxes: Residential, Commercial, Rural, Survivor Camps. Shapes where in the world the next companion is likely to be found. |
| **Mental Health Consequence Scale** | Slider from None to Severe. Controls how deeply companion death at high bond levels impacts the player character's psychological state. Default: Moderate. |

> **On death consequences at high bond:** At maximum relationship depth, losing the companion is the most destabilizing event in the playthrough. Sleep quality tanks. Happiness caps drop. Certain tasks feel heavier. At the Severe setting, the player character enters a grief spiral — real mechanical impairment requiring active recovery. The player has to choose to survive it. That is not grimdark for its own sake. That is the game honoring what the relationship was worth.

---

## 05 — Finding the Companion

The companion does not spawn with the player. They exist somewhere in the world, already surviving, shaped by the Companion Profile settings but fully autonomous until discovered. **Finding them is the beginning of the story — it should feel like a discovery, not a tutorial prompt.**

### Where They Are

Spawn location is drawn from the zones checked in the Companion Profile. The companion is placed in a location that makes narrative sense for their skill background — a Nurse near a clinic, a Mechanic in an auto shop, a Cook in a restaurant or home kitchen. They are not in the open. They are holed up, surviving.

The location is flagged at world generation and does not change. If the player never goes there, the companion waits. They are in one place, surviving alone, and the player has to find them.

### The Encounter

When the player enters the companion's zone, the game does not fire a notification. The player simply finds them — through sound (a radio playing, movement inside a building), through sight (a light in a window at night, a barricaded door that wasn't there before), or through pure exploration.

The first interaction is not a recruitment menu. The companion is wary. They speak first — what they say is shaped by their dominant trait. The Sardonic one has a dry line ready. The Cautious one has a weapon visible. The Nurturing one asks if you're hurt before asking your name.

The player can engage or leave. Leaving is allowed — the companion stays there. The player can come back. This is the first moment the companion's trust in the player begins to form, and doing nothing is a kind of answer too.

### Earning the Companion's Trust — Bidirectional from Day One

**This is the critical design distinction from every other companion mod:** the companion's trust in the player is independent of the player's trust in the companion, and it starts at zero for both.

The companion is not a recruit waiting to be activated. They are a survivor who has stayed alive without you. They do not automatically follow. They do not automatically agree. They evaluate the player — with particular weight on whether the player's behavior in dangerous situations protects both of them, not just themselves.

**Early trust signals the companion reads:**
- Player retreats cleanly rather than creating unnecessary danger → positive
- Player shares food or medicine without being asked → positive
- Player engages a horde that could have been avoided → negative
- Player escapes clean while companion was in danger → significant negative
- Player returns after initially leaving → neutral to slightly positive (they came back)
- Player gives the companion a weapon → positive (practical trust signal)
- Player gives orders before trust is established → companion pushes back or ignores

> **The recruitment moment:** There is no formal recruitment dialogue. The companion begins following when their trust score crosses a threshold — they decide. The transition is marked by a single line of dialogue from them, shaped by their trait and the specific events of the encounter. It is not a confirmation prompt. It is a person choosing to take a chance on someone.

---

## 06 — System 1: The Bond Engine

The Bond Engine is the emotional core of the mod — a persistent relationship state machine stored in `ModData`, tracking the relationship across five independent dimensions simultaneously. Every other system reads from and writes to it.

### Dimension 1: Player Trust in Companion

How much the player character trusts the companion's judgment, reliability, and commitment. Increases through demonstrated competence and consistency. Decreases when the companion makes errors that cost the player.

### Dimension 2: Companion Trust in Player

**This is the dimension no other mod has tracked.** The companion's confidence that the player will not endanger, abandon, or betray them. This governs the companion's willingness to follow instructions, take risks, and open up emotionally.

**Companion trust increases when:**
- Player stays close during danger
- Player shares limited resources
- Player returns from solo runs
- Player treats companion injuries unprompted
- Player shields companion from direct threat
- Player makes decisions that protect both, not just themselves

**Companion trust decreases when:**
- Player repeatedly creates avoidable danger
- Player escapes clean while companion was at risk
- Player denies companion food or medicine while holding both
- Player gives orders without pulling their weight
- Player initiates unnecessary combat near the companion
- Extended separation without the companion having enough supplies

At critically low companion trust, the companion does not comply with orders — they push back, ask questions, or refuse. At rock bottom — sustained low trust over many days — **they leave.** This is a more interesting failure state than death.

### Dimension 3: Intimacy

Depth of emotional openness. Cannot be rushed — requires both trust dimensions to be reasonably high before it moves. At low intimacy the companion speaks practically. At high intimacy they are unguarded, they ramble, they share things they have not told anyone. Romance, if the ceiling allows it, only becomes possible at high intimacy.

### Dimension 4: Dependency

A natural byproduct of survival style, not a value judgment. High-dependency companions are more anxious during separations and carry higher emotional weight in death. Low-dependency companions are stoic and autonomous. Dependency drifts organically — it cannot be artificially inflated.

### Dimension 5: The Rapport Log

A capped ring buffer of up to 30 named, located, timestamped events stored in `ModData`. Not abstract stat increments — real moments: the first zombie killed together, the night the power went out, the gas station on the second run, the birthday cake. The dialogue system pulls from this log. The companion references real events in conversation.

**Rapport Log entry schema:**
```
eventType:         string   (FIRST_KILL | CLOSE_CALL | GIFT | SHELTER_FOUND | INJURY |
                             SACRIFICE_PLAY | MILESTONE | REUNION | DANCE | ...)
locationName:      string   (pulled from vanilla map zone definitions)
dayNumber:         integer
companionReaction: enum     (SHAKEN | PROUD | GRATEFUL | AFRAID | MOVED |
                             DARKLY_AMUSED | QUIET)
referenceKey:      string   (unique key for the dialogue system to retrieve by name)
```

---

## 07 — System 2: The Threat AI

**This is the technical core of the mod and its primary differentiator.** The failure of every previous companion mod is rooted in a single design error: companions treat combat engagement as a default. Other Half treats it as a last resort.

The threat AI runs on a configurable tick — defaulting to every 1.5 seconds, never every frame — and operates as a four-stage assessment loop, written from scratch against the B42 Lua API.

### Stage 1: Zone Classification

| Zone | Condition | Behavior |
|---|---|---|
| **SAFE** | 0 zombies visible within 15 tiles | Full independent activity. Normal movement. Task execution continues. |
| **CAUTION** | 1–3 zombies, not actively aggro'd | Crouches if player is crouching. Silent melee allowed if skill qualifies. No firearms, ever, unless commanded. |
| **DANGER** | 4+ zombies OR any sprinters OR player grabbed | Flee routing activates. Only engages to protect player if player is grabbed. Routes to player first, then safe waypoint. |

### Stage 2: Engagement Gating

Combat is opt-in, not default. The companion will **not** initiate an attack when:
- The player is sneaking (mirrored stance, hard override on all companion behavior)
- The zone is DANGER
- A firearm is the available weapon — **never autonomous, ever**
- The companion is more than 8 tiles from the player without direct melee contact on them

### Stage 3: Noise Discipline

The companion inherits the player's `isSneaking()` state as a hard override. While sneaking:
- Movement capped at sneak-walk
- Door interaction suppressed
- Task execution paused
- No commentary fires

### Stage 4: Flee Routing — The Core Fix

**This is the fix for the single most infuriating behavior in all existing companion mods.** Priority chain, always computed, never reflexive:

1. **Priority 1:** Route to the player's current tile. Stay within 3 tiles.
2. **Priority 2:** If blocked, route to the most recently placed Safe Waypoint — a droppable marker the player can place anywhere.
3. **Priority 3:** Route to nearest tile that is zombie-free, has line-of-sight to the player, and does not require passing through a zombie cluster.

The companion never picks a random flee direction. The flee path is always computed.

---

## 08 — System 3: The Personality Core

The companion is generated from a randomized but internally consistent trait matrix, drawn from the pools the Companion Profile left available. Personality actively shapes behavior patterns, dialogue tone, task preferences, fear thresholds, and how the companion changes over time.

### The Trait Matrix

**Dominant Trait** (one of six, probability-weighted by Companion Profile checkboxes)

| Trait | Character |
|---|---|
| Cautious | High threat threshold, calm, methodical. Rarely initiates, but never panics. |
| Reckless | Low threat threshold, bold, more useful in combat, more dangerous to be around. |
| Nurturing | Prioritizes player health and comfort. High first-aid initiative. Notices everything. |
| Sardonic | Dry under pressure. Harder to read emotionally. Deeply loyal at high bond. |
| Devout | Has moral lines they won't cross early on. These erode with shared darkness. |
| Pragmatic | Transactional early. Genuinely warm only at high intimacy. |

**Skill Background** (determines utility role)

| Background | What they bring |
|---|---|
| Nurse | High First Aid, notices player injuries, pushes medical care |
| Mechanic | Vehicles last longer, identifies car sounds, knows parts |
| Cook | Forages food, identifies edibles, better calorie rationing |
| Construction Worker | Faster barricading, spots structural vulnerability |
| Farmer | Plant care, animal management (B42), food preservation |
| Ex-Military | Calm under fire, weapon maintenance, lower fear threshold |

**Personal Fear** (shapes danger behavior and thresholds)
- Dark — won't go indoors at night without a light source
- Heights — refuses upper floors unless player has been there first
- Enclosed spaces — panic state in small rooms with threats present
- Large hordes — flee threshold lower than trait would otherwise suggest
- Infection — obsessively checks player for bites after every combat encounter

**Pre-Knox Anchor** — one specific detail from before the outbreak the companion returns to repeatedly. A sibling they haven't found yet, a hobby they keep wanting to pick back up, a small obsessive habit, a regret they circle around without ever naming directly. This is the emotional core of who they were before.

### Trait Evolution

Traits are not static. The dominant trait can shift along specific axes over time, driven by accumulated Rapport Log entries of the right type. A Reckless companion who survives several close calls trends Cautious. A Devout companion who witnesses enough horror stops mentioning God. These arcs are slow, never forced, and never announced.

---

## 09 — System 4: Interaction & Communication

How the player talks to the companion is the system that determines whether the companion feels like a person or a tool. It serves two competing needs: **fast, reliable communication in the field** and **something that feels like actual conversation at home.** The solution is two modes of interaction that switch naturally based on context.

### Field Communication — The Contextual Radial

In the field, interaction is fast and radial. The critical design decision: **the options on the wheel are situationally aware.** The game always knows what is around the player. The wheel only ever shows the four or five options that make sense right now.

**While outdoors, zombies nearby:**
- Stay Close
- Fall Back
- Hold Here — I'll handle it
- We need to run

**While looting a building:**
- Watch the door
- Help me search
- Take this *(hands item)*
- Stay quiet — follow my lead

**While at the home base, no threat:**
- Can we talk? *(opens home dialogue)*
- I need you to do something
- Here, try this on *(opens equipment exchange)*
- I'll be back *(departure acknowledgment)*

**At any time:**
- **Hey** *(single button — prompts companion for an unprompted response)*
- **Safe Waypoint** *(drops a navigation marker at current location)*

The companion responds to every field command with a one-liner shaped by their dominant trait and current disposition. Not a confirmation sound. A sentence.

> **The "Hey" Button**
> A single key binding that prompts the companion for a response based on the current moment — their disposition, the environment, whether they have something on their mind. Sometimes practical. Sometimes emotional. Sometimes they just say something quiet about the weather. This is the interaction that makes players feel the companion is alive. Because sometimes you don't want to give an order. You just want to hear from them.

### Home Communication — Contextual Dialogue

When both are in the home zone, safe, no immediate threat — the interaction system opens fully. When the player initiates conversation, the dialogue system does not present a generic menu. **It assembles a short list of topics based on what it knows right now** — the current Rapport Log, the companion's disposition state, recent events, and unresolved emotional beats.

**Dynamic topic assembly (examples):**
- Companion had a bad day → *"You seem off. What's going on?"* appears as an option
- Player just returned from a long run → a topic about where they went is available
- A close call happened recently → companion may surface it before the player does
- Bond Tier 3+ and a Rapport Log entry references a specific location → that location can be brought up by name
- Companion is running low on a personal supply → they raise it as a topic themselves
- Romance tier reached → a new dialogue branch becomes available at the right moment

The player does not type anything. They choose from three to five dynamically assembled options. No voice acting. All text, surfaced through the vanilla text bubble system or a lightweight custom `ISPanel` dialogue frame.

### Giving Orders vs. Making Requests — Trust-Gated Language

The framing of instructions shifts as companion trust develops. Both the companion's response behavior and the subtle wording of the radial options change over time.

| Companion Trust | Radial Framing | Companion Response |
|---|---|---|
| Low — Stranger | *"Could you watch the door?"* | May comply, may push back, may ask why. Compliance not guaranteed. |
| Mid — Acquaintance | *"Watch the door."* | Complies with verbal acknowledgment. May express a reservation if it seems risky. |
| High — Friend | *"Door."* | Complies immediately with a short affirmation. Occasionally volunteers a better idea. |
| Partner | *(shorthand or gesture)* | Some actions no longer need to be said. The companion reads the situation and acts. |

### Equipment & Clothing Exchange

Clothing and gear exchange is never a stats screen. The player hands the companion an item. The companion evaluates it and responds.

> **The companion knows their body.** Strength and Fitness are tracked companion stats. When given gear, they evaluate it against their current capability.
> - Full plate armor on a low-Strength companion: *"This is going to slow me down more than it protects me."*
> - A jacket that fits their style: they put it on without comment, or with one quiet line.
> - Gear that is clearly better than what they have: accepted, may be acknowledged at higher bond.
> - At high trust, they wear what you ask even with reservations. They may grumble quietly.
> - At low trust, they reject gear that conflicts with their self-assessment regardless of stats.

The player can also give clothing purely for aesthetic reasons — because they like the way the companion looks in it. The companion can tell the difference. At higher bond, they notice. They say something about it.

---

## 10 — System 5: The Home Base

The companion needs a concept of *"ours."* Home is not just a spawn point — it is the emotional anchor of the relationship. As bond deepens, the companion's attachment to the home zone grows from practical orientation to something harder to name.

### Establishing Home

The player designates a home zone using a flagging item — a handwritten note or a piece of tape, craftable from vanilla materials. The flag defines a tile radius (configurable, default 30 tiles) that the companion's AI recognizes as home. The flag can be moved, ending the previous designation.

### What the Companion Does at Home

- **Return point:** Default pathfinding target during Independent Mode and after any flee event.
- **Spatial habits:** Within 7–14 days, the companion develops a preferred chair, a sleeping-side preference, a standing spot when idle. Randomized per companion. Persistent.
- **Maintenance drive:** Cleans nearby corpses, reorganizes scattered floor items, barricades broken windows, tidies the base during personal time.
- **Decoration:** Moves furniture, hangs pictures found in the world, places lamps. No new assets — vanilla item repositioning via `ISTimedAction` chains only.
- **Defense instinct:** When zombies approach the home zone, flee threshold increases significantly. They hold ground longer here than anywhere else.
- **Inventory consciousness:** Maintains a running model of the base inventory. Cooks before being asked if food is available and hunger approaches. Organizes new items into type-appropriate containers. Communicates when supplies run low.

---

## 11 — System 6: Independent Life

When not in Team Mode or under direct instruction, the companion lives. This is a priority-weighted autonomous schedule that responds to their needs, time of day, disposition, and available resources — not an idle animation loop.

### The Priority Queue

1. **Urgent needs** — Hunger, thirst, severe injury. Override everything else.
2. **Assigned tasks** — Any explicit player-issued instruction.
3. **Home maintenance** — Cleaning, barricading, organizing. Run until home is in good order.
4. **Survival tasks** — Cooking, foraging nearby, light crafting. Skill-background weighted.
5. **Personal time** — Trait-driven. See below.

### Personal Time Activities

**Reading** — Picks up books or magazines from the base. Prefers skill books matching their background. At high intimacy, leaves a book on the player's side of the base with a text note.

**Exercise** — Uses vanilla workout animations. Frequency is trait-weighted — athletic types do this daily, sedentary types almost never. Skipped entirely on a bad-day disposition roll.

**Dancing** *(requires True Actions Dancing dependency)* — Fires on: working radio found, good-day disposition roll, relationship milestone event. The first time a player witnesses this, they will screenshot it.

**Sleep** — Uses the vanilla sleep mechanic at appropriate in-game hours. Has a preferred sleeping spot. Proximity bonus to rest quality when Comfy Sleeping is active. Reports bad sleep in morning dialogue — nightmares tied to trauma state.

**Map Study** — If a map item is in the base, companion studies it during personal time. Generates a suggestion event — a contextual recommendation of a specific location based on current supply needs.

**Sitting & Quiet** *(enhanced by True Actions)* — At any intimacy level, the companion sometimes just sits near a window. High-intimacy companions may sit next to the player's last-known resting position.

**Tinkering** — Mechanic and Construction backgrounds find something broken to work on. Occasionally produces a small crafted item left somewhere the player will find it.

**Journaling** — During extended separations, Nurturing and Devout companions write in a notebook if one is available. Contents surface as a reveal on the player's return.

### The Disposition System

Each in-game day the companion rolls a disposition score weighted by:
- Sleep quality from the previous night
- Current hunger and thirst state
- Bond tier (higher bond = higher floor; no immunity to bad days)
- Recent trauma log entries
- Weather (prolonged rain, extreme heat)
- Days since meaningful interaction with the player

A good day: commentary is lighter, energy is higher, they initiate more. A bad day: quieter, shorter, need space. At sufficient bond the player can ask what is wrong, and they answer honestly.

---

## 12 — System 7: Separation & Reunion

### The Separation Timer

| Time Away | Companion Behavior |
|---|---|
| 0 – 4 hours | Normal. Independent activity continues unaffected. |
| 4 – 12 hours | Checks the door more frequently. Task frequency decreases. Begins writing in notebook if one is available. |
| 12+ hours | Minimal activity. Mostly sitting near the entrance. High-dependency companions show visible distress. Notebook entry locks in. |

### The Reunion

When the player re-enters the home zone, a proximity event fires. The companion drops their current action, turns toward the player, and speaks. The line is bond-tier, dependency, and trait-weighted:

- *"I was starting to wonder."* — low bond, low dependency
- *"Don't do that again."* — high bond, low dependency (the sardonic version of relief)
- *"You're hurt. Sit down."* — any bond, Nurturing trait — checks for injury before anything else
- *"[Name] wrote something while you were gone."* — extended absence, notebook content surfaces as a text reveal

The reunion moment lands harder than any scripted cutscene. It costs almost nothing technically.

---

## 13 — System 8: The Sacrifice Play

**"Run. I'll catch up."** — and then they actually do. Or they don't.

### Trigger Conditions

All of the following must be simultaneously true:
- The Desperation Index exceeds the companion's trait-weighted threshold
- A viable split-path exists — the pathfinder has identified a clear route for the player
- Companion health is above 50% (they are choosing this, not desperate)
- Bond tier is at least mid-level (they care enough to do this)

The Desperation Index is a real-time composite of: zombie density in a 10-tile radius, both characters' health states, distance to the nearest safe zone, and whether the player is currently grabbed.

### The Call-Out

Not a canned line. A line shaped by their dominant trait and rapport log history. The Sardonic one makes a dark joke. The Nurturing one says something quiet. The Devout one doesn't say anything — they just push you toward the exit.

### The Outcome

The companion is not martyring themselves. They fight toward a secondary escape. The outcome is probabilistic — adjustable in sandbox options:

- Companion escapes and returns to home base: **40–60%** (skill and health weighted)
- Companion is severely injured but survives: **20–30%**
- Companion does not survive: **20–30%**

The first time they pull this off and show up at the base later — limping, health critical — it will be one of the most memorable moments any player has ever had in this game. If they don't make it, that is System 11.

---

## 14 — System 9: The Dialogue Engine

No voice acting. All text, reactive, firing off Lua event hooks — not on timers. The companion speaks when something happens.

### Event Hooks That Trigger Commentary

**Survival events:**
- Player picks up medication
- Player character reaches Very Hungry
- Rain begins while both are outdoors
- Generator runs out of fuel
- First vehicle found that runs
- Player character falls asleep in the field

**World & milestone events:**
- Zombie heard off-screen — companion goes quiet
- Helicopter event begins
- Power goes out for the first time
- Player kills 100th zombie
- First month anniversary of recruitment
- Player finds a pre-Knox photograph *(new lootable item)*

### Tone Buckets by Bond Tier

| Bond Tier | Dialogue Character |
|---|---|
| 1 — Stranger | Short, practical. *"We should move."* *"There's a pharmacy two blocks east."* No personal content. |
| 2 — Acquaintance | Opinions emerge. Dry observations. Preferences stated. Some humor. No past content. |
| 3 — Friend | Rapport Log references begin. *"Remember the gas station?"* Longer sentences. Deliberate silences. |
| 4 — Intimate | Pre-Knox anchor content unlocks. Stories. Questions about the player's past. Vulnerability. Rambling, in a good way. |
| 5 — Partner | Shorthand. Assumed context. Inside references. The language of people who no longer need to explain themselves. |

---

## 15 — System 10: Romance

Romance is not a menu option. It is a state that emerges when two conditions are simultaneously true for long enough: Bond Tier 5 (Partner) is reached, and the Relationship Type Ceiling is set to Open.

The player does not trigger romance. Acts of care accumulate — sharing the last of a food item, giving a gift, sleeping in the same room voluntarily, staying close during sustained danger, personally tending to an injury. When enough of these compound against a sufficient bond tier, the dialogue shifts. The companion's spatial proximity increases. They save you the better side of whatever you're sleeping in.

There is no cutscene. No flag pop. No achievement sound. Players will know.

Romance does not change the companion's survival behavior or reliability. It changes the texture of the relationship — what they say, how close they stand, what the Rapport Log entries sound like.

---

## 16 — System 11: Death & Grief

**The companion can die.** That is non-negotiable. A companion who cannot die has no stakes, and a companion with no stakes cannot be loved. The Survival Stakes setting in the Companion Profile controls how likely this is — but even at Enduring, the emotional weight of near-death is preserved.

### Warning States

Death is preceded by a visible deterioration sequence. The companion limps. Dialogue becomes shorter and more fragmented. They stop initiating tasks. Their disposition bottoms out. With Dynamic Moodles active, a critical health indicator appears on the companion panel. The player has a window to intervene — generous but finite.

### Mental Health Consequences — Scaled to Bond

| Bond Level at Death | Player Character Consequences |
|---|---|
| Low (Stranger–Acquaintance) | Mild sadness moodle, short duration. A setback. Life continues. |
| Mid (Friend) | Extended sadness moodle. Sleep quality drops. Boredom cap reduces temporarily. |
| High (Intimate) | Deep grief moodle stack. XP from social activities suppressed. Happiness cap significantly reduced. Decays over 14 in-game days. |
| **Partner** | At Mental Health Consequence Scale: Severe — the player character enters a grief spiral. Real mechanical impairment requiring active recovery. The player has to choose to survive it. |

### The Death Event Sequence

1. A single in-game text notification — their name, the location, the cause. No melodrama.
2. Grief Moodle stack applied to player character. Duration and intensity drawn from bond level and consequence scale setting.
3. One item from the companion's inventory is flagged as a **Keepsake**. No gameplay function. It exists in inventory. The player can drop it if they want.
4. Companion name and a brief entry written to the persistent **Memorial Log** — accessible across playthroughs.

### The Mourning Period & Continuation

No new companion can be found until the Mourning Buffer Period expires. The world should feel emptier. The player has to sit with it.

If Death = End of Mod is toggled off, a new companion exists somewhere in the world after the buffer — drawn fresh from the Companion Profile generation pools. They start at zero trust. They do not know you. They are a different person. The world does not pretend the previous one did not exist.

> **The Design Test**
> If a player reads about this system and thinks *"I don't want to lose them,"* the mod has already worked.
> If a player loses their companion and describes it to someone else later, the mod has done its job completely.

---

## 17 — Implementation Roadmap

| # | Phase | Deliverable |
|---|---|---|
| **1** | **Foundation** | Companion spawning framework, Threat AI (all 4 stages), noise discipline, flee routing, `ModData` persistence skeleton. Do not move on until companion behavior is bulletproof across 10 consecutive sessions with no player-caused deaths. |
| **2** | **Bond & Memory** | Full Bond Engine (all 5 dimensions including bidirectional trust), Companion Profile screen, trait matrix generation, home zone designation, separation timer, reunion event, finding and recruitment sequence. |
| **3** | **Interaction & Dialogue** | Contextual radial wheel (situationally aware), home dialogue system (dynamic topic assembly), Hey button, trust-gated language, equipment and clothing exchange, event hook list, tone buckets, rapport-log referencing. |
| **4** | **Independent Life** | Full personal time activity stack, disposition system, all autonomous base behaviors (cooking, organizing, decorating, map study), Team vs. Independent mode toggle, all optional dependency integrations (True Actions, Comfy Sleeping, True Music). |
| **5** | **Death & Grief** | Warning state sequence, scaled grief moodle system, Keepsake item, Memorial Log, mourning period, Sacrifice Play probabilistic outcomes, continuation configuration, new companion regeneration. |

> **What to build first within Phase 1:** The Threat AI. Not the personality. Not the dialogue. The Threat AI. Build it, test it in a city with 500 zombies, and do not move on until the companion has not caused a player death by reckless behavior in 10 consecutive sessions. That is the bar.

---

## 18 — Technical Architecture

### Language & API

- Lua (Kahlua / B42 API) — no Java modifications, no decompiled class edits
- Workshop-safe and standard-install compatible
- `IsoSurvivor` / `ISCharacter` for NPC base
- `ModData` (`getModData` / `save`) for all persistence
- `Events` (`OnTick`, `OnPlayerUpdate`, etc.) for hooks
- `ISTimedActionQueue` for all companion actions
- `getSquare` / `adjacentFreeTile` for pathfinding
- Native B42 `ModOptions` API for all configuration
- Sandbox options for all gameplay-affecting toggles

### UI Layer

- `ISPanel`-based companion relationship screen (bond tier, rapport log, companion stats)
- Contextual radial wheel — situationally assembled on open
- Home dialogue `ISPanel` with dynamic topic list
- Vanilla text bubble system for ambient commentary
- Equipment exchange via item-handoff interaction, not menus

### Module File Structure

```
media/lua/shared/Companion/
  CompanionCore.lua           — Bond Engine, trait matrix, ModData I/O, bidirectional trust
  CompanionThreatAI.lua       — Zone classification, flee routing, engagement gating
  CompanionDialogue.lua       — Event hooks, tone buckets, rapport-log queries, dynamic topics
  CompanionInteraction.lua    — Radial wheel assembly, Hey button, equipment exchange logic
  CompanionScheduler.lua      — Independent life priority queue, disposition rolls
  CompanionHomeBase.lua       — Zone detection, inventory consciousness, spatial habits
  CompanionDependencies.lua   — Optional mod detection, feature flag management

media/lua/client/Companion/
  CompanionHUD.lua            — Relationship panel, moodle integration, companion status
  CompanionRadial.lua         — Contextual wheel UI, situational option assembly
  CompanionOptions.lua        — ModOptions API registration, Companion Profile screen

media/lua/server/Companion/
  CompanionSpawn.lua          — World placement, recruitment trigger, encounter logic
  CompanionPersist.lua        — Save/load, Memorial Log write, cross-playthrough data
```

### Dependency Detection Pattern

```lua
-- CompanionDependencies.lua
COMPANION_FEATURES = {}

local function detectDependencies()
    COMPANION_FEATURES.trueActions    = getActivatedMods():contains("TMC_TrueActions")
    COMPANION_FEATURES.comfySleep     = getActivatedMods():contains("ComfySleeping")
    COMPANION_FEATURES.trueMusic      = getActivatedMods():contains("TrueMusic")
    COMPANION_FEATURES.dynamicMoodles = getActivatedMods():contains("DynamicMoodles")
    COMPANION_FEATURES.tombsBody      = getActivatedMods():contains("TombsPlayerBody")
end

Events.OnGameStart.Add(detectDependencies)

-- Usage anywhere in the mod:
if COMPANION_FEATURES.trueActions then
    -- queue sitting animation via TrueActions API
else
    -- use vanilla idle animation set
end
```

---

## 19 — Competitive Differentiators

| Feature | Superb Survivors | **Other Half** |
|---|---|---|
| Horde aggro behavior | Frequently causes cascade | Gated. No autonomous gunfire. |
| Flee behavior | Random, often into player | 4-stage priority route to player. |
| Companion count | Many (squad-focused) | One. Depth over breadth. |
| Trust direction | One-way | Bidirectional. Companion trusts player independently. |
| Relationship system | None / hostility probability | 5-dimension Bond Engine, fully persistent. |
| Finding the companion | Spawns randomly | Placed, discoverable, wary, must be earned. |
| Interaction model | Right-click context menu | Context-aware radial + home dialogue system. |
| Equipment exchange | Stats menu | Item handoff with personality-shaped response. |
| Order compliance | Always follows | Trust-gated. Low trust = pushback or refusal. |
| Event memory | None | Rapport Log, 30 named events, referenced in dialogue. |
| Death impact | Frustration or nothing | Scaled grief, keepsake item, memorial log. |
| Player customization | None | Full Companion Profile in character creation. |
| Custom asset requirement | None | None. Zero. Vanilla-complete. |
| B42 support | Broken / not ported | Native B42 target. |

---

## 20 — Closing Statement

The landmark differentiator of Other Half is not any single feature. It is the accumulation of consistent, believable behavior across many hours of play.

A companion who exercises more when stressed. Who sleeps badly after a close call. Who reorganizes the kitchen when you have been gone too long. Who checks the door. Who picks up a book. Who dances when there is music and nobody is watching. Who evaluates your gear against their own body before accepting it. Who pushes back when you give an order they have not yet earned the trust to just take. Who says something quiet when you come back.

That person feels real. And when real people die, it hurts.

Build the Threat AI first. Make it so solid that no one ever loses their companion to a bad pathfind again. Then everything else — the personality, the bond, the dialogue, the grief — lands with the weight it deserves.

---

> *"Did I feel something?"*
> That is the only metric that matters. If the answer is yes, ship it.

---

*Other Half · Project Zomboid Companion Mod · Design Document v2.0 · Build 42 Target*
