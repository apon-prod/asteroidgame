# CLAUDE.MD — AsteroidGame Project Context

> **For AI agents:** Read this file at the start of every session.
> Update it whenever the architecture, conventions, or game design changes.
> Do NOT speculate — only record decisions that have been made and implemented.

---

## Project Overview

**Game:** 2D top-down space mining game built in Godot 4 (GDScript).
**Working title:** AsteroidGame (rename TBD).
**Engine version:** Godot 4.5, Forward Plus renderer.
**Platform target:** PC (Windows primary, Linux/Mac secondary).

### Core Game Loop
1. Player pilots a spaceship through a procedurally scattered asteroid field.
2. Player mines asteroids to collect resources.
3. Resources are spent at a shop/station to buy upgrades.
4. Enemy ships spawn and escalate over time, attacking the player.
5. Player survives, mines, upgrades — repeat.

---

## Repository Structure

```
asteroid-game/                  ← Godot project root
├── CLAUDE.MD                   ← This file (agent context)
├── PROJECTSTATUS.MD            ← Live task tracker
├── project.godot
├── icon.svg
│
├── scenes/                     ← .tscn scene files only
│   ├── main/                   ← Entry point scene + game loop
│   ├── player/                 ← Player ship scene
│   ├── enemies/                ← Enemy ship scenes (scout, fighter, …)
│   ├── asteroids/              ← Asteroid scenes + resource drop scene
│   ├── ui/
│   │   ├── hud/                ← In-game HUD (health, shields, resources)
│   │   ├── menus/              ← Main menu, pause, game-over screens
│   │   └── shop/               ← Upgrade shop UI
│   ├── world/                  ← World container, space station
│   └── effects/                ← Particles, explosions, mining beam VFX
│
├── scripts/                    ← .gd GDScript files (mirrors scenes where possible)
│   ├── player/
│   ├── enemies/
│   ├── asteroids/
│   ├── ui/
│   ├── world/
│   ├── effects/
│   ├── systems/                ← Autoloads: EventBus, GameState, ResourceManager, WaveManager
│   └── utils/                  ← Shared constants, helper functions
│
├── resources/                  ← .tres data files (no logic — config/data only)
│   ├── upgrades/               ← UpgradeData resources (one .tres per upgrade tier)
│   ├── enemies/                ← EnemyData resources (stats per enemy type)
│   └── items/                  ← ResourceDrop definitions
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
│   └── shaders/                ← .gdshader files (screen effects, glow, etc.)
│
├── addons/                     ← Godot plugins (none yet)
└── .godot/                     ← Godot cache (git-ignored)
```

### Folder Rules
- **`scenes/`** — `.tscn` files only. Never put scripts here.
- **`scripts/`** — `.gd` files only. Mirrors `scenes/` structure where the script belongs to a scene.
- **`resources/`** — `.tres` data files. No logic — data/config only. Reference scripts via `ExtResource`.
- **`assets/`** — Art and audio. Never scripts or scene files.
- **`addons/`** — Godot plugins installed via AssetLib or manually.

---

## Architecture Decisions

### Autoloads (Singletons)
Global systems are registered as Autoloads in `project.godot` so any script can access them.

| Autoload name    | Script path                          | Purpose                              |
|------------------|--------------------------------------|--------------------------------------|
| `ResourceManager`| `scripts/systems/resource_manager.gd`| Tracks player's resource inventory   |
| `GameState`      | `scripts/systems/game_state.gd`      | Game phase, score, session data      |
| `WaveManager`    | `scripts/systems/wave_manager.gd`    | Spawns and escalates enemy waves     |
| `EventBus`       | `scripts/systems/event_bus.gd`       | Decoupled signal hub (pub/sub)       |

> **Status:** Planned — not yet implemented.

### Signal Architecture
Prefer signals over direct node references to keep scenes loosely coupled.
Use `EventBus` (a plain Node autoload) for cross-scene communication.
Direct node signals are fine within a single scene.

### World Structure
- **Infinite scrolling** world — no hard boundaries.
- Background uses **parallax layers** (multiple depths of star/nebula sprites) to convey depth.
- Asteroids and enemies spawn dynamically around the player as they travel.

### Scene Instancing
Enemies, asteroids, and projectiles are instanced from `.tscn` files at runtime.
Keep scene files lean — logic lives in attached scripts.

---

## Game Systems (Planned)

### Player Ship
- Node type: `CharacterBody2D`
- Movement: **WASD** for directional thrust, **mouse** for aiming. Controller: left stick for movement, right stick for aiming (twin-stick layout).
- Health: Hit points, shield layer (upgradeable).
- Mining: Laser/beam weapon that damages asteroids and collects drops.
- Combat: Separate weapon slot for enemy combat.

### Asteroids
- Three sizes: Large → Medium → Small (split on destruction).
- Each size yields different resource amounts.
- Resource types: Metal, Crystal, Fuel (names subject to change).
- Large asteroids have more HP and take more hits to break.

### Resources
| Resource | Source          | Use                              |
|----------|-----------------|----------------------------------|
| Metal    | All asteroids   | Hull upgrades, weapons           |
| Crystal  | Medium/Large    | Shield upgrades, special weapons |
| Fuel     | Small/Large     | Engine upgrades, dash ability    |

### Enemies
| Type       | Behaviour                                      | Difficulty |
|------------|------------------------------------------------|------------|
| Scout      | Fast, low HP, ram/ram-shoot hybrid             | Easy       |
| Fighter    | Balanced, shoots projectiles, strafes          | Medium     |
| Hauler     | Slow, high HP, tries to steal asteroid drops   | Medium     |
| Boss       | Periodic, multi-phase, unique attack patterns  | Hard       |

### Upgrades / Shop
- Shop appears at a Space Station node (present in the world or accessed via menu).
- Upgrades are tiered (1–3 per category).
- Categories: Engine, Hull, Shield, Mining Laser, Combat Weapon, Cargo Hold.

### Wave System
- Waves increase in enemy count and type mix over time/score.
- Downtime between waves gives player mining windows.

---

## Coding Conventions

### GDScript Style
- Use `class_name` at the top of every script that will be referenced elsewhere.
- Snake_case for variables and functions: `player_speed`, `take_damage()`.
- PascalCase for class names and node names: `PlayerShip`, `AsteroidLarge`.
- Constants in SCREAMING_SNAKE_CASE: `MAX_HEALTH`, `BASE_SPEED`.
- Exported variables (`@export`) go at the top of the script, grouped and commented.
- Prefix private/internal functions with `_`: `_on_body_entered()`, `_calculate_thrust()`.

### Comments
- Every script starts with a brief docblock explaining what it does.
- Public functions have a one-line comment describing their purpose.
- Non-obvious logic has inline comments.
- Agents: leave `# TODO(agentname):` comments for unfinished work.

### File Naming
- Scene files: `snake_case.tscn` — e.g., `player_ship.tscn`, `asteroid_large.tscn`.
- Script files: `snake_case.gd` — match the scene they attach to where possible.
- Asset files: `snake_case` with type suffix — e.g., `player_ship_sprite.png`.

### Node Naming (inside scenes)
- Nodes use PascalCase: `Sprite2D`, `CollisionShape2D`, `AnimationPlayer`.
- Group related nodes under clearly named parent nodes.

---

## Key Constants & Values (Working Defaults)

These are starting values — balance passes will update them.

```gdscript
# Player
BASE_SPEED       = 200.0   # pixels/sec max velocity
ROTATION_SPEED   = 3.0     # radians/sec
MAX_HEALTH       = 100
MAX_SHIELD       = 50

# Asteroids
LARGE_HP         = 30
MEDIUM_HP        = 15
SMALL_HP         = 5

# Resources per asteroid (base drops)
LARGE_METAL      = 15
MEDIUM_METAL     = 7
SMALL_METAL      = 3
```

---

## Agent Instructions

1. **Always read `CLAUDE.MD` and `PROJECTSTATUS.MD` before starting work.**
2. **Update `PROJECTSTATUS.MD`** when you begin a task, complete it, or discover blockers.
3. **Do not change architecture decisions** without noting the change in both files.
4. **Keep scripts self-contained and commented** — another agent will extend your work.
5. **Prefer signals over direct calls** for cross-node communication.
6. **Test scenes work in isolation** before wiring them into the main scene.
7. Leave `# TODO(agent):` markers for anything left intentionally incomplete.

---

## Session Log

| Date       | Agent / Human | Summary                                              |
|------------|---------------|------------------------------------------------------|
| 2026-03-05 | Human + Claude| Project initialised. CLAUDE.MD + PROJECTSTATUS.MD created. Git repo set up. Pushed to github.com/apon-prod/asteroidgame. |
| 2026-03-05 | Human        | Decisions locked: WASD+mouse controls, twin-stick controller, placeholder art, infinite scrolling with parallax. |
