# Game Design Document
## "Imperium Populorum" — A Browser-Based Empire Building Game
**Version:** 0.3
**Status:** Foundation Document — Open for Iteration

---

## 1. Concept & Vision

### Elevator Pitch
Up to 5 players each control a great empire from history, competing across a stylized world map over roughly 200 years. Victory isn't just about conquest — it's about *legitimacy*. The empire whose subjects are most prosperous, most culturally integrated, and least likely to rebel wins.

The central question the game asks: **"Which empire in history would you actually prefer to be conquered by?"**

### What Makes This Game Unique
- **Friend-authored history.** Empires, their abilities, and historical events are contributed by the players before the game is set up — not hardcoded. The engine runs whatever world your group defines.
- **Time has weight.** Each turn represents 10–25 years. A 200-year game spans 8–20 turns of sweeping historical change.
- **Legitimacy over dominance.** The most feared empire rarely wins. The best-governed one does.
- **Empires can fall.** Overextension, rebellion, and economic ruin are real threats — not just flavour.
- **Plays across days.** A full game unfolds asynchronously — players submit their turns when they have time, over hours or days. No need to coordinate a shared session window.

### Design Pillars
- Historical flavour without historical accuracy — the game asks "what if," not "what was"
- Each turn feels weighty — 30–40 turns over a multi-day session, each representing a decade of history
- Data-driven — the game engine adapts to whatever era and empires the group inputs
- Built for remote async play, no installation needed, notifications via Discord

---

## 2. Game Overview

| Property | Value |
|---|---|
| Players | 2–5 |
| Session length | Multi-day (1–2 weeks typical) |
| Turn window | 48 hours per turn |
| Turns | 30–40 turns (each = ~10 years) |
| Total span | ~300–400 in-game years |
| Platform | Browser (text-based with styled UI) |
| Turn structure | Async simultaneous — each player submits privately, turn resolves when all submit or timer expires |
| Notifications | Discord bot pings when a new turn opens and when it resolves |
| Map | Stylized world map with named real-world regions, adapted to chosen era |

### Choosing an Era
Before the game begins, the group agrees on a **start year**. This determines:
- Which empires are available (only those that existed at or near that date)
- Which anchor events are active (only those that fall within the game's timespan)
- The starting configuration of independent nations on the map

Examples of valid start years: 100 AD, 750 AD, 1200 AD, 1400 AD, 1600 AD.

---

## 3. Victory Condition — The Legitimacy Score

The winner is the empire with the highest **Legitimacy Score** when the game ends. The game ends when either the final turn is reached, or only one empire remains standing.

### Score Composition

| Component | Weight | Measures |
|---|---|---|
| Stability Index | 40% | How unlikely your subjects are to rebel |
| Prosperity Index | 40% | Economic wellbeing across your empire |
| Territorial Reach | 20% | Size and strategic value of controlled regions |

**Core design intent:** A small, well-governed empire can beat a vast unstable one. Players must balance expansion with the quality of their rule. Taking territory is easy. Making conquered people genuinely accept their fate is hard.

---

## 4. The Map

### Structure
The world map is divided into named **regions** based on real historical geography, scaled to the chosen era. Each region is either:
- **Controlled by a major empire** (one of the 2–5 players)
- **An independent nation** (neutral, can be conquered or diplomatically absorbed)

Regions connect by adjacency — expansion requires a land or sea border with a territory you already hold.

### Region Properties
Each region has the following attributes, defined at game setup:

| Property | Description |
|---|---|
| Base Resources | Food, Wealth, Production output per turn |
| Population Size | Small / Medium / Large — affects rebellion strength and prosperity potential |
| Cultural Identity | Determines assimilation difficulty for each empire |
| Strategic Value | Some regions carry bonus multipliers (trade hubs, fertile heartlands, mountain passes) |

### Starting Positions
Each empire begins with 1–3 home territories representing their historical heartland. All other regions begin as independent nations.

---

## 5. The Empire System

Every empire in the game is **defined by the players before the session begins**, using the Contribution Guide (see separate document). The game engine reads this data and runs accordingly.

Each empire has four components:

### 5.1 Identity
- Name, historical period, home region(s)
- A one-line governing philosophy (flavour only, but sets tone)

### 5.2 Base Stats
Four stats rated 1–5:

| Stat | What It Affects |
|---|---|
| Military | Conquest success rate, rebellion suppression |
| Economy | Resource output, trade income |
| Culture | Assimilation speed, cultural gravity on neighbours |
| Diplomacy | Deal effectiveness, absorption of independent nations |

Stats must sum to exactly 14 points. This ensures all empires are balanced at the start regardless of who submits them.

### 5.3 Passive Ability
A permanent rule that applies throughout the game. Examples:
- "Conquered territories assimilate 25% faster"
- "Trade agreements generate double Wealth"
- "Rebellion suppression costs 1 fewer Production"

### 5.4 Active Ability
A powerful action usable once every 5 turns. Examples:
- "Conquer any adjacent territory this turn without a military check"
- "All territories gain +1 Stability for 3 turns"
- "Establish a trade mission in a non-adjacent territory"

---

## 6. Core Resources

Three resources drive the economy, collected at the start of each turn:

| Resource | Primary Source | Primary Use |
|---|---|---|
| Food | Fertile regions, farms | Population growth, army upkeep, famine prevention |
| Wealth | Trade routes, cities, tribute | Buildings, diplomacy, recruitment |
| Production | Industrial and mineral regions | Infrastructure, military units, development |

### Resource Scarcity
Resources are finite and contested. A region taken from another empire reduces their output. Managing scarcity — especially Food — becomes critical as turns advance and populations grow.

---

## 7. Turn Structure

Each turn represents ~10 years. Turns follow five phases resolved in order after all players have submitted.

### Phase 1 — The Chronicle (Era Marker)
The game displays the current in-game year and announces any **anchor events** that fire this turn. Shown prominently in the turn resolution summary sent to all players.

### Phase 2 — Resource Collection
All empires collect Food, Wealth, and Production from territories they control, modified by development level and any active events.

### Phase 3 — Stability Update
- Resistance in occupied territories ticks down (assimilation progresses passively)
- **Strain Indicators** appear if any empire is approaching collapse thresholds
- Rebellions trigger and resolve if resistance has reached critical levels

### Phase 4 — Player Actions (Simultaneous Resolution)
All player actions submitted during the turn window are revealed and resolved together. Each player has **3 Action Points (AP)** per turn as a base.

| Action | AP Cost | Description |
|---|---|---|
| Move Army | 1 | Reposition military units to an adjacent territory |
| Conquer | 2 | Attack an adjacent territory; outcome depends on Military stat + units vs. defence |
| Invest | 1 | Spend Wealth to improve Prosperity or reduce Resistance in a territory |
| Build | 2 | Permanently improve a territory's output or assimilation rate |
| Diplomacy | 1 | Propose a deal to another empire (trade, non-aggression, vassal) |
| Contract | 1 | Voluntarily abandon a territory to reduce overextension strain |
| Empire Ability | varies | Use your empire's unique active ability |

### Phase 5 — Score & Strain Update
Legitimacy Scores are recalculated. Strain levels are updated. The leaderboard is shown. Any **ripple events** triggered by this turn's game state are announced. A full turn resolution summary is generated and sent to all players via Discord.

---

## 8. The Historical Event System

Events are the primary way your friends shape the game. There are two types:

### 8.1 Anchor Events
Fixed to a specific year. They fire automatically when the game reaches that turn.

Anchor events can affect all empires equally, or target specific empire types (e.g. "empires with more than 8 territories").

### 8.2 Ripple Events
Triggered by game state, not the calendar. They fire when a specific condition is met during any Phase 5 update.

**Design note:** Ripple events create emergent historical logic. Players who expand too fast face systemic consequences, not just strategic ones — which is historically accurate.

---

## 9. Conquest & Assimilation

### 9.1 Conquering a Territory
When a territory is conquered it enters the **Occupied** state:
- Resources collected at 50% efficiency
- Resistance set to High (scaled by cultural distance from the conquering empire)
- Rebellion risk begins accumulating

### 9.2 The Three Stages of Control

| Stage | Resource Efficiency | Rebellion Risk | Legitimacy Contribution |
|---|---|---|---|
| Occupied | 50% | High | Negative |
| Integrated | 75% | Medium | Neutral |
| Assimilated | 100% | Low | Positive |

Progression between stages happens automatically over turns. It can be accelerated by investing in a territory or using empire abilities.

### 9.3 Resistance & Rebellion
Each turn, resistance in a territory moves based on empire behaviour:

- **Decreases** if: empire invests in territory, cultural alignment is high, or empire ability is active
- **Holds flat** if: empire ignores territory
- **Increases** if: empire exploits territory (extracts resources without investment), or an anchor/ripple event fires

If resistance maxes out, a **rebellion** triggers. The territory fights back using its population size as strength. If the empire cannot suppress it (Military stat + units present), the territory is lost and becomes independent again. The empire takes a significant Legitimacy Score hit.

### 9.4 Cultural Distance
Each empire has a Cultural Affinity profile (defined at empire creation). Territories with low cultural distance assimilate faster; high cultural distance territories resist longer. This creates natural spheres of influence — expanding into culturally similar regions is always easier.

---

## 10. Collapse System

### 10.1 Strain
Every empire has a **Strain meter** that fills based on:
- Number of Occupied (not yet Integrated) territories relative to empire size
- Negative resource balance (spending more than collecting)
- Active rebellions
- Consecutive turns without Wealth growth

### 10.2 Warning Signs
When Strain reaches 60%, the empire enters **Stressed** state. Visible indicators appear:
- A warning flag on the empire's status panel
- Discord notification sent directly to that player: *"Your empire shows signs of strain. Act within the next 2 turns or face collapse."*
- Reduced AP next turn (3 → 2)
- All Resistance ticks slow

This gives the player 2–3 turns to respond: contract voluntarily, invest heavily, or suppress.

### 10.3 Collapse
If Strain reaches 100% while in Stressed state:
- The empire **collapses** immediately at the end of that phase
- All Occupied territories become independent nations
- All Integrated territories have a 50% chance of breaking away
- All Assimilated territories remain under a **Successor State** (controlled by no player — generates resources for no one but blocks the map)
- The player is eliminated
- A dramatic collapse summary is posted to the Discord channel for all to see

Collapse is fast — but it was visible coming. The tragedy is in ignoring the warnings.

---

## 11. Diplomacy

Empires can negotiate directly each turn via the action system. In async play, diplomatic proposals are sent to the target player's dashboard and must be accepted or rejected before they submit their turn.

| Deal Type | Effect |
|---|---|
| Non-Aggression Pact | Cannot attack each other for N turns; breaking it costs Legitimacy |
| Trade Agreement | Both empires gain +Wealth per turn for duration |
| Vassal Arrangement | Smaller empire acknowledges dominance; gains protection, loses 1 AP |
| Cultural Exchange | Both empires gain assimilation speed in shared border regions |

All deals are public. Betrayal always costs Legitimacy — the game tracks your word as part of your legacy.

---

## 12. Async Multiplayer System

This section defines how the game operates across days rather than a single session.

### 12.1 The Turn Cycle

```
Turn opens → Discord bot notifies all players
        ↓
Players log in at their convenience (48-hour window)
        ↓
Each player sees the current map, their resources, and the Chronicle
        ↓
Each player plans and submits their actions privately
        ↓
Once all players submit OR 48 hours expire:
        ↓
Server resolves all actions simultaneously
        ↓
Turn resolution summary generated (what happened, who won what, events fired)
        ↓
Discord bot posts summary to group channel + notifies each player
        ↓
Next turn opens immediately
```

### 12.2 The Default Action Rule
If a player does not submit within 48 hours, their empire takes a **default action** automatically:
- Invest 1 AP in the territory with highest Resistance
- Hold all armies in place
- No conquests or diplomacy

This keeps the game moving without punishing absent players too harshly. Three consecutive default turns triggers a **Neglect Warning** — if the player misses a fourth, their empire enters Stressed state regardless of Strain level. This is a social mechanic as much as a game mechanic.

### 12.3 Discord Integration

The Discord bot serves as the game's primary communication layer:

| Bot Action | Trigger |
|---|---|
| "Turn N is now open — you have 48 hours" | Turn opens |
| "⚠ 12 hours remaining — [Player] has not yet submitted" | 12 hours before deadline |
| "Turn N has resolved — here's what happened" | Turn closes and resolves |
| "⚔ [Empire A] has conquered [Region] from [Empire B]" | Major conquest event |
| "🔥 [Empire] is showing signs of strain" | Strain reaches 60% |
| "💀 The [Empire] has collapsed" | Collapse event |
| "📜 [Anchor Event] fires: [description]" | Anchor event triggers |

The full turn resolution summary is always posted publicly. Individual empire dashboards (private) are accessed via a link in the bot's DM to each player.

### 12.4 Player Dashboard
Each player has a private dashboard showing:
- Current map state (full view)
- Their empire's resources, AP, and Strain level
- Their territories and assimilation stages
- Active diplomatic deals
- Pending diplomatic proposals from other empires
- Action submission form for the current turn
- Turn history (all previous resolution summaries)

### 12.5 Game Setup Flow
1. One player (the host) creates a game and sets the start year
2. Host shares an invite link to the group
3. Each player claims an empire from those available for the chosen era
4. Host confirms the lineup and starts the game
5. Discord bot is invited to the group's server and configured with the game channel
6. Turn 1 opens automatically

### 12.6 Session Pacing Options
The host can configure the turn window at game creation:

| Mode | Turn Window | Typical Game Length |
|---|---|---|
| Standard | 48 hours | 2–3 weeks |
| Fast | 24 hours | 1–2 weeks |
| Leisurely | 72 hours | 3–5 weeks |
| Weekend | Opens Friday, closes Sunday | Months |

---

## 13. Open Design Questions

To discuss with your group before development begins:

1. **Successor states:** When an empire collapses, should a player be able to re-enter as a new rising empire, or are they eliminated for the rest of the game?
2. **Map scope:** Full world map, or region-locked (e.g. just Eurasia, just the Mediterranean)?
3. **Empire count flexibility:** Should the game support 2-player duels, or is 3–5 the minimum for the map to feel alive?
4. **Diplomatic negotiation:** Should players be able to send private messages to each other through the game, or is all communication through Discord outside the game?
5. **Turn resolution visibility:** Should each player see only their own actions before resolution, or can they see all submitted actions once the turn closes?

---

## 14. Development Roadmap

### Milestone 0 — Infrastructure Foundation
*Must be done before any game logic. Everything else depends on this.*
- [ ] Set up Supabase (database + auth)
- [ ] Player account system (sign up, log in, persistent identity)
- [ ] Game session creation and invite flow
- [ ] Persistent game state stored server-side (not in browser)
- [ ] Basic Discord bot setup (notifications in/out)

### Milestone 1 — Data-Driven Content Layer
- [ ] Empire data schema (stats, abilities, era, home regions)
- [ ] Event data schema (anchor and ripple, with trigger conditions)
- [ ] Map region schema (resources, population, cultural identity)
- [ ] Import pipeline: read `contributions.json` from the contributor portal into the game database
- [ ] Game setup flow: choose era, filter available empires and events, assign to players

### Milestone 2 — Core Turn Engine
- [ ] Turn phase engine (5 phases in order)
- [ ] Resource collection system
- [ ] Action submission system with AP budget
- [ ] Simultaneous action resolution
- [ ] Turn resolution summary generator
- [ ] Conquest mechanic
- [ ] Assimilation stage progression
- [ ] Legitimacy Score calculation

### Milestone 3 — Historical Event Engine
- [ ] Anchor event system (year-triggered, fires during Phase 1)
- [ ] Ripple event system (condition-triggered, fires during Phase 5)
- [ ] Event narrative display in resolution summary

### Milestone 4 — Empire Identity & Collapse
- [ ] Passive ability engine
- [ ] Active ability engine
- [ ] Strain meter and Stressed state
- [ ] Collapse resolution and successor state generation
- [ ] Cultural distance modifiers on assimilation

### Milestone 5 — Async Turn Management
- [ ] 48-hour turn window with configurable modes
- [ ] Default action system for absent players
- [ ] Neglect warning system
- [ ] Discord bot: turn open/close notifications
- [ ] Discord bot: event and collapse announcements
- [ ] Discord bot: 12-hour deadline reminders
- [ ] Player dashboard (private per-player view)

### Milestone 6 — Diplomacy
- [ ] Deal proposal and acceptance flow (async-compatible)
- [ ] Active deal tracking and expiry
- [ ] Betrayal detection and Legitimacy penalty
- [ ] Public deal ledger visible to all players

### Milestone 7 — Polish & Launch
- [ ] Full map UI with territory ownership and assimilation stage visualisation
- [ ] Turn history and replay view
- [ ] Leaderboard and score history chart
- [ ] Game end summary ("The Verdict of History")
- [ ] Mobile-friendly layout

---

*This document is a living design reference. Update it as decisions are made during playtesting.*
