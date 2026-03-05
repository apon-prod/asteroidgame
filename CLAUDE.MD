# CLAUDE.MD — AsteroidGame Technical Architecture

> **For AI agents:** Read this file AND `PROJECTSTATUS.MD` at the start of every session.
> Update this file when architecture, systems, or conventions change.
> For story, lore, and game design details see `GAME_DESIGN.MD`.
> Do NOT speculate — only record decisions that have been made.

---

## Project Overview

**Game:** 2D top-down space mining game with run-based progression and narrative arc.
**Working title:** AsteroidGame
**Engine:** Godot 4.5, Forward Plus renderer, GDScript only.
**Platform:** PC (Windows primary, Linux/Mac secondary).
**Repo:** https://github.com/apon-prod/asteroidgame

### Core Game Loop
1. Player starts docked at the Mothership (colony ark "The Meridian").
2. Player launches a run — flies out into an infinite asteroid field.
3. Player mines asteroids with a continuous laser beam, collecting resources.
4. Resources have weight — cargo slows the player as it fills.
5. Enemies escalate based on distance from mothership + cargo carried.
6. Player must return to the mothership to bank resources (death = lose all carried).
7. Banked resources buy **Mothership Research** (unlocks upgrade categories, drives story) and **Ship Upgrades** (stat improvements).
8. Repeat until the mothership is fully repaired → final boss → epilogue.

---

## Repository Structure

```
asteroid-game/                  ← Godot project root
├── CLAUDE.MD                   ← This file (technical architecture)
├── GAME_DESIGN.MD              ← Story, lore, design tables, balance values
├── PROJECTSTATUS.MD            ← Live task tracker / backlog
├── project.godot
│
├── scenes/                     ← .tscn scene files only
│   ├── main/                   ← Entry point scene + game loop
│   ├── player/                 ← Player ship scene
│   ├── enemies/                ← Enemy ship scenes (scout, fighter, …)
│   ├── asteroids/              ← Procedural asteroid scene + resource drop scene
│   ├── ui/
│   │   ├── hud/                ← In-game HUD (health, shield, energy, cargo, threat)
│   │   ├── menus/              ← Main menu, pause, game-over, death screens
│   │   └── shop/               ← Mothership upgrade UI + ship upgrade UI
│   ├── world/                  ← Mothership scene, dock zone, world container
│   └── effects/                ← Particles, explosions, mining beam VFX
│
├── scripts/                    ← .gd files (mirrors scenes/ + systems + utils)
│   ├── player/
│   ├── enemies/
│   ├── asteroids/
│   ├── ui/
│   ├── world/
│   ├── effects/
│   ├── systems/                ← Autoloads (see below)
│   └── utils/                  ← Shared constants, helper functions
│
├── resources/                  ← .tres data files (no logic — config/data only)
│   ├── upgrades/               ← UpgradeData .tres (mothership + ship)
│   ├── enemies/                ← EnemyData .tres (stats per enemy type)
│   └── items/                  ← ResourceType definitions
│
├── assets/                     ← Art and audio (never scripts)
│   ├── sprites/
│   │   ├── player/
│   │   ├── enemies/
│   │   ├── asteroids/
│   │   ├── ui/
│   │   └── environment/        ← Parallax layers, starfields, nebulae
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   ├── fonts/
│   └── shaders/
│
├── addons/                     ← Godot plugins (none yet)
└── .godot/                     ← Engine cache (git-ignored)
```

### Folder Rules
- **`scenes/`** — `.tscn` files only. Never put scripts here.
- **`scripts/`** — `.gd` files only. Mirrors `scenes/` structure where applicable.
- **`resources/`** — `.tres` data files. No logic — data/config only.
- **`assets/`** — Art and audio. Never scripts or scenes.

---

## Autoloads (Global Singletons)

Registered in `project.godot`. Accessible by name from any script.

| Autoload         | Script path                            | Purpose                                                    |
|------------------|----------------------------------------|------------------------------------------------------------|
| `EventBus`       | `scripts/systems/event_bus.gd`         | Decoupled signal hub for cross-scene communication         |
| `GameState`      | `scripts/systems/game_state.gd`        | Game phase FSM, settings, current run metadata             |
| `SaveManager`    | `scripts/systems/save_manager.gd`      | Persistent save/load to `user://savegame.json`             |
| `ResourceManager`| `scripts/systems/resource_manager.gd`  | Banked resources (persistent) + carried resources (per-run)|
| `UpgradeManager` | `scripts/systems/upgrade_manager.gd`   | Mothership research tree + ship upgrade tree, unlock logic |
| `RunManager`     | `scripts/systems/run_manager.gd`       | Run lifecycle: start, end, death, bank, threat calculation |

> **Note:** `WaveManager` is NOT an autoload — it is a scene-local node inside `World`, active only during runs.

> **Status:** Planned — not yet implemented.

---

## Game State Machine

```
MAIN_MENU → NEW_GAME → MOTHERSHIP_DOCKED ⇄ RUN_ACTIVE → (BANKED | PLAYER_DIED)
                                    ↓                         ↓
                              FINAL_BOSS               MOTHERSHIP_DOCKED
                                    ↓
                               EPILOGUE
```

| State              | Description                                                              |
|--------------------|--------------------------------------------------------------------------|
| `MAIN_MENU`        | Title screen. New game / load game / quit.                               |
| `MOTHERSHIP_DOCKED`| Player at mothership. Upgrade UI overlay active. Can launch a run.       |
| `RUN_ACTIVE`       | Player in the field. Mining, fighting, carrying cargo.                   |
| `PLAYER_DIED`      | Death screen. All carried resources lost. Returns to `MOTHERSHIP_DOCKED`.|
| `BANKED`           | Player returned safely. Carried → banked. Transitions to `MOTHERSHIP_DOCKED`.|
| `FINAL_BOSS`       | Story milestone triggers enemy Ark boss fight.                           |
| `EPILOGUE`         | Win state. Credits / ending.                                             |

Managed by `GameState` autoload. State transitions emit signals via `EventBus`.

---

## Scene Architecture

```
Main (Node2D)
├── ParallaxBackground
│   ├── StarLayer1 (far, slow)
│   ├── StarLayer2 (mid)
│   └── NebulaLayer (near, fast)
├── World (Node2D)
│   ├── Mothership (StaticBody2D — safe zone, dock trigger, visual repair stages)
│   ├── AsteroidField (Node2D — spawner manages procedural asteroid instances)
│   ├── ResourceDropContainer (Node2D — holds floating resource pickups)
│   └── EnemyContainer (Node2D)
│       └── WaveManager (Node — scene-local, spawns enemies based on threat level)
├── PlayerShip (CharacterBody2D)
│   ├── Sprite2D (ship visual)
│   ├── CollisionShape2D
│   ├── MiningBeam (Line2D + RayCast2D)
│   ├── WeaponMount (Node2D — projectile spawn point, rotates with aim)
│   ├── Camera2D (smooth follow)
│   ├── CargoComponent (Node — tracks carried resources, weight, speed penalty)
│   ├── HealthComponent (Node — HP + shield, regen)
│   └── EnergyComponent (Node — mining beam energy, drain + regen)
├── HUD (CanvasLayer)
│   ├── HealthBar
│   ├── ShieldBar
│   ├── EnergyBar
│   ├── CargoMeter (weight / capacity)
│   ├── ResourceCounters (carried this run)
│   ├── ThreatIndicator
│   └── MothershipDirectionIndicator (arrow pointing home)
└── MothershipUI (CanvasLayer — overlay when docked)
    ├── ResearchPanel (mothership research tree)
    ├── ShipUpgradePanel (ship stat upgrades)
    ├── StoryPanel (narrative / dialogue)
    └── LaunchButton
```

---

## Systems Design

### Movement System
- **WASD** = directional thrust (8-directional, screen-relative)
- **Mouse** = aim direction (ship rotates to face cursor)
- **Controller**: Left stick = move, right stick = aim (twin-stick)
- **Physics-based** with drag: apply thrust force from input, apply drag coefficient each frame, cap at `max_speed`
- **Spacey feel**: Low drag → ship drifts when thrust stops. Use `lerp()` / `move_toward()` for smooth deceleration. Never instant stop.
- **Cargo slowdown**: `effective_max_speed = base_max_speed * (1.0 - (cargo_weight / overload_threshold))`, clamped to a floor minimum
- **Visual juice**: sprite tilts slightly toward movement direction (tween), thruster particles scale with thrust magnitude

### Mining Beam System
- **Input**: Hold LMB (controller: right trigger) to fire toward cursor
- **Implementation**: `RayCast2D` for hit detection, `Line2D` for beam visual
- **DPS**: Continuous damage to asteroid under beam. Rate scales with Mining Beam upgrade tier.
- **Energy**: Drains energy bar while active. Recharges when not firing. `EnergyComponent` manages capacity + regen rate (both upgradeable via Energy Cell).
- **On asteroid death**: asteroid splits into smaller pieces and/or drops `ResourceDrop` scene instances

### Combat Weapon System
- **Input**: RMB (controller: right shoulder button) to fire
- **Base weapon**: Projectile (bullet/bolt) fired toward cursor
- **Upgrade path**: Higher tiers unlock new weapon types:
  - Tier 1: Basic bolt (single projectile)
  - Tier 2: Missiles (homing, slower)
  - Tier 3: Shotgun spread (multiple projectiles, short range)
- **Each weapon type**: Separate scene inheriting from a base `Projectile` class
- **No ammo**: Fire rate limited by cooldown per weapon type

### Procedural Asteroid System

**Phase 1 — Procedural polygon asteroids (MVP):**
- Generate irregular polygon using radial vertex variation
- Algorithm: N vertices at equal angular intervals, radius varied by ±random factor per vertex
- Size bands: Small (15–30px radius), Medium (40–70px), Large (90–140px)
- Color tinted by primary resource composition (see Resource Table in `GAME_DESIGN.MD`)
- HP proportional to size
- On destruction: split into 2–3 smaller asteroids + resource drops
- Collision: `CollisionPolygon2D` generated from same vertex array
- `Polygon2D` for rendering

**Phase 2 — Cell-grid destructible asteroids (future enhancement):**
- Each asteroid backed by a 2D grid of cells
- Mining beam destroys individual cells → visible craters
- Collision shape regenerated periodically
- Disconnected regions (flood-fill) split into separate physics bodies

### Enemy & Threat System
- **Threat level** = `(distance_from_mothership / MAX_DIST) * 0.5 + (cargo_weight / max_capacity) * 0.5`
- Threat drives: spawn rate, enemy type mix, enemy stat scaling
- `WaveManager` (scene-local in `EnemyContainer`): timer-based spawning, selects types by threat threshold
- Enemies spawn at viewport edge + buffer, approach player or patrol
- See `GAME_DESIGN.MD` for full enemy roster and stats

### Resource System
Two pools managed by `ResourceManager`:
- **`carried`** (Dictionary) — current run cargo. Lost entirely on death.
- **`banked`** (Dictionary) — persistent across runs. Saved to disk. Used for purchases.

| Resource | Weight | Value | Rarity   | Color  |
|----------|--------|-------|----------|--------|
| Iron     | 1.0    | 1     | Common   | Grey   |
| Copper   | 1.5    | 3     | Common   | Orange |
| Crystal  | 2.0    | 5     | Uncommon | Blue   |
| Titanium | 3.0    | 10    | Rare     | Silver |
| Uranium  | 5.0    | 25    | V. Rare  | Green  |

### Dual Upgrade Tree

**Mothership Research** (costs banked resources, drives narrative):

| System         | Max Tier | Unlocks                                                  |
|----------------|----------|----------------------------------------------------------|
| Reactor        | 3        | Powers all other systems (prerequisite). Unlocks Energy Cell upgrades. |
| Life Support   | 5        | Unlocks Mining Beam, Combat Weapon, Resource Scanner upgrades. |
| Hull Integrity | 5        | Unlocks Ship Hull HP, Shield, Cargo Capacity upgrades.   |
| Engines        | 5        | Unlocks Ship Speed/Acceleration upgrades.                |
| Weapons Array  | 3        | Passive: reduces enemy threat level near mothership.     |

**Ship Upgrades** (costs banked resources, requires matching mothership research tier):

| Upgrade          | Max Tier | Requires                   | Effect per tier                  |
|------------------|----------|----------------------------|----------------------------------|
| Hull HP          | 5        | Mothership Hull [tier]     | +20 HP                           |
| Shield           | 3        | Mothership Hull 2+         | +25 shield, shield regen         |
| Speed            | 5        | Mothership Engines [tier]  | +15% max speed                   |
| Cargo Capacity   | 5        | Mothership Hull [tier]     | +20 weight capacity              |
| Mining Beam      | 3        | Mothership Life Sup [tier] | +DPS, +range, −energy cost       |
| Combat Weapon    | 3        | Mothership Life Sup [tier] | Unlock new weapon types          |
| Resource Scanner | 2        | Mothership Life Sup 3+     | Show resource types on minimap   |
| Energy Cell      | 3        | Mothership Reactor [tier]  | +energy capacity, +regen rate    |

See `GAME_DESIGN.MD` for exact costs and full design rationale.

### Save System
`SaveManager` writes to `user://savegame.json`:
```json
{
  "banked_resources": { "iron": 0, "copper": 0, "crystal": 0, "titanium": 0, "uranium": 0 },
  "mothership_research": { "reactor": 0, "life_support": 0, "hull": 0, "engines": 0, "weapons": 0 },
  "ship_upgrades": { "hull_hp": 0, "shield": 0, "speed": 0, "cargo": 0, "mining_beam": 0, "combat_weapon": 0, "scanner": 0, "energy_cell": 0 },
  "story_progress": { "current_milestone": "none", "dialogue_seen": [] },
  "stats": { "total_runs": 0, "total_deaths": 0, "total_resources_banked": 0 }
}
```
- Saved after every successful bank and every upgrade purchase.
- Loaded on game start if save file exists.
- No autosave during runs (death = lose carried resources by design).

---

## Signal Contracts (EventBus)

All cross-scene signals go through the `EventBus` autoload. Direct signals within a single scene are fine.

```gdscript
# --- Run Lifecycle ---
signal run_started()
signal run_ended(banked: bool)        # true = returned safely, false = died
signal player_died()
signal player_docked()
signal player_launched()

# --- Resources ---
signal resource_collected(type: String, amount: int)
signal resource_banked(totals: Dictionary)
signal cargo_changed(current_weight: float, max_weight: float)

# --- Combat / Health ---
signal player_health_changed(current: int, max_val: int)
signal player_shield_changed(current: int, max_val: int)
signal player_energy_changed(current: float, max_val: float)
signal enemy_destroyed(enemy_type: String, position: Vector2)

# --- Upgrades ---
signal upgrade_purchased(tree: String, upgrade_id: String, new_tier: int)
signal research_completed(system: String, new_tier: int)

# --- Story ---
signal story_milestone_reached(milestone_id: String)
signal dialogue_triggered(dialogue_id: String)

# --- Threat ---
signal threat_level_changed(level: float)
```

---

## Coding Conventions

### GDScript Style
- `class_name` at the top of every script referenced elsewhere.
- `snake_case` for variables and functions: `player_speed`, `take_damage()`.
- `PascalCase` for class names and node names: `PlayerShip`, `AsteroidLarge`.
- `SCREAMING_SNAKE_CASE` for constants: `MAX_HEALTH`, `BASE_SPEED`.
- `@export` variables grouped at the top with `@export_group()`.
- Prefix private/internal functions with `_`: `_on_body_entered()`, `_calculate_thrust()`.
- Type hints on all function signatures and exported variables.

### Comments
- Every script starts with a `##` docblock explaining purpose.
- Public functions get a one-line `##` comment.
- Non-obvious logic gets inline comments.
- Agents: leave `# TODO(agent):` markers for unfinished work.

### File Naming
- Scenes: `snake_case.tscn` — e.g., `player_ship.tscn`, `asteroid.tscn`
- Scripts: `snake_case.gd` — match the scene they attach to where possible.
- Assets: `snake_case` — e.g., `player_ship_sprite.png`

### Node Naming
- PascalCase: `Sprite2D`, `CollisionShape2D`, `HealthComponent`.
- Group related child nodes under clearly named parents.

---

## Key Constants (Working Defaults)

Balance values — will change. See `GAME_DESIGN.MD` for full tables.

```gdscript
# Player movement
const BASE_MAX_SPEED: float = 200.0     # px/sec
const THRUST_FORCE: float = 400.0       # px/sec²
const DRAG_COEFFICIENT: float = 0.98    # multiplied per frame (1.0 = no drag)
const CARGO_OVERLOAD_THRESHOLD: float = 100.0  # weight at which speed = floor
const MIN_SPEED_FRACTION: float = 0.3   # never slower than 30% of base

# Player stats (base, before upgrades)
const BASE_MAX_HEALTH: int = 100
const BASE_MAX_SHIELD: int = 0          # starts with no shield
const BASE_MAX_ENERGY: float = 100.0
const ENERGY_REGEN_RATE: float = 15.0   # per second when not firing
const BASE_CARGO_CAPACITY: float = 50.0 # weight units

# Mining beam
const MINING_DPS: float = 10.0
const MINING_RANGE: float = 200.0       # pixels
const MINING_ENERGY_DRAIN: float = 20.0 # per second

# Asteroids (HP by size band)
const ASTEROID_HP_SMALL: int = 5
const ASTEROID_HP_MEDIUM: int = 15
const ASTEROID_HP_LARGE: int = 30

# Threat
const MAX_THREAT_DISTANCE: float = 3000.0  # px from mothership for max distance threat
```

---

## Multi-Agent Coordination

When multiple agents work in parallel, see `AGENT_ASSIGNMENTS.MD` for full details. Key rules:

### File Ownership (Phases 0–4)

| Path | Owner |
|------|-------|
| `scripts/systems/`, `scripts/utils/`, `scenes/main/`, `project.godot` | **ARIA** |
| `scripts/player/`, `scenes/player/` | **Pilot** |
| `scripts/world/`, `scripts/asteroids/`, `scenes/world/`, `scenes/asteroids/` | **Navigator** |

**Rule:** Never create, edit, or delete files outside your owned paths. If you need something from another agent's domain, code against the API contracts in `AGENT_ASSIGNMENTS.MD` and trust it will exist at integration time.

### Collision Layers

| Layer | Name | Used by |
|-------|------|---------|
| 1 | Player | PlayerShip body |
| 2 | Enemies | Enemy bodies |
| 3 | Asteroids | Asteroid bodies |
| 4 | Projectiles | Player + enemy projectiles |
| 5 | Pickups | Resource drops |
| 6 | Zones | Mothership dock/safe zone |

### Groups

| Group | Added by | Purpose |
|-------|----------|---------|
| `"player"` | Pilot | Find player: `get_tree().get_first_node_in_group("player")` |
| `"asteroids"` | Navigator | Find all asteroids |
| `"resource_drops"` | Navigator | Find all floating drops |
| `"mothership"` | Navigator | Find mothership node |
| `"enemies"` | (future) | Find all enemy nodes |

---

## Agent Instructions

1. **Always read `CLAUDE.MD`, `GAME_DESIGN.MD`, and `PROJECTSTATUS.MD` before starting work.**
2. **If running in parallel, also read `AGENT_ASSIGNMENTS.MD`** for file ownership and API contracts.
3. **Update `PROJECTSTATUS.MD`** when you begin a task, complete it, or discover blockers.
4. **Do not change architecture decisions** without noting the change in both `CLAUDE.MD` and `PROJECTSTATUS.MD`.
5. **Keep scripts self-contained and commented** — another agent will extend your work.
6. **Prefer signals over direct calls** for cross-node communication. Use `EventBus` for cross-scene.
7. **Test scenes work in isolation** before wiring them into the Main scene.
8. Leave `# TODO(agent):` markers for anything left intentionally incomplete.
9. **Use `/godot` skill** when working on `.tscn` or `.tres` files for format guidance.
10. **Type hints on all function signatures** — no untyped public APIs.
11. **Never touch files outside your owned paths** during parallel runs.

---

## Session Log

| Date       | Agent / Human | Summary |
|------------|---------------|---------|
| 2026-03-05 | Human + Claude | Project initialised. CLAUDE.MD + PROJECTSTATUS.MD created. Git repo set up. Pushed to github.com/apon-prod/asteroidgame. |
| 2026-03-05 | Human          | Decisions locked: WASD+mouse controls, twin-stick controller, placeholder art, infinite scrolling with parallax. |
| 2026-03-05 | Human + Claude | Full architecture designed. CLAUDE.MD rewritten. GAME_DESIGN.MD created. PROJECTSTATUS.MD updated with 12-phase backlog. Run-based loop, dual upgrade trees, procedural asteroids, narrative arc finalised. |
| 2026-03-05 | Human + Claude | Created AGENT_ASSIGNMENTS.MD. Split Phases 0–4 across 3 parallel agents (ARIA, Pilot, Navigator). Added multi-agent coordination section to CLAUDE.MD. |
