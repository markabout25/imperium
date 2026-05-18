# Game Design Document
## *Imperium Populorum* — A Browser-Based Empire Building Game
**Version:** 0.5
**Status:** Canonical reference. Supersedes v0.3 and v0.4. Folds in the resource-pool and philosophy systems introduced in v0.4, and the four §2 design resolutions worked through in May 2026.

---

## 1. Concept & Vision

### Elevator Pitch
Up to 5 players each control a great empire from history, competing across a stylized world map over roughly 200 years. Victory isn't measured by conquest but by **legitimacy** — the empire whose subjects are most prosperous, most culturally integrated, and least likely to rebel wins. The central question the game asks:

> **"Which empire in history would you actually prefer to be conquered by?"**

### What Makes This Game Unique
- **Friend-authored history.** Empires, their abilities, and historical events are contributed by the players before the game begins, through a non-technical contributor portal. The engine runs whatever world the group defines.
- **Time has weight.** Each turn represents 10–25 years. A typical game spans 8–40 turns of sweeping historical change.
- **Legitimacy over dominance.** The most feared empire rarely wins. The best-governed one does.
- **Empires fall but players don't.** A collapsed empire becomes a warlord — diminished but still in play, capable of deciding who wins.
- **Plays across days.** Turns resolve asynchronously over 48-hour windows. No need to coordinate a shared session.

### Design Pillars
- Historical flavour without historical accuracy — the game asks "what if," not "what was"
- Each turn carries weight; each decision should generate a story worth telling later
- Data-driven — the engine adapts to whatever era and empires the group inputs
- Elegant simplicity over layered complexity — mechanics generate stories, not spreadsheets
- Emergent narrative over special-case rules — surprising outcomes arise from existing systems interacting, not from bespoke mechanics added to produce those moments
- Era-neutral event design — no era should structurally favour any one Governing Philosophy

---

## 2. Game Overview

| Property | Value |
|---|---|
| Players | 2–5 major empires, plus collapsed-empire warlords |
| Session length | Multi-day (1–2 weeks typical) |
| Turn window | 48 hours per turn (configurable) |
| Turns | 8–40 turns (each = ~10–25 years) |
| Total span | ~200–400 in-game years |
| Platform | Browser, with Discord bot integration |
| Turn structure | Async simultaneous — each player submits privately; turn resolves when all submit or timer expires |
| Notifications | Discord bot pings on turn open, close, and major events |
| Map | Full world map, divided into five continental zones |
| Era brackets | Three planned (Ancient, Medieval, Early Modern); launching with Early Modern (1300–1648) only |

### Eras and Empire Availability

Three era brackets are planned, each with its own anchor and ripple event pool:

| Bracket | Period | Event pool status |
|---|---|---|
| Ancient World | ~500 BC – 500 AD | Future expansion |
| Medieval | ~700 – 1300 AD | Future expansion |
| Early Modern | 1300 – 1648 AD | **Launch bracket** — delivered as `imperium-events.json` |

An empire is available in a bracket if its historical lifespan overlaps that bracket. So the Byzantine Empire (active 330–1453) is available in both Medieval and Early Modern; classical Rome is available only in Ancient. This permits a player to choose "Rome" in a Medieval game and play it as Byzantium without breaking historical plausibility or the event pool's internal logic.

The launch product ships with the Early Modern bracket only. Medieval and Ancient will be added as expansion packs once the engine is proven through play.

---

## 3. Victory Condition — The Legitimacy Score

The winner is the major empire with the highest **Legitimacy Score** when the game ends.

### Score Composition

| Component | Weight | Measures |
|---|---|---|
| Stability Index | 40% | How unlikely subjects are to rebel |
| Prosperity Index | 40% | Economic wellbeing across the empire |
| Territorial Reach | 20% | Size and strategic value of controlled regions |

**Core design intent:** A small, well-governed empire can beat a vast unstable one. Conquest is easy; making conquered people genuinely accept their fate is hard.

### End Conditions

The game ends in one of three ways, whichever comes first:

1. **Turn limit reached.** Highest Legitimacy Score wins.
2. **Concession Vote.** Triggered when one empire holds a 30%+ Legitimacy lead over the second-place empire for three consecutive turns. Resolved by majority vote of all surviving major empires (warlords do not vote). If the majority agrees, the leading empire wins immediately. If not, play continues and the trigger resets.
3. **Runaway Victory.** Auto-triggered if an empire reaches a 35%+ Legitimacy lead, holds zero Occupied territories, and has positive generation across all three resource pools. No vote required; the game ends.

The two early-end conditions exist so that games don't drag on once an outcome is clear. The Concession Vote in particular is meant to be a moment of formal acknowledgement — the table recognising that one empire has played a better game.

---

## 4. The Map

### Structure

The map covers the full world, divided into **five continental zones**:

- Europe
- Americas
- Africa
- Middle East / South Asia
- East Asia

Each zone contains roughly 3–10 empire-capable home regions and a larger pool of independent territories that can be conquered or diplomatically absorbed. Zones are internally connected by an adjacency graph (land and sea borders within the zone). **Zones are not connected to each other by default** — they are connected only by ocean tiles, which require the Oceanic Threshold (see §4.2) to cross.

### 4.1 Region Properties

Each region carries the following attributes, defined at game setup:

| Property | Description |
|---|---|
| Base Resources | Wealth, Manpower, and Influence generation per turn |
| Population Size | Small / Medium / Large — affects rebellion strength and resource potential |
| Cultural Identity | Tags used to calculate cultural distance against each empire |
| Strategic Value | Optional flags: trade hub, fertile heartland, mountain pass, coastal |
| Special Properties | Optional flags referenced by specific events (e.g. "coastal" for the Age of Discovery) |

### 4.2 The Oceanic Threshold

Ocean crossing is **gated**. Until an empire meets the Oceanic Threshold, the oceans cannot be traversed. To cross an ocean, an empire must hold all of the following simultaneously:

- A coastal Assimilated territory (a stable port)
- A minimum Wealth reserve (the expedition is funded)
- A minimum Influence reserve (the institutional knowledge to navigate)

Specific thresholds will be tuned during playtesting. The first empire to cross opens a permanent ocean route that other empires may follow at reduced cost (representing accumulated maritime knowledge becoming public).

The **Age of Discovery** anchor event, when it fires, significantly lowers the Oceanic Threshold globally — making crossing easier for everyone. Empires that meet the threshold *before* the anchor fires get a head start; this represents an alternate history where a non-European power (the Ming, the Ottomans, the Inca) might have ventured beyond their continent first.

The first ocean crossing of any game is a major narrative event. The Discord bot announces it publicly in the chronicle voice.

### 4.3 Starting Positions

Each empire begins with 1–3 home territories representing its historical heartland. The continental zone of those territories is determined by the empire's submission. All other regions begin as independent nations.

---

## 5. The Empire System

Every empire in the game is defined by the players before the session begins, through the contributor portal. The engine reads this data and runs accordingly.

Each empire has five components.

### 5.1 Identity
- Name, historical period (`from` / `until` years)
- Home regions (1–3)
- Continental zone
- A one-sentence governing philosophy description (flavour text, not mechanical)

### 5.2 Governing Philosophy

Each empire belongs to one of five Governing Philosophies. This is **not** flavour — the philosophy materially affects:

- Which event response choices are discounted or unlocked for the empire
- Which ripple events the empire is vulnerable to
- The shape of crises the empire faces
- Optional victory-condition modifiers (e.g. Thread empires benefit specifically from active diplomatic deals)

The five philosophies:

| Philosophy | Legitimacy through |
|---|---|
| **The Sword** | Force and order |
| **The Pen** | Institutions, law, learning |
| **The Coin** | Trade and prosperity |
| **The Word** | Religion or ideology |
| **The Thread** | Diplomacy and network |

Event pools must be audited for **era-neutrality** — no era should structurally favour one philosophy over another. Era flavour is welcome; era structural bias is not.

### 5.3 Base Stats

Four stats rated 1–5, **summing to exactly 14 points**:

| Stat | Resource generation | Action discount applies to |
|---|---|---|
| Military | Manpower | Conquer, Suppress |
| Economy | Wealth | Invest, Build |
| Culture | Influence (primary) | Assimilate, Empire Ability |
| Diplomacy | Influence (secondary) | Diplomacy, Cultural Exchange |

Stats serve a **dual function**:

1. They set the *generation rate* of the relevant resource pool from controlled territories
2. They *discount action costs* via the formula:

> **Final action cost = Base cost − (relevant stat − 3)**

A stat of 3 is neutral; 5 grants a strong discount; 1 imposes a penalty. This means every point allocation is a meaningful design decision when submitting an empire.

### 5.4 Passive Ability

A permanent rule that applies throughout the game. Examples:
- "Conquered territories assimilate 25% faster"
- "Trade agreements generate double Wealth"
- "Rebellion suppression costs 1 fewer Manpower"

### 5.5 Active Ability

A powerful action usable once every 5 turns. Examples:
- "Conquer any adjacent territory this turn without a military check"
- "All territories gain +1 Stability for 3 turns"
- "Establish a trade mission in a non-adjacent territory"

---

## 6. Resources

Three resource pools drive the game. Each is generated by territories (modified by the relevant stat) and consumed by actions (discounted by the relevant stat). Resources collected at the start of each turn during Phase 2.

| Resource | Primary source | Primary use |
|---|---|---|
| **Wealth** | Trade, cities, tribute (modified by Economy) | Buildings, diplomacy, recruitment |
| **Manpower** | Population, fertile regions (modified by Military) | Armies, conquest, upkeep |
| **Influence** | Culture, religious institutions, courts (modified by Culture / Diplomacy) | Assimilation, diplomatic actions, abilities |

### Resource Scarcity

Resources are finite and contested. A region taken from another empire reduces their output. Managing scarcity — especially Manpower as populations grow — becomes critical as turns advance. Negative resource balances accumulate Strain (see §10).

---

## 7. Turn Structure

Each turn represents ~10–25 years. Turns follow five phases resolved in order after all players have submitted or the 48-hour window expires.

### Phase 1 — The Chronicle
- Current in-game year is displayed
- Any **anchor events** scheduled for this turn fire
- Affected players see response choices on their dashboard

### Phase 2 — Resource Collection
All empires collect Wealth, Manpower, and Influence from controlled territories, modified by:
- Assimilation stage (50% / 75% / 100% efficiency)
- Active event modifiers
- Passive abilities
- Stat multipliers

### Phase 3 — Stability Update
- Resistance ticks in occupied territories (down where invested, flat where neglected, up where exploited)
- Strain indicators recalculate
- Rebellions trigger and resolve if Resistance is at critical levels

### Phase 4 — Player Actions (Simultaneous Resolution)
All submitted actions are revealed and resolved together. Action costs are deducted from the relevant resource pool, discounted by the stat formula.

Available actions:

| Action | Base cost | Discounted by | Description |
|---|---|---|---|
| Move Army | 1 Manpower | Military | Reposition military units |
| Conquer | 3 Manpower | Military | Attack an adjacent territory |
| Invest | 2 Wealth | Economy | Reduce Resistance / improve Prosperity in a territory |
| Build | 3 Wealth | Economy | Permanent territorial improvement |
| Diplomacy | 1 Influence | Diplomacy | Propose a deal to another empire |
| Cultural Exchange | 2 Influence | Diplomacy | Mutual assimilation boost in shared border regions |
| Assimilate | 2 Influence | Culture | Accelerate stage progression in a controlled territory |
| Suppress | 2 Manpower | Military | Reduce rebellion risk in one territory |
| Contract | Free | — | Voluntarily abandon a territory to reduce Strain |
| Empire Ability | varies | Culture | Use the empire's active ability (5-turn cooldown) |
| Warlord Pledge | 1 Influence | Diplomacy | Propose or break a vassal agreement (warlord-only or warlord-as-target) |

Final costs are calculated as: `Base cost − (relevant stat − 3)`, with a minimum of 1 (free actions remain free).

#### Simultaneous Resolution Order

When two empires submit conflicting actions in the same turn (e.g. both attempt to conquer the same independent territory), the engine resolves them in a deterministic order. **The specific resolution order is not yet specified — see §15 Open Items.**

### Phase 5 — Score & Strain Update
- Legitimacy Scores recalculated
- Strain levels updated
- **Ripple events** evaluated against the new game state; any that meet their trigger conditions fire
- Turn resolution summary generated and posted to Discord
- Next turn opens

---

## 8. Conquest & Assimilation

### 8.1 Conquering a Territory
When a territory is conquered, it enters the **Occupied** state:
- Resources collected at 50% efficiency
- Resistance set to High, scaled by cultural distance from the conquering empire
- Rebellion risk begins accumulating

### 8.2 The Three Stages of Control

| Stage | Resource efficiency | Rebellion risk | Legitimacy contribution |
|---|---|---|---|
| **Occupied** | 50% | High | Negative |
| **Integrated** | 75% | Medium | Neutral |
| **Assimilated** | 100% | Low | Positive |

Progression between stages happens automatically over turns and can be accelerated by Invest actions, the Assimilate action, empire abilities, and low cultural distance.

### 8.3 Resistance & Rebellion

Each turn, Resistance in a territory moves based on empire behaviour:
- **Decreases** if the empire invests in the territory, cultural alignment is high, or an empire ability is active
- **Holds flat** if the empire ignores the territory
- **Increases** if the empire extracts resources without investing, or an event drives it up

If Resistance maxes out, a **rebellion** triggers. The territory fights back using its population size as strength. If the empire cannot suppress it (Military stat + units present), the territory becomes independent again and the empire takes a significant Legitimacy hit.

### 8.4 Cultural Distance

Each empire has a Cultural Affinity profile defined at empire creation. Territories with low cultural distance assimilate faster; high cultural distance territories resist longer. This creates natural spheres of influence — expanding into culturally similar regions is always easier.

---

## 9. Diplomacy

Empires negotiate through the in-game diplomacy panel. **All in-game communication is formal action; conversational chatter, threats, jokes, and side conversations happen on Discord** (see §13).

### 9.1 Standard Deal Types

| Deal Type | Effect |
|---|---|
| Non-Aggression Pact | Cannot attack each other for N turns; breaking it costs Legitimacy |
| Trade Agreement | Both empires gain +Wealth per turn for duration |
| Cultural Exchange | Both empires gain assimilation speed in shared border regions |
| Dynastic Marriage | Long-term diplomatic binding; produces specific event hooks |

### 9.2 Pledge Agreements (Warlord Vassalage)

The collapse mechanic (§10) transitions a fallen empire to **warlord status** rather than eliminating them. Warlords can pledge vassalage to any major empire — including the leading one — through a **pledge agreement** with three negotiable terms:

| Term | Default | Range | Description |
|---|---|---|---|
| **Resource share** | 50% | 25%–75% | What percentage of the warlord's territorial output goes to the overlord |
| **Duration lock** | 0 turns | 0–5 turns | Turns during which breaking the pledge costs the warlord Legitimacy, scaled to remaining duration |
| **Named guarantee** | — | Choose one | The overlord's specific commitment to the warlord |

The named guarantee is chosen from a short list:

- **Protection** — overlord commits to defending the warlord's territories. Failing to defend (allowing conquest without contesting it) costs the overlord Legitimacy.
- **Tribute** — overlord pays a fixed resource amount (Wealth, Manpower, or Influence) to the warlord each turn for the duration of the pledge.
- **Honour** — if the overlord wins the game, the warlord receives a named honour at the Verdict of History (e.g. "Loyal Vassal," "Crowned Successor").

All pledge agreements are **public**. The terms are visible to every player. This permits rival empires to counter-offer with better terms, creating competitive bidding for warlord allegiance.

A warlord may break a pledge at any time, but if the duration lock has not yet expired, they take a Legitimacy penalty proportional to remaining locked turns. Breaking a pledge is a public action, narrated by the Discord bot.

### 9.3 Visibility

- **Active deals** (accepted by both parties) are public to all players
- **Rejected deal proposals** remain private to sender and recipient
- **Betrayal** (breaking any active deal) is always public, with Legitimacy penalties scaled to the value of the broken terms

The game tracks every signed and broken deal as part of the empire's diplomatic record. This record is referenced at game end in the Verdict of History.

---

## 10. Strain & Collapse

### 10.1 Strain
Every empire has a **Strain meter** that fills based on:
- Ratio of Occupied (not yet Integrated) territories to total empire size
- Negative resource balances (spending more than collecting across any pool)
- Active rebellions
- Consecutive turns with no Wealth growth

### 10.2 The Stressed State
When Strain reaches 60%, the empire enters **Stressed** state. Visible indicators appear:
- A warning flag on the empire's dashboard
- Private Discord DM to the player: *"Your empire shows signs of strain. Act within the next 2 turns or face collapse."*
- One fewer Action Point next turn (3 → 2)
- All Resistance ticks slow

This gives the player 2–3 turns to respond before collapse: contract voluntarily, invest heavily in unstable territories, suppress active rebellions, or pursue diplomatic relief.

### 10.3 Collapse — Transition to Warlord

If Strain reaches 100% while in Stressed state, the empire **collapses**. The transition:

- **The empire keeps its capital territory automatically**
- **Assimilated territories pass a coin-flip:** each one independently rolls 50% to remain under the warlord, 50% to become an inert no-player Successor State that blocks the map but generates resources for no one
- **All Integrated territories** have a 50% chance of breaking away to become independent again
- **All Occupied territories** immediately become independent again
- The empire **retains its identity** — "the Byzantines, now a minor power in Anatolia" — but is marked as `status: warlord`
- The empire's `collapse` flavour text fires on Discord; the chronicle voice narrates the moment dramatically

Collapse is fast but always foreshadowed. The tragedy is in ignoring the warnings.

### 10.4 Warlord Status

A warlord is no longer competing for the main victory. They cannot win the Legitimacy Score race. But they remain in the game, with a reduced action set and a new goal.

**Available actions:**
- Invest in their own territories
- Suppress rebellions within their domain
- Propose or accept small trade deals (with major empires or other warlords)
- Pledge vassalage to a major empire (§9.2), or break an existing pledge
- Defend their territories if attacked

**Restricted actions:**
- Cannot Conquer territories held by major empires (a warlord lacks the strength for real invasion)
- Cannot use Empire Abilities
- Cannot trigger Concession Votes or Runaway Victory

**Resource generation:** Warlords generate resources from their remaining territories at standard rates but **without philosophy multipliers and without stat-based action discounts**. They play closer to raw numbers than major empires do.

### 10.5 The Verdict of History — Maker of Kings

At game end, the Verdict of History recognises not only the winning empire but also names the **Maker of Kings** — the warlord whose decisions most shaped the outcome. This is scored on a combination of:

- Remaining Legitimacy at game end
- Number of pledge agreements signed during their warlord existence
- Whether any of their allegiance switches were decisive (occurring close to game end, flipping to the eventual winner or away from a would-be winner)

A warlord who collapsed early and rebuilt to a respected minor power, or who played both sides cleverly and tipped the late game, can earn a named place in the game's final story. This gives the warlord a genuine game to play — not a consolation existence, but a different game on a different axis.

---

## 11. The Historical Event System

Events are the primary mechanism through which the contributed history shapes gameplay. Two types exist; both are data-driven and load from JSON.

### 11.1 Anchor Events
Tied to a specific year. They fire automatically when the game reaches that turn, regardless of game state.

Anchor events typically affect all empires but with **differential impacts** based on Governing Philosophy. They include:
- Chronicle text shown to all players when the event fires
- Mechanical effects (modifiers, durations, scaling rules)
- Response choices (typically 3) that affected empires must select between
- A `philosophy_impact` mapping describing asymmetric effects
- A `discord_announcement` string in the chronicle voice
- An optional `flavour_quote`

The Early Modern event pool is delivered as `imperium-events.json` and includes the Black Death, the Hundred Years War, the Fall of Constantinople, the Printing Press, the Reformation, the Age of Discovery, the Sack of Rome, and the Peace of Westphalia.

### 11.2 Ripple Events
Triggered by game state, not the calendar. They fire when specific conditions are met during Phase 5.

Ripple events typically affect only the empire whose actions (or inaction) triggered them. They include:
- Trigger conditions (a structured rule set, evaluated each Phase 5)
- Mechanical effect
- Frequency limit (once per game, once per empire, once every N turns)
- Historical reasoning
- `discord_announcement` and `flavour_quote`

**Design intent:** Ripple events represent historical patterns — overextension, cultural diffusion, dynastic crisis, economic instability. Players who neglect their territories or exhaust their resources face systemic consequences, not just strategic ones.

### 11.3 Era-Neutrality

When new event pools are written (Medieval, Ancient), they must be audited for era-neutrality: the `philosophy_impact` fields across the whole pool should reach rough parity across the five Governing Philosophies. Era flavour is welcome; era structural bias is not.

---

## 12. Turn Resolution Visibility

After a turn resolves, **every player sees every submitted action and its outcome.** This is full transparency. Failed attempts, abandoned plans, and actions that targeted players who didn't realise they were targeted are all visible.

**The one exception:** diplomatic proposals that were rejected remain private to sender and recipient. Once a proposal is accepted, it becomes a public active deal. Rejected proposals are like private letters — they happened, but no third party needs to know they happened.

The Discord bot **narrates rather than enumerates**. Instead of `Ottoman: Conquer(Hungary). Result: failed.`, the turn summary reads: *"The Ottoman army crossed the Danube in force, but Habsburg defences held the line at Mohács."* Same information, better register. The narration layer is where the formal-and-witty voice does its work.

---

## 13. Async Multiplayer & Discord Integration

### 13.1 The Turn Cycle

```
Turn opens → Discord bot notifies all players
        ↓
Players log in at their convenience (48-hour window)
        ↓
Each player sees the current map, their resources, the Chronicle
        ↓
Each player plans and submits actions privately
        ↓
Once all players submit OR 48 hours expire:
        ↓
Server resolves all actions through the five phases
        ↓
Turn resolution summary generated (narrated, not enumerated)
        ↓
Discord bot posts summary publicly + DMs individual players where attention needed
        ↓
Next turn opens immediately
```

### 13.2 Default Action

If a player does not submit within 48 hours, their empire takes a default action:
- Invest 1 Wealth in the territory with highest Resistance
- Hold all armies in place
- No conquests, no diplomacy

Three consecutive default turns triggers a **Neglect Warning**. A fourth missed turn pushes the empire into Stressed state regardless of Strain level. This is a social mechanic as much as a game mechanic — the friend group should know when one of them has gone quiet.

### 13.3 Discord as Communication Layer

The Discord bot is the game's primary communication layer. The bot is the **voice of the chronicle**, not a system notification engine — its output should read like an in-character historian-narrator, not a status update.

| Bot Action | Trigger |
|---|---|
| "Turn N is now open — you have 48 hours" | Turn opens |
| Private DM: Strain warning, response choice needed, diplomatic proposal received | Player attention needed |
| "⚠ 12 hours remaining — [Player] has not yet submitted" | Deadline approaching |
| Full turn resolution summary (narrated) | Turn closes and resolves |
| "⚔ [Empire A] has conquered [Region] from [Empire B]" | Major conquest |
| "🔥 [Empire] is showing signs of strain" | Strain reaches 60% |
| "💀 The [Empire] has fallen. [Collapse flavour text]" | Collapse event |
| "🏴 [Warlord] has pledged to [Empire]. Terms: [share / lock / guarantee]" | Pledge agreement signed |
| "📜 [Anchor Event] fires: [chronicle text]" | Anchor event triggers |
| "🌊 [Empire] has crossed the ocean. The world is changed." | First ocean crossing |

**Side conversations** — chatter, threats, jokes, the actual texture of friendship and rivalry — happen on Discord, not in the game. The game UI handles formal actions; Discord handles everything else. The bot announces formal moves in the chronicle voice so the conversation has natural beats to follow.

### 13.4 Player Dashboard

Each player has a private dashboard showing:
- Current map state (full view, including all zones)
- Their empire's resources, action points, Strain level
- Their territories and assimilation stages
- Active diplomatic deals (public)
- Pending diplomatic proposals (private to them)
- Active pledge agreements where they are overlord or warlord
- Action submission form for the current turn
- Turn history (all previous resolution summaries)

### 13.5 Game Setup Flow

1. Host creates a game and selects the era bracket (currently Early Modern only)
2. Host shares an invite link
3. Each player claims an empire from those available in the chosen bracket
4. Host confirms the lineup and starts the game
5. Discord bot is invited to the group's server and configured with the game channel
6. Turn 1 opens automatically

### 13.6 Session Pacing Options

The host can configure the turn window at game creation:

| Mode | Turn Window | Typical Game Length |
|---|---|---|
| Standard | 48 hours | 2–3 weeks |
| Fast | 24 hours | 1–2 weeks |
| Leisurely | 72 hours | 3–5 weeks |
| Weekend | Opens Friday, closes Sunday | Months |

---

## 14. Data Schemas

### 14.1 Empire (from contributor portal)

```
type: "empire"
author: string
name: string
from: integer (start year)
until: integer (end year)
regions: string (1–3 home regions, comma-separated)
continental_zone: enum [Europe, Americas, Africa, Middle_East_South_Asia, East_Asia]
era_bracket: enum [Ancient, Medieval, Early_Modern] (multiple permitted if lifespan overlaps)
philosophy: string (one-sentence flavour)
governing_philosophy: enum [Sword, Pen, Coin, Word, Thread]
stats: { military, economy, culture, diplomacy }  // integers, sum = 14
passive: string (passive ability, plain English)
active: string (active ability, plain English)
affinity: string (cultural affinity description)
collapse: string (optional dramatic collapse text)
notes: string (historical notes)
timestamp: integer (millis since epoch)
```

**Portal additions required for v0.5:** `continental_zone`, `era_bracket`, `governing_philosophy` dropdowns. The `era_bracket` field should be multi-select where applicable (the Byzantines, for example, would tag both Medieval and Early Modern).

### 14.2 Anchor Event

Schema defined in `imperium-events.json`. Key fields: `id`, `name`, `year`, `affected`, `chronicle`, `mechanical_effect` (structured), `response_choices` (array), `differential`, `philosophy_impact` (machine-readable), `flavour_quote`, `discord_announcement`.

The portal currently captures a flatter version. The dev team will either need a translation layer between portal submissions and the engine format, or the portal must be extended to capture response choices and philosophy impact directly. **Recommended: extend the portal.**

### 14.3 Ripple Event

Schema defined in `imperium-events.json`. Like anchor events but with `trigger_condition` (structured rule set) instead of `year`, and `frequency` (once per game / once per empire / every N turns / etc.).

The trigger format uses a rules array supporting `operator: "OR"`, comparison operators, and named conditions like `anchor_printing_press_has_fired`. The engine needs a rule evaluator that reads these rules and checks them against game state at the end of each Phase 5.

### 14.4 Region / Map Schema

Each region requires:
- `id`, `name`, `continental_zone`
- `adjacencies` — array of region IDs within the same zone
- `ocean_adjacencies` — array of region IDs in other zones, traversable only when the Oceanic Threshold is met
- `base_resources` — { wealth_generation, manpower_generation, influence_generation }
- `population_size` — enum [Small, Medium, Large]
- `cultural_identity` — array of cultural identity tags
- `strategic_value` — array of flags [trade_hub, fertile_heartland, mountain_pass, coastal, ...]
- `starting_owner` — empire ID, "independent", or null

**The initial map dataset must be authored** before the engine can run a game. Estimated scope: ~50 regions across five zones, plus ocean adjacency edges. This is a content task, not an engineering task; it can be authored by the friend group through an extended portal form or directly as JSON.

---

## 15. Open Items

These are unresolved items that block Milestone 0 or near-term work:

1. **Simultaneous-action conflict resolution order.** When two empires submit actions targeting the same territory in the same turn, what is the deterministic resolution sequence? Options include: highest relevant stat wins; submission timestamp wins; randomised; defender priority. **Not yet decided.**

2. **Initial map dataset.** Roughly 50 regions across five continental zones, with full adjacency graphs and ocean edges. To be authored. Could be a friend-group contribution task once the portal is extended.

3. **Oceanic Threshold values.** Specific Wealth and Influence reserve thresholds required to cross before vs. after the Age of Discovery anchor fires. To be tuned in playtesting; placeholder values needed for initial implementation.

4. **Concession Vote voting weights.** All surviving major empires vote with equal weight, or is the vote weighted by current Legitimacy? Default assumption: equal weight, simple majority.

5. **Pledge agreement Legitimacy values.** Specific Legitimacy penalty for breaking a pledge with N turns remaining on the duration lock; specific Legitimacy penalty on the overlord for failing a Protection guarantee. To be tuned in playtesting.

---

## 16. Development Roadmap

### Milestone 0 — Infrastructure Foundation
*Prerequisite for everything else. Do not begin game logic until this lands.*

- Firebase Realtime Database setup (project: `imperium-populi-1d8ec`)
- Player accounts (auth, persistent identity)
- Game session creation and invite flow
- Persistent server-side game state
- Discord bot scaffold (notifications in/out, DMs)

### Milestone 1 — Data-Driven Content Layer
- Schemas: empire, anchor event, ripple event, region/map
- Import pipeline from contributor portal to game database
- Contributor portal extensions: continental zone, era bracket, governing philosophy dropdowns
- Game setup flow: choose era bracket, filter available empires and events, assign to players
- Initial map dataset authored

### Milestone 2 — Core Turn Engine
- Five-phase turn engine
- Resource collection with stat multipliers
- Action submission with AP / stat discount formula
- Simultaneous action resolution (decision needed — see §15)
- Conquest mechanic
- Assimilation stage progression
- Legitimacy Score calculation
- Concession Vote and Runaway Victory triggers

### Milestone 3 — Historical Event Engine
- Anchor event firing (year-triggered)
- Ripple event evaluation (condition-triggered)
- Response choice presentation
- Philosophy differential application
- Event narration in turn resolution summary

### Milestone 4 — Empire Identity & Collapse
- Passive ability engine
- Active ability engine
- Strain meter and Stressed state
- Collapse resolution
- Warlord state transition (identity preserved, reduced action set)
- Maker of Kings tracking for Verdict of History

### Milestone 5 — Async Turn Management
- 48-hour turn window with configurable modes
- Default action system
- Neglect warning system
- Discord bot: full event vocabulary (per §13.3 table)
- Player dashboard

### Milestone 6 — Diplomacy & Pledges
- Standard deal proposals and acceptance flow
- Pledge agreement proposals with three negotiable terms
- Public deal ledger
- Betrayal detection and Legitimacy penalty calculation
- Counter-offer mechanism for competitive pledge bidding

### Milestone 7 — Ocean Crossing
- Oceanic Threshold evaluation
- First-crossing announcement (chronicle voice)
- Permanent ocean route opening
- Age of Discovery anchor integration

### Milestone 8 — Polish & Launch
- Full map UI with zone visualisation and ocean tiles
- Turn history and replay view
- Leaderboard and score history chart
- "The Verdict of History" game-end summary, including Maker of Kings
- Mobile-friendly layout

---

## 17. Working Principles

These principles emerged through iteration and override default instincts:

- **Elegant simplicity over layered complexity.** Mechanics generate stories, not spreadsheet management. If a new mechanic is proposed, the question to ask is: does this produce a better story at the table, or just more numbers?
- **Emergent narrative over special-case rules.** Surprising outcomes should arise from existing systems interacting. If a desirable behaviour requires a new rule, first ask whether existing rules can be tuned to produce it instead.
- **Era-neutrality.** Event pools must reach rough philosophy parity. Era flavour is welcome; era structural bias is not.
- **Data-driven, no hardcoding.** Empires, events, regions, and map data load from JSON. The friends shape the world; the engine reads.
- **Visible consequences.** Strain, rebellion build-up, ripple triggers — all visible before they bite. The drama is in seeing the cliff edge coming, not in being shoved off it.
- **Tone and register.** In-game text reads like a 19th-century historian with a sense of humour. Look at the chronicle fields in `imperium-events.json` for the target voice.

---

*This document is the canonical reference. v0.3 is superseded and should be archived. Update as decisions are made during playtesting.*
