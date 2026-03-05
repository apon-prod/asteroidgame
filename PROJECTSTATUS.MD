# PROJECTSTATUS.MD — AsteroidGame Live Task Tracker

> **For AI agents:** Update this file whenever you start, finish, or block on a task.
> One agent works on one task at a time. Claim a task by moving it to "In Progress"
> and adding your identifier. Mark it "Done" when complete.
> If you discover sub-tasks mid-work, add them to the relevant section.
> Read `CLAUDE.MD` for architecture and `GAME_DESIGN.MD` for design/story.

---

## Current Agent Activity

| Agent / Session | Task                        | Status      | Notes                        |
|-----------------|-----------------------------|-------------|------------------------------|
| *(none)*        | *(none active)*             | Idle        |                              |

---

## Backlog — Prioritised

Tasks are in implementation order. Dependencies are noted. Each phase should be fully testable before moving to the next.

### Phase 0 — Foundation
- [ ] Create `EventBus` autoload (`scripts/systems/event_bus.gd`) — all signals from CLAUDE.MD signal contracts
- [ ] Create `GameState` autoload (`scripts/systems/game_state.gd`) — FSM with states: MAIN_MENU, MOTHERSHIP_DOCKED, RUN_ACTIVE, PLAYER_DIED, BANKED, FINAL_BOSS, EPILOGUE
- [ ] Create `SaveManager` autoload (`scripts/systems/save_manager.gd`) — save/load to `user://savegame.json`
- [ ] Create `ResourceManager` autoload (`scripts/systems/resource_manager.gd`) — carried + banked pools, resource type definitions
- [ ] Create `UpgradeManager` autoload (`scripts/systems/upgrade_manager.gd`) — dual tree definitions, unlock checks, purchase logic
- [ ] Create `RunManager` autoload (`scripts/systems/run_manager.gd`) — run start/end/death/bank lifecycle, threat calculation
- [ ] Register all 6 autoloads in `project.godot`
- [ ] Create placeholder `Main` scene (`scenes/main/main.tscn`) with basic node structure from CLAUDE.MD scene architecture
- [ ] Set `Main` as the project's main scene in `project.godot`
- [ ] Wire input map in `project.godot`: move_up, move_down, move_left, move_right, fire_mining, fire_weapon, interact/dock

### Phase 1 — Player Ship
- [ ] Create player ship placeholder sprite (coloured polygon)
- [ ] Create `PlayerShip` scene (`scenes/player/player_ship.tscn`) as CharacterBody2D
- [ ] Write `player_ship.gd` — WASD thrust + mouse aim, physics-based with drag, drift
- [ ] Create `HealthComponent` (`scripts/player/health_component.gd`) — HP + shield, damage, death signal
- [ ] Create `EnergyComponent` (`scripts/player/energy_component.gd`) — energy pool, drain, regen
- [ ] Create `CargoComponent` (`scripts/player/cargo_component.gd`) — carried resources, weight, speed penalty
- [ ] Attach `Camera2D` with smooth follow (position smoothing enabled)
- [ ] Add thruster particles (CPUParticles2D) that scale with thrust
- [ ] Add PlayerShip to Main scene and verify: movement feels spacey, drifts, rotates to mouse
- [ ] Verify cargo slowdown: add debug weight and confirm speed reduction

### Phase 2 — Mothership
- [ ] Create `Mothership` scene (`scenes/world/mothership.tscn`) — StaticBody2D with placeholder sprite
- [ ] Add dock zone (Area2D trigger) — detects when PlayerShip enters
- [ ] On dock: emit `EventBus.player_docked`, transition GameState to MOTHERSHIP_DOCKED
- [ ] On launch: emit `EventBus.player_launched`, transition to RUN_ACTIVE
- [ ] Implement safe zone — enemies cannot spawn within mothership radius
- [ ] Add visual indicators: emergency beacon at start, lights turn on per research tier (see GAME_DESIGN.MD)
- [ ] Place Mothership at world origin (0,0) in Main scene

### Phase 3 — Procedural Asteroids
- [ ] Write `asteroid_generator.gd` (`scripts/asteroids/`) — procedural polygon generation
  - Radial vertex variation algorithm
  - Size bands: Small/Medium/Large with continuous variation
  - Resource composition determines colour tint
- [ ] Create `Asteroid` scene (`scenes/asteroids/asteroid.tscn`) — RigidBody2D with Polygon2D + CollisionPolygon2D
- [ ] Asteroid script: HP, take_damage(), split-on-death (Large→Medium→Small→drops)
- [ ] Uranium asteroids: shatter directly into drops (no split) — rare, high-value
- [ ] Create `AsteroidField` spawner (`scripts/asteroids/asteroid_field.gd`) — spawns asteroids around player as they travel, despawns distant ones
- [ ] Test: asteroids look varied, break chain works, resource type/colour visible

### Phase 4 — Mining Beam + Resource Drops + Cargo
- [ ] Create `MiningBeam` node (Line2D + RayCast2D) attached to PlayerShip
- [ ] Wire LMB input to activate beam toward mouse cursor
- [ ] Beam deals continuous DPS to asteroid under raycast hit
- [ ] Beam drains EnergyComponent while active; stops when energy = 0
- [ ] Energy recharges when beam is not firing
- [ ] Create `ResourceDrop` scene (`scenes/asteroids/resource_drop.tscn`) — Area2D with sprite, floats/drifts
- [ ] ResourceDrop: on player overlap, add to CargoComponent and emit `EventBus.resource_collected`
- [ ] CargoComponent: track weight, emit `EventBus.cargo_changed`, apply speed penalty to PlayerShip
- [ ] Test: mine an asteroid → drops appear → collect drops → cargo weight increases → speed decreases

### Phase 5 — HUD
- [ ] Create `HUD` scene (`scenes/ui/hud/hud.tscn`) as CanvasLayer
- [ ] Health bar (connects to `EventBus.player_health_changed`)
- [ ] Shield bar (connects to `EventBus.player_shield_changed`)
- [ ] Energy bar (connects to `EventBus.player_energy_changed`)
- [ ] Cargo meter: weight / capacity bar (connects to `EventBus.cargo_changed`)
- [ ] Resource counters: show carried quantities per type
- [ ] Threat indicator: colour/icon showing current threat level
- [ ] Mothership direction indicator: arrow pointing toward (0,0) when off-screen
- [ ] Add HUD to Main scene
- [ ] Test: all bars update in real-time during gameplay

### Phase 6 — Enemies + Threat System
- [ ] Create base `Enemy` scene/script (`scripts/enemies/enemy_base.gd`) — HP, speed, damage, death
- [ ] Create `EnemyScout` scene — fast, rams player (see GAME_DESIGN.MD stats)
- [ ] Create `EnemyFighter` scene — strafes, shoots projectiles
- [ ] Create enemy `Projectile` scene (`scenes/enemies/projectile.tscn`)
- [ ] Create `EnemyHauler` scene — targets resource drops, steals them
- [ ] Create `WaveManager` node (`scripts/world/wave_manager.gd`) — scene-local in EnemyContainer
- [ ] Implement threat level calculation: `f(distance, cargo_weight)`
- [ ] WaveManager spawns enemies based on threat thresholds (see GAME_DESIGN.MD table)
- [ ] Enemies spawn at viewport edge + buffer, despawn when far from player
- [ ] Player-enemy collision damage
- [ ] Test: fly far from mothership with cargo → enemies escalate correctly

### Phase 7 — Mothership UI + Upgrades + Save/Load
- [ ] Create `MothershipUI` scene (`scenes/ui/shop/mothership_ui.tscn`) as CanvasLayer overlay
- [ ] Research panel: shows mothership systems, current tiers, costs, purchase buttons
- [ ] Ship upgrade panel: shows available upgrades, locked/unlocked state, costs, purchase buttons
- [ ] Grey out locked upgrades (missing research prerequisite)
- [ ] On purchase: deduct from banked resources via ResourceManager, update UpgradeManager
- [ ] Apply ship upgrade effects to PlayerShip components (HP, speed, cargo, energy, etc.)
- [ ] Implement SaveManager: save after bank + after purchase, load on game start
- [ ] Banking: on dock, transfer CargoComponent contents to ResourceManager.banked, save
- [ ] Death: clear CargoComponent, transition to MOTHERSHIP_DOCKED (no bank)
- [ ] Story panel: show ARIA dialogue when milestones are reached
- [ ] Launch button: start new run
- [ ] Test: full loop — launch → mine → return → bank → buy upgrade → stats change → save → reload → state persists

### Phase 8 — Combat Weapons
- [ ] Create base `Projectile` class (`scripts/player/projectile_base.gd`)
- [ ] Create `BasicBolt` scene — single projectile toward cursor
- [ ] Wire RMB input to fire combat weapon
- [ ] Create `Missile` scene (T2 unlock) — homing, slower, area damage
- [ ] Create `ShotgunBlast` scene (T3 unlock) — spread of projectiles, short range
- [ ] WeaponMount on PlayerShip selects active weapon based on Combat Weapon upgrade tier
- [ ] Player projectiles damage enemies
- [ ] Test: all weapon types work, upgrades switch between them

### Phase 9 — Story System
- [ ] Create dialogue data structure (Dictionary or Resource)
- [ ] ARIA dialogue for each milestone (see GAME_DESIGN.MD narrative milestones)
- [ ] Trigger dialogue via `EventBus.story_milestone_reached` when research thresholds met
- [ ] Display dialogue in StoryPanel (text box overlay)
- [ ] Track seen dialogue in save data to avoid replaying
- [ ] Test: reach research milestones → correct dialogue appears → only once

### Phase 10 — Visual Polish + Audio
- [ ] Parallax background: 3 layers (far stars, mid stars, near nebula) that scroll with camera
- [ ] Screen shake on player damage (Camera2D offset + tween)
- [ ] Explosion particles on asteroid/enemy death
- [ ] Mining beam visual: glow, particles at hit point
- [ ] Mothership visual progression (lights, damage decals per research tier)
- [ ] Placeholder SFX: mining hum, explosion, projectile fire, pickup collect, dock
- [ ] Ambient music: calm near mothership, escalating with threat
- [ ] Ship sprite tilt tween during movement

### Phase 11 — Final Boss
- [ ] Create `EnemyArk` boss scene — large ship with multi-phase AI
- [ ] Phase 1: spawns scout/fighter waves
- [ ] Phase 2: direct beam weapon attacks
- [ ] Phase 3: ramming approach + all remaining drones
- [ ] Trigger when "The Signal" milestone reached and player launches a run
- [ ] Boss HP bar on HUD
- [ ] On defeat: trigger "Last Stand" → "New Dawn" epilogue
- [ ] Epilogue scene / credits screen

### Phase 12 — Build, Balance & Release
- [ ] Balance pass: resource scarcity, enemy difficulty, upgrade costs, cargo limits
- [ ] Main menu scene (New Game, Continue, Quit)
- [ ] Pause menu (Resume, Settings, Quit to Menu)
- [ ] Game over / death screen with stats
- [ ] Configure Godot export presets (Windows, Linux)
- [ ] Export and playtest builds
- [ ] Update README with play instructions and screenshots

---

## In Progress

*(Nothing in progress — architecture planning complete, implementation not yet started.)*

---

## Completed

| Date       | Task                                              | Agent / Human |
|------------|---------------------------------------------------|---------------|
| 2026-03-05 | Initialise Godot 4.5 project                      | Human         |
| 2026-03-05 | Create CLAUDE.MD (v1)                              | Claude        |
| 2026-03-05 | Create PROJECTSTATUS.MD (v1)                       | Claude        |
| 2026-03-05 | Initialise git repository + first commit           | Claude        |
| 2026-03-05 | Scaffold project folder structure                  | Claude        |
| 2026-03-05 | Install `/godot` skill                             | Claude        |
| 2026-03-05 | Full architecture design + CLAUDE.MD rewrite       | Human + Claude|
| 2026-03-05 | Create GAME_DESIGN.MD (story, lore, design tables) | Claude        |
| 2026-03-05 | Update PROJECTSTATUS.MD with 12-phase backlog      | Claude        |

---

## Blockers / Open Questions

| # | Issue                                        | Raised by | Status  |
|---|----------------------------------------------|-----------|---------|
| 1 | Player movement style | Human | **Resolved** — WASD thrust + mouse aim. Twin-stick controller. |
| 2 | Art direction | Human | **Resolved** — Placeholder art for now. |
| 3 | World structure | Human | **Resolved** — Infinite scrolling with parallax. |
| 4 | Death penalty | Human | **Resolved** — Lose all un-banked resources. |
| 5 | Mothership representation | Human | **Resolved** — Physical in-world + menu overlay when docked. |
| 6 | Enemy escalation driver | Human | **Resolved** — Distance from mothership + cargo weight. |
| 7 | Mining weapon type | Human | **Resolved** — Continuous laser beam with energy drain. |
| 8 | Combat weapon type | Human | **Resolved** — Multiple types (bolt → missile → shotgun), unlocked via upgrades. |
| 9 | Win condition | Human | **Resolved** — Defeat enemy Ark boss → epilogue. Story-driven, not endless. |
| 10| Save system | Human | **Resolved** — Persistent save. Upgrades + banked resources saved across sessions. |
| 11| Procedural asteroids | Human | **Resolved** — Phase 1: procedural polygons. Phase 2 (future): cell-grid destructible. |
| 12| Mothership visual progression | Human | **Resolved** — Lights and glow improve per research tier. Same base sprite. |

---

## Architecture Notes for Agents

- **Autoloads** (6): `EventBus`, `GameState`, `SaveManager`, `ResourceManager`, `UpgradeManager`, `RunManager`. Access by name anywhere.
- **WaveManager** is scene-local (inside `World/EnemyContainer`), NOT an autoload.
- All cross-scene communication goes through `EventBus` signals. See `CLAUDE.MD` for full signal contracts.
- Scenes must work standalone (runnable in the editor) before wiring into `Main`.
- Use `/godot` skill when creating `.tscn` or `.tres` files.
- Read `CLAUDE.MD` for conventions, scene tree, and system details.
- Read `GAME_DESIGN.MD` for stats, costs, story, and design rationale.
