# Null Signal — Game Design Specification

> Off-grid skirmish combat in a world that forgot your name

**Setting:** Present-to-future / Hard Sci-Fi / Cyberpunk
**Scale:** 2–15 miniatures per side
**Modes:** PvP, Solo, Co-op
**Tone:** Gritty world, competent operators. Lethal streets, skilled professionals. Dredd meets Ghost in the Shell.
**Deliverables:** Rules webapp (HTML), Warband Forge builder (HTML), Solo Engine (HTML) — all free, single-file apps.
**Miniature-agnostic:** Use any minis. Designed around widely available metal sets of 4–6 models.

---

## 1. Core Stats

Five attributes, each represented by a **die type** (d6 / d8 / d10 / d12). Higher die = better odds.

| Stat | Abbr | Governs |
|------|------|---------|
| Combat | CMB | All attacks — melee and ranged. Weapon determines range/effects, CMB determines dice. |
| Reflexes | REF | Ranged defense (evasion under fire), reactive checks, overwatch triggers. Defenders roll REF to cancel incoming ranged hits. |
| Grit | GRT | Physical/mental toughness. Resisting suppression, morale, enduring hazards. |
| Tech | TEC | Hacking terminals, disarming traps, using specialist gear, operating devices. |
| Awareness | AWR | Spotting hidden enemies, overwatch reactions, detecting ambushes. |

**Resolution:** Roll a pool of dice of the stat's type. Each die showing **4+ = one success.** More successes = better result. The number of dice in the pool depends on context:
- **Combat:** Weapon determines pool size (see Section 6). Stat determines die type.
- **Stat checks:** Roll 2 dice of the stat's type (default). Scenarios may require 1 or 3.
- **Opposed checks:** Both sides roll their stat pool. More successes wins.

**Die probabilities (per die, 4+ success):**
- d6: 50%
- d8: 62.5%
- d10: 70%
- d12: 75%

**Example:** A Leader (CMB d8) firing a Rifle (3 dice) rolls 3d8, needing 4+ per die.

---

## 2. Health System

Health is a **die type** that degrades when an operative takes hits. Inspired by Pulp Alley's health spiral.

### Health Track

```
d10 → d8 → d6 → d4 → DOWN → OUT
```

### Taking Hits

1. Operative takes a hit (from firefight, brawl, hazard, etc.)
2. Roll current health die — **4+ = stay standing**, health die degrades one step
3. Fail = operative goes **Down** (incapacitated, on the ground)
4. A Down operative that takes another hit = **Out** (removed from play)

### Key Rules

- **Stats never degrade.** An operative fights at full capability until they go Down. No tracking injured skills.
- **Down operatives** cannot activate. They may attempt a **Recovery roll** during the End Phase (roll health die at current step, 4+ = get back up at that health level).
- Starting health die depends on tier (see Section 5).

---

## 3. Combat — Simultaneous Resolution

All combat is resolved simultaneously — both sides roll at the same time. No free attacks.

### 3.1 Firefights (Ranged Combat)

1. Active operative declares Shoot action and chooses a target in **line of sight**
2. **Attacker rolls CMB pool** (number of dice = weapon's Attack value, die type = CMB stat)
3. **Defender rolls REF pool** (number of dice = 2, die type = REF stat) — representing evasion, diving for cover, return fire
4. Each attacker success (4+) = one hit. Each defender success (4+) = one hit **cancelled**.
5. Remaining hits trigger health checks on the target
6. **Simultaneously**, the target fires back if they have line of sight — resolve their attack using the same process (they roll CMB, original attacker rolls REF)

**Range Rules:**
- **Line of sight = can shoot.** If you can see them, you can shoot them.
- Each weapon has a **range band** — this is the no-penalty distance
- Shooting **beyond** the range band: shift CMB die type down one step (e.g., d8 → d6)
- Some weapons have **hard range caps** (shotguns, pistols) — cannot fire beyond that distance at all

**Modifiers:**
- Cover: shift attacker's CMB die down one additional step (stacks with range penalty)
- Engaged in melee: -1 die from shooting pool (distracted by close combat)

### 3.2 Brawls (Melee Combat)

1. Active operative moves into base contact and declares Fight action
2. **Both combatants roll CMB pools simultaneously** (dice = melee weapon's Attack value, type = CMB stat)
3. Each success (4+) = one hit on the opponent
4. Resolve hits against both combatants (health checks)
5. Melee weapons may grant bonus dice or special traits

Note: In brawls, REF is not used for defense — both fighters are committed. The mutual exchange IS the defense. REF governs ranged evasion and reactive situations.

### 3.3 Opposed Checks

For non-combat contested actions (e.g., hacking vs. firewall, spotting vs. stealth), both sides roll 2 dice of the relevant stat type. More successes wins. Ties go to the defender/status quo.

---

## 4. Turn Structure

### Round Sequence

Each game lasts **5–6 fixed rounds** (scenario-dependent).

**Phase 1 — Initiative**
Both players roll 1d6. Higher roll activates first. Ties: player with fewer remaining operatives chooses. Re-roll if still tied.

**Phase 2 — Alternating Activations**
Players take turns activating **one operative at a time**. Each activation:
1. **Move** — up to 6" (standard for all operatives)
2. **Perform one Action:**
   - **Shoot** — Engage a target in line of sight (firefight)
   - **Fight** — Engage an adjacent enemy (brawl)
   - **Interact** — Hack a terminal, open a door, pick up an objective, disarm a trap (roll relevant stat per scenario)
   - **Use Gear** — Activate a piece of equipment (medkit, scanner, etc.)

When one player has no remaining operatives to activate, the other player activates all remaining operatives sequentially.

**Phase 3 — End Phase**
1. Down operatives may attempt Recovery rolls
2. Check scenario-specific end-of-round effects
3. Score VP for currently held objectives
4. Advance round counter

---

## 5. Warband Building

### Budget

**Standard game: 10 points.** The Leader is always free. Remaining points buy additional operatives.

No restrictions on tier composition — build whatever crew fits your minis and playstyle.

### Character Tiers

| Tier | Cost | Health | Stat Spread | Gear Slots | Abilities |
|------|------|--------|-------------|------------|-----------|
| **Leader** | Free | d10 | 3×d8, 2×d6 | 2 | 2 |
| **Specialist** | 3 pts | d8 | 2×d8, 3×d6 | 1 | 1 |
| **Operative** | 2 pts | d8 | 1×d8, 4×d6 | 0 | 0 |
| **Recruit** | 1 pt | d6 | 5×d6 | 0 | 0 |

**Stat assignment is free** — the player chooses which of the 5 stats (CMB, REF, GRT, TEC, AWR) receive the d8 slots. This lets the same tier produce very different operatives (a combat-focused Specialist vs. a tech-focused Specialist).

### Example Crews (10 points)

| Crew Style | Composition | Models | Points |
|------------|-------------|--------|--------|
| Elite | Leader + 2 Specialists + 1 Operative | 4 | 10 |
| Balanced | Leader + 1 Specialist + 2 Operatives + 2 Recruits | 6 | 9 |
| Standard | Leader + 1 Specialist + 3 Operatives + 1 Recruit | 6 | 10 |
| Pack-friendly | Leader + 2 Operatives + 4 Recruits | 7 | 8 |
| Horde | Leader + 10 Recruits | 11 | 10 |

### Scalable Games

For larger games, increase the point budget:
- **Small (2–5 models):** 6–8 points, 2'×2' table
- **Standard (5–8 models):** 10 points, 3'×3' table
- **Large (8–15 models):** 15–20 points, 4'×4' table

---

## 6. Weapons

Each operative carries one weapon. Standard weapons are free; premium weapons cost gear slots.

### 6.1 Standard Weapons (Free)

| Weapon | Attack Dice | Range Band | Hard Cap | Trait |
|--------|------------|-----------|----------|-------|
| Pistol | 2 | 8" | 16" | Sidearm (can be taken as secondary) |
| SMG | 3 | 12" | — | Rapid (+1 die within 6") |
| Rifle | 3 | 20" | — | Reliable (no special trait) |
| Shotgun | 3 | 8" | 16" | Spread (+1 die within 6", hard cap) |
| Blade | 2 | Melee | Melee | Melee-only |

### 6.2 Premium Weapons (Cost 1 Gear Slot each)

| Weapon | Attack Dice | Range Band | Hard Cap | Trait |
|--------|------------|-----------|----------|-------|
| Sniper Rifle | 2 | 30" | — | Precise (+1 die beyond range band instead of penalty) |
| LMG | 4 | 20" | — | Suppressive (target at -2" movement next activation) |
| Launcher | 2 | 20" | — | Blast (all models within 2" of target point take hits) |
| Power Weapon | 3 | Melee | Melee | Piercing (target's health die shifts an extra step on failed check) |
| Exotic/Special | Varies | Varies | Varies | Scenario-specific or expansion weapons |

### 6.3 Weapon Notes

- Every operative has a free standard weapon. Premium weapons replace the standard loadout.
- Leaders get 2 gear slots, Specialists get 1. These can be spent on premium weapons or gear items.
- Gear slots not spent on weapons can be spent on equipment (medkits, scanners, hacking decks — see Abilities/Gear).
- Range band = no-penalty distance. Beyond it, shift CMB die down one step. Hard cap = absolute maximum range.
- In line of sight and within range band = roll CMB as normal.
- In line of sight but beyond range band (no hard cap) = shift CMB down one step.
- Beyond hard cap = cannot shoot.

---

## 7. Abilities

A curated list of ~15–20 abilities. Leaders choose 2, Specialists choose 1. Operatives and Recruits have none — they're defined by weapon choice.

### Design Principles

- Each ability has **one clear mechanical effect** — no paragraph-long descriptions
- Abilities are **not tiered by level** — any available ability can be taken by Leaders or Specialists
- Abilities should complement weapons and stats, not replace them

### Ability List (Draft — ~18 abilities)

| Ability | Effect |
|---------|--------|
| **Hardened** | +1 step to GRT die for recovery and morale checks |
| **Overwatch** | May react-shoot when an enemy moves within line of sight (uses AWR to determine if triggered) |
| **CQB Expert** | Bonus die in brawls |
| **Marksman** | Bonus die when shooting within range band |
| **Medic** | Use Gear action: roll TEC to attempt recovery on a Down ally in contact |
| **Hacker** | Bonus die on TEC checks for scenario interactions |
| **Networked** | Once per round, one ally within 6" may re-roll a failed AWR check |
| **Stealthy** | Enemies beyond 12" must pass an AWR check to target this operative |
| **Spotter** | Once per round, designate a target: all friendly shooters gain +1 vs. that target this round |
| **Resilient** | First failed health check each game is treated as a pass (health still degrades) |
| **Tactician** | Once per round, swap activation order with one friendly operative |
| **Rapid Assault** | After a successful Fight action, may move 3" (no additional action) |
| **Suppressive Fire** | Shoot action: instead of damage, target must pass GRT check or lose next action |
| **Breacher** | Ignores cover penalty when shooting |
| **Field Repair** | Use Gear action: roll TEC to restore one health step to a friendly operative in contact |
| **Ghost** | When this operative is hidden, enemies must pass AWR check at -1 to spot them |
| **Heavy Weapons** | May carry one premium weapon without spending a gear slot |
| **Commander** | All friendly operatives within 6" gain +1 to GRT for morale checks |

---

## 8. Solo / Co-op AI System

Designed for solo play and cooperative play against AI-controlled enemy warbands. No cards — tables only.

### 8.1 Behavior Tags

Each AI operative is assigned one of four behavior tags during scenario setup. Tags can be assigned per-model or by role (e.g., "all Recruits are Aggressive, the Leader is Cautious").

| Tag | Personality | Priority |
|-----|-------------|----------|
| **Aggressive** | Closes distance, prefers melee | Get into combat ASAP |
| **Defensive** | Holds position, shoots from cover | Stay in cover, engage at range |
| **Cautious** | Avoids risk, repositions | Stay at maximum range, use cover |
| **Flanker** | Seeks angles, avoids frontline | Move to sides/rear, attack from unexpected angles |

### 8.2 AI Activation

On each AI operative's activation:
1. Roll **d6** on the operative's behavior table
2. Execute the action as written
3. If the action is impossible (e.g., "move toward nearest enemy" but already in contact), default to the next entry down the table

### 8.3 Reaction Tables

**Aggressive**

| d6 | Action |
|----|--------|
| 1–2 | Move toward nearest visible enemy, Fight if in contact |
| 3–4 | Move toward nearest enemy, Shoot if in line of sight |
| 5–6 | Rush nearest enemy (full move + charge into melee) |

**Defensive**

| d6 | Action |
|----|--------|
| 1–2 | If in cover: Shoot nearest visible enemy. If not in cover: move to nearest cover |
| 3–4 | Hold position, Shoot nearest visible enemy |
| 5–6 | Move to better cover within 6", Shoot if possible |

**Cautious**

| d6 | Action |
|----|--------|
| 1–2 | Move away from nearest enemy (maintain range), Shoot if able |
| 3–4 | Move toward nearest objective, Interact if in contact |
| 5–6 | Move to cover with best sight lines, Shoot nearest enemy |

**Flanker**

| d6 | Action |
|----|--------|
| 1–2 | Move toward the flank/rear of nearest enemy, Shoot if able |
| 3–4 | If behind cover with line of sight to enemy rear: Shoot. Otherwise: reposition |
| 5–6 | Move to closest enemy not currently engaged by a friendly, Shoot or Fight |

### 8.4 AI Target Priority

When multiple targets are valid, AI selects by:
1. Nearest visible enemy
2. If tied: lowest health enemy
3. If tied: enemy holding an objective

### 8.5 Solo/Co-op Setup

- **Solo:** Player controls one warband, AI controls the opposing warband(s)
- **Co-op:** Multiple players each control a warband (or split one warband), AI controls the enemy
- **Scaling:** AI warband is built using the same point system. For harder games, give the AI more points or extra operatives
- **Behavior assignment:** Scenario defines default behavior tags, or player rolls randomly per AI model (d6: 1–2 Aggressive, 3–4 Defensive, 5 Cautious, 6 Flanker)

---

## 9. Scenarios & Victory

### 9.1 Victory Points

Every game scores VP from three sources:

**Elimination VP:**
- Take an enemy operative Out = 1 VP
- Take an enemy Leader Out = 2 VP

**Objective VP:**
- Control an objective marker at end of round = 1 VP per marker
- Controlling = at least one friendly operative within 3", no enemy operatives within 3"

**Scenario VP:**
- Each scenario defines bonus VP conditions (complete the hack, extract the VIP, hold the zone for 3 consecutive rounds, etc.)

### 9.2 Game Length and Table Size

| Game Size | Points | Models | Table | Rounds |
|-----------|--------|--------|-------|--------|
| Small | 6–8 | 2–5 | 2'×2' | 5 |
| Standard | 10 | 5–8 | 3'×3' | 6 |
| Large | 15–20 | 8–15 | 4'×4' | 6 |

### 9.3 Scenario Structure

Each scenario defines:
- **Briefing:** Narrative context and objective description
- **Setup:** Table layout, objective placement, deployment zones
- **Behavior tags:** Default AI behavior assignments (for solo/co-op)
- **Scenario VP conditions:** What earns bonus VP
- **Special rules:** Any scenario-specific mechanics (environmental hazards, reinforcements, time pressure)
- **Stat checks:** Which stats get tested by scenario interactions (e.g., "hacking the terminal requires a TEC check")

### 9.4 Starter Scenarios (to be designed)

1. **Blackout** — Seize and hold data relay points in a blacked-out district
2. **Extraction** — Locate and extract a VIP from a hostile zone
3. **Scorched Data** — Race to hack/destroy terminals before the enemy does
4. **Dead Drop** — Retrieve intel packages scattered across the map, exfil to your deployment zone
5. **Signal Jam** — Hold the central transmitter for 3 consecutive rounds to win
6. **Sweep & Clear** — Eliminate all hostiles in a building complex (solo/co-op focused)

---

## 10. Deliverables

### 10.1 Rules Webapp (`NullSignal_Rules.html`)

Single-file HTML application containing the complete rulebook:
- All rules sections as navigable, searchable content
- Reference tables for weapons, abilities, AI behavior
- Quick-reference summary for during play
- Print-friendly layout option
- Mobile-responsive

### 10.2 Warband Forge (`NullSignal_Forge.html`)

Single-file HTML application for building and managing warbands:
- Select tier, assign stats, pick weapons, choose abilities
- Point budget tracking with live validation
- Save/load warbands to localStorage
- Export/import as JSON
- Print roster sheets
- Mobile-responsive

### 10.3 Solo Engine (`NullSignal_SoloEngine.html`)

Single-file HTML application for running AI opponents:
- AI warband builder with behavior tag assignment
- Digital reaction table roller
- AI target priority resolver
- Turn tracker
- Compatible with co-op play

### 10.4 Technical Approach

All apps follow the established pattern from the Pulp Alley Forge and Voidbreak ecosystem:
- Single HTML files, no build step
- React 18 via CDN (UMD)
- Tailwind CSS via CDN
- Babel for JSX processing
- localStorage for persistence
- JSON export/import for sharing

---

## 11. Design Principles

1. **Teach in 10 minutes.** If a rule needs a paragraph to explain, simplify it.
2. **Miniature-agnostic.** Any mini, any manufacturer, any scale (designed for 28mm but works at any).
3. **Metal pack friendly.** A set of 4–6 minis should form a viable crew out of the box.
4. **No cards.** Tables, dice, and reference sheets only.
5. **Flat power curve.** A Recruit can contribute meaningfully. A Leader is better, not invincible.
6. **Simultaneous combat.** Every engagement is a mutual risk. No safe attacks.
7. **Solo-first design.** The AI system isn't an afterthought — it's a core feature.
8. **Free forever.** Rules, apps, and tools are free. No paywalled content.
