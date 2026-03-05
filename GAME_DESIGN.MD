# GAME_DESIGN.MD — AsteroidGame Story, Lore & Design

> This file contains all narrative, design, and balance information.
> For technical architecture, autoloads, scene trees, and signal contracts see `CLAUDE.MD`.
> For task tracking and backlog see `PROJECTSTATUS.MD`.

---

## Story Synopsis

### The Meridian

The colony ark *Meridian* was humanity's last hope — a generation ship carrying 12,000 souls in cryosleep toward a habitable world catalogued as Kepler-442b. Halfway through the 200-year voyage, a dense debris field tore through the ship's outer hull, shredding critical systems.

The ship's AI, **ARIA** (Autonomous Repair & Intelligence Assistant), could keep the reactors warm and the cryopods sealed, but lacked the physical capability to gather raw materials for repairs. With no other option, ARIA initiated an emergency de-icing protocol: revive one passenger — the one whose skill profile best matched "resourceful under pressure."

That passenger is **the Pilot**.

### The Pilot

The player character. No fixed name — referred to as "Pilot" by ARIA. A former asteroid surveyor from the Belt colonies, chosen specifically because of their experience navigating debris fields and operating mining equipment.

The Pilot wakes alone in a damaged shuttle bay. ARIA explains the situation: the Meridian is crippled, drifting through an asteroid-rich region of space. The ship needs raw materials — iron for hull patches, crystal for electronics, titanium for structural supports, uranium for the reactor. The Pilot must take a small mining shuttle out into the field, gather what they can, and bring it back.

But they are not alone. An unidentified vessel — another ark ship — has been shadowing the Meridian. Its intentions are unknown, but its scout drones have already been detected in the debris field.

### Tone & Atmosphere
- **Isolation**: The Pilot is alone. ARIA is a voice, not a companion.
- **Urgency**: 12,000 lives depend on the Pilot's runs.
- **Hope**: Each repair milestone brings the ship closer to salvation.
- **Escalating threat**: The enemy ark's presence grows as the Meridian powers up.
- **Inspired by**: FTL's tension + Asteroids' simplicity + Deep Rock Galactic's run-and-bank loop.

---

## Narrative Milestones

Story beats are triggered when the player reaches specific Mothership Research tiers. Each milestone delivers dialogue from ARIA and may unlock new mechanics.

| # | Milestone ID       | Trigger                    | Title              | Description |
|---|-------------------|----------------------------|--------------------|-------------|
| 1 | `awakening`       | Reactor Tier 1             | "Awakening"        | ARIA brings the reactor online. Lights flicker to life. The Meridian's systems begin to hum. ARIA explains the full scope of the damage and the Pilot's mission. |
| 2 | `first_breath`    | Life Support Tier 1        | "First Breath"     | Air recyclers restored. ARIA reports crew vital signs stabilising in cryosleep. Unlocks weapon systems — "We're not alone out here." |
| 3 | `patching_wounds` | Hull Integrity Tier 2      | "Patching the Wounds" | Hull breaches sealed. The Meridian stops venting atmosphere. Ship is structurally stable for the first time since the incident. |
| 4 | `getting_moving`  | Engines Tier 1             | "Getting Moving"   | Navigation thrusters online. The Meridian can adjust its course to avoid further debris. ARIA detects the enemy ark more clearly — it's changing course to match. |
| 5 | `showing_teeth`   | Weapons Array Tier 1       | "Showing Teeth"    | Mothership point-defence turrets restored. Enemies near the mothership face automated fire. ARIA warns: the enemy ark has noticed the Meridian powering up. |
| 6 | `the_signal`      | All systems at Tier 3+     | "The Signal"       | ARIA intercepts a transmission from the enemy ark. It's a colony ship too — but its crew isn't in cryosleep. They want the Meridian's resources and route data. Triggers the boss fight unlock. |
| 7 | `last_stand`      | Defeat the enemy Ark boss  | "Last Stand"       | The enemy ark is destroyed or driven off. The Meridian's course is clear. |
| 8 | `new_dawn`        | Post-boss                  | "New Dawn"         | Epilogue. The Meridian arrives at the new world. The Pilot watches from the observation deck as the crew begins to wake. Credits. |

### Dialogue System (Planned)
- ARIA speaks through text boxes overlaid on the HUD or mothership UI.
- Dialogue is stored in a dictionary/resource and triggered by milestone signals.
- No player dialogue choices in MVP — ARIA monologues with contextual responses.

---

## Resource Table

| Resource | Weight | Value | Rarity   | Spawn % | Color  | Flavour |
|----------|--------|-------|----------|---------|--------|---------|
| Iron     | 1.0    | 1     | Common   | 40%     | Grey   | Structural base metal. Every repair needs it. |
| Copper   | 1.5    | 3     | Common   | 30%     | Orange | Wiring, circuitry, power conduits. |
| Crystal  | 2.0    | 5     | Uncommon | 15%     | Blue   | Quantum processors, shield emitters. |
| Titanium | 3.0    | 10    | Rare     | 10%     | Silver | Structural reinforcement, weapon housings. |
| Uranium  | 5.0    | 25    | V. Rare  | 5%      | Green  | Reactor fuel, high-energy systems. |

- **Spawn %** = probability that an asteroid's primary resource type is this type.
- Asteroids may contain 1–2 resource types; primary type dominates drops.
- Resource drops are scene instances that float and are collected on player overlap.

---

## Enemy Roster

### Scout Drone
| Stat     | Value |
|----------|-------|
| HP       | 15    |
| Speed    | 250   |
| Damage   | 5 (ram) |
| Behaviour| Spawns in pairs. Beeline toward player. Ram on contact. |
| Threat threshold | 0.1+ |

### Fighter
| Stat     | Value |
|----------|-------|
| HP       | 40    |
| Speed    | 150   |
| Damage   | 8 (projectile) |
| Behaviour| Strafes at medium range. Fires aimed projectiles. Retreats if HP < 30%. |
| Threat threshold | 0.3+ |

### Hauler
| Stat     | Value |
|----------|-------|
| HP       | 60    |
| Speed    | 80    |
| Damage   | 3 (ram, low priority) |
| Behaviour| Ignores player. Moves toward floating resource drops and steals them. Flees when cargo is full. |
| Threat threshold | 0.5+ |

### Elite (variant)
| Stat     | Value |
|----------|-------|
| HP       | ×2 of base type |
| Speed    | ×1.2 of base type |
| Damage   | ×1.5 of base type |
| Behaviour| Enhanced version of Scout/Fighter/Hauler with glow effect. Faster, tougher, hits harder. |
| Threat threshold | 0.7+ |

### Enemy Ark (Final Boss)
| Stat     | Value |
|----------|-------|
| HP       | 500 (multi-phase) |
| Speed    | 50 (slow advance) |
| Damage   | Varies by phase |
| Behaviour| Phase 1: Sends waves of drones. Phase 2: Direct fire (beam weapons). Phase 3: Ramming speed + all remaining drones. |
| Trigger  | Story milestone "The Signal" |

---

## Mothership Research Tree

Research is purchased with banked resources. Each tier costs more than the last. Research unlocks ship upgrade categories.

### Costs (Resource Units)

"Resource units" means a mix of resources. Costs below are in **total value** (resource value × quantity). The specific resource mix is flexible — player pays using whatever they have.

| System         | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Tier 5 |
|----------------|--------|--------|--------|--------|--------|
| Reactor        | 20     | 60     | 150    | —      | —      |
| Life Support   | 15     | 40     | 100    | 200    | 400    |
| Hull Integrity | 15     | 40     | 100    | 200    | 400    |
| Engines        | 15     | 40     | 100    | 200    | 400    |
| Weapons Array  | 30     | 80     | 200    | —      | —      |

**Prerequisites:**
- Reactor Tier 1 is required before ANY other research can begin.
- All other systems are independent after Reactor T1.
- Weapons Array requires Hull T1 + Life Support T1.

### What Each System Unlocks

**Reactor** (3 tiers):
- T1: Powers mothership. All other research becomes available. Unlocks Energy Cell ship upgrade.
- T2: Increased passive energy regen for ship.
- T3: Unlocks Energy Cell T3. Mothership lights brighten.

**Life Support** (5 tiers):
- T1: Crew stable. Unlocks Mining Beam + Combat Weapon ship upgrades.
- T2: Unlocks Mining Beam T2 + Combat Weapon T2.
- T3: Unlocks Resource Scanner ship upgrade.
- T4: Unlocks Mining Beam T3 + Combat Weapon T3.
- T5: Full life support. Crew can be woken (story flavour).

**Hull Integrity** (5 tiers):
- T1: Basic patches. Unlocks Hull HP + Cargo Capacity ship upgrades.
- T2: Unlocks Shield ship upgrade.
- T3: Unlocks Hull HP T3 + Cargo T3.
- T4: Unlocks Shield T2.
- T5: Full hull integrity. Mothership is structurally sound.

**Engines** (5 tiers):
- T1: Navigation restored. Unlocks Speed ship upgrade.
- T2–T5: Each tier unlocks matching Speed upgrade tier.

**Weapons Array** (3 tiers):
- T1: Mothership point-defence online. Enemies take passive damage near mothership.
- T2: Increased point-defence range and DPS.
- T3: Mothership safe zone significantly expanded.

---

## Ship Upgrade Tree

Ship upgrades cost banked resources and require the matching mothership research tier to be purchased first.

### Costs (Resource Value)

| Upgrade          | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Tier 5 |
|------------------|--------|--------|--------|--------|--------|
| Hull HP          | 10     | 25     | 50     | 100    | 200    |
| Shield           | 30     | 70     | 150    | —      | —      |
| Speed            | 10     | 25     | 50     | 100    | 200    |
| Cargo Capacity   | 10     | 25     | 50     | 100    | 200    |
| Mining Beam      | 15     | 40     | 100    | —      | —      |
| Combat Weapon    | 15     | 40     | 100    | —      | —      |
| Resource Scanner | 20     | 50     | —      | —      | —      |
| Energy Cell      | 10     | 25     | 60     | —      | —      |

### Effects Per Tier

| Upgrade          | Per Tier Effect                                                |
|------------------|----------------------------------------------------------------|
| Hull HP          | +20 max HP (base 100 → max 200 at T5)                        |
| Shield           | T1: +25 shield. T2: +25 + regen. T3: +25 + faster regen.    |
| Speed            | +15% max speed per tier (base 200 → ~370 px/s at T5)        |
| Cargo Capacity   | +20 weight capacity per tier (base 50 → 150 at T5)          |
| Mining Beam      | T1: +5 DPS, +50 range. T2: +5 DPS, −5 energy/s. T3: +5 DPS, +50 range. |
| Combat Weapon    | T1: Basic bolt. T2: Unlock missiles (homing). T3: Unlock shotgun (spread). |
| Resource Scanner | T1: Resource types visible on minimap (nearby). T2: Extended range. |
| Energy Cell      | +25 max energy, +5 regen/s per tier                          |

---

## Mothership Visual Progression

The mothership sprite itself stays the same, but visual indicators change:

| Milestone          | Visual Change |
|--------------------|---------------|
| Start (all T0)     | Dark hull, no lights, small flickering emergency beacon. |
| Reactor T1         | Interior lights turn on. Dim warm glow from windows. |
| Life Support T1    | Green status lights appear along hull. |
| Hull T2            | Damage decals (cracks, scorch marks) fade/disappear. |
| Engines T1         | Engine pods emit faint blue glow. |
| Weapons T1         | Turret barrels extend from hull. Point-defence fire visible when enemies approach. |
| All T3+            | Full glow. Ship looks pristine. Engine trails bright. All lights on. |
| Post-boss          | Ship in motion — parallax background scrolling to indicate travel to new world. |

---

## Asteroid Composition Rules

Asteroids have a **primary resource** (determines colour and dominant drop) and optionally a **secondary resource** (smaller chance to drop).

| Asteroid Size | HP Range | Primary Drop Qty | Secondary Drop Qty | Split Count |
|---------------|----------|-------------------|---------------------|-------------|
| Large         | 25–35    | 8–15              | 3–6                 | 2–3 Medium  |
| Medium        | 12–18    | 4–8               | 1–3                 | 2–3 Small   |
| Small         | 3–7      | 2–4               | 0–1                 | 0 (destroyed)|

- When a Large asteroid splits, children inherit the parent's resource composition but with slight variation.
- Uranium asteroids are rarer but don't split — they shatter directly into drops (high risk/reward since they appear at greater distances).

---

## Threat Escalation Details

```
threat_level = clamp(
    (distance_from_mothership / MAX_THREAT_DISTANCE) * 0.5
    + (cargo_weight / max_cargo_capacity) * 0.5,
    0.0, 1.0
)
```

| Threat Range | Enemy Types Available | Spawn Interval | Max Concurrent |
|-------------|----------------------|----------------|----------------|
| 0.0 – 0.1  | None (safe zone)     | —              | 0              |
| 0.1 – 0.3  | Scout                | 15s            | 3              |
| 0.3 – 0.5  | Scout, Fighter       | 10s            | 5              |
| 0.5 – 0.7  | Scout, Fighter, Hauler | 7s           | 8              |
| 0.7 – 1.0  | All + Elites         | 4s             | 12             |

- Mothership Weapons Array research reduces effective threat level within a radius of the mothership.

---

## Audio Direction (Planned)

| Context          | Audio Style                                              |
|------------------|----------------------------------------------------------|
| Mothership docked| Ambient hum, mechanical sounds, quiet. Warm and safe.    |
| Mining (low threat)| Soft ambient synth. Laser hum. Asteroid crack SFX.     |
| Combat (mid threat)| Pulsing beat, escalating layers as threat rises.       |
| High threat      | Intense synth, fast percussion, warning klaxons.         |
| Boss fight       | Full orchestral/synth boss track.                        |
| Epilogue         | Hopeful, swelling, resolving theme.                      |

---

## Balance Notes

All numeric values in this document and in `CLAUDE.MD` are **working defaults** subject to change during playtesting. When adjusting balance:

1. Update the value in the relevant file.
2. Note the change in `PROJECTSTATUS.MD` so other agents see it.
3. Prefer small incremental adjustments over large swings.
4. The core tension is: **greed vs. survival**. If the player never feels pressure to return, enemies are too weak or cargo limit is too high. If they can never fill their hold, resources are too scarce.
