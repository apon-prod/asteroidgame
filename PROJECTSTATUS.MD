# PROJECTSTATUS.MD — AsteroidGame Live Task Tracker

> **For AI agents:** Update this file whenever you start, finish, or block on a task.
> One agent works on one task at a time. Claim a task by moving it to "In Progress"
> and adding your identifier. Mark it "Done" when complete.
> If you discover sub-tasks mid-work, add them to the relevant section.

---

## Current Agent Activity

| Agent / Session | Task                        | Status      | Notes                        |
|-----------------|-----------------------------|-------------|------------------------------|
| *(none)*        | *(none active)*             | Idle        |                              |

---

## Backlog — Prioritised

Tasks are roughly in implementation order. Re-order as dependencies become clear.

### Phase 0 — Foundation
- [ ] Set up Godot project folder structure (`scenes/`, `scripts/`, `assets/` subdirs)
- [ ] Create `EventBus` autoload (`scripts/systems/event_bus.gd`)
- [ ] Create `GameState` autoload (`scripts/systems/game_state.gd`)
- [ ] Create `ResourceManager` autoload (`scripts/systems/resource_manager.gd`)
- [ ] Register all autoloads in `project.godot`
- [ ] Create a placeholder `Main` scene (`scenes/main/main.tscn`) as the entry point
- [ ] Set `Main` as the project's main scene in `project.godot`

### Phase 1 — Player Ship
- [ ] Design player ship sprite (placeholder: coloured polygon or imported asset)
- [ ] Create `PlayerShip` scene (`scenes/player/player_ship.tscn`)
- [ ] Write `player_ship.gd` — movement (thrust + rotation), health, shield
- [ ] Wire player input map in `project.godot` (move forward, rotate left/right, brake, fire)
- [ ] Camera: attach `Camera2D` to player with smooth follow
- [ ] Add player to `Main` scene and verify movement

### Phase 2 — Asteroids
- [ ] Create sprites/placeholders for Large, Medium, Small asteroids
- [ ] Create `AsteroidLarge` scene + script (HP, split on death, resource drop)
- [ ] Create `AsteroidMedium` scene + script
- [ ] Create `AsteroidSmall` scene + script
- [ ] Create `ResourceDrop` scene (collectible that floats and is picked up on overlap)
- [ ] Create `AsteroidField` spawner — randomly places asteroids in the world
- [ ] Test full break chain: Large → 2× Medium → 2× Small → drops

### Phase 3 — Mining
- [ ] Add mining laser to `PlayerShip` (beam/raycast that damages asteroids)
- [ ] Wire mining input action
- [ ] On resource drop overlap, call `ResourceManager.add(type, amount)`
- [ ] Display resource counts in HUD

### Phase 4 — HUD & UI
- [ ] Create `HUD` scene (`scenes/ui/hud.tscn`) — health bar, shield bar, resource counters
- [ ] Add HUD to `Main` as a CanvasLayer
- [ ] Connect HUD to `ResourceManager` and player health signals via `EventBus`

### Phase 5 — Enemies
- [ ] Create `EnemyScout` scene + script (patrol → chase → attack AI)
- [ ] Create `EnemyFighter` scene + script (strafe + shoot AI)
- [ ] Create projectile scene (`scenes/enemies/projectile.tscn`)
- [ ] Implement player-enemy collision damage
- [ ] Create `WaveManager` autoload — timed wave spawning

### Phase 6 — Shop & Upgrades
- [ ] Design upgrade data structure (resource + dict or `Resource` class)
- [ ] Create `SpaceStation` scene in the world (safe zone, triggers shop UI)
- [ ] Create `Shop` UI scene with upgrade buttons
- [ ] Connect shop purchases to `ResourceManager` and player stats

### Phase 7 — Polish & Game Loop
- [ ] Game over screen (player death)
- [ ] Win condition or endless scoring
- [ ] Main menu scene
- [ ] Sound effects (mining, explosion, shooting)
- [ ] Background parallax starfield
- [ ] Screenshake on damage
- [ ] Balance pass on all numbers

### Phase 8 — Build & Release
- [ ] Configure Godot export presets (Windows, Linux)
- [ ] Export and playtest build
- [ ] Update README with play instructions

---

## In Progress

*(Nothing in progress — project just initialised.)*

---

## Completed

| Date       | Task                                              | Agent / Human |
|------------|---------------------------------------------------|---------------|
| 2026-03-05 | Initialise Godot 4.5 project                      | Human         |
| 2026-03-05 | Create CLAUDE.MD                                  | Claude        |
| 2026-03-05 | Create PROJECTSTATUS.MD                           | Claude        |
| 2026-03-05 | Initialise git repository + first commit          | Claude        |

---

## Blockers / Open Questions

| # | Issue                                        | Raised by | Status  |
|---|----------------------------------------------|-----------|---------|
| 1 | Player movement style: tank controls vs mouse-aim? | Human | **Resolved** — WASD moves, mouse aims. Controller: twin-stick (left = move, right = aim). |
| 2 | Art direction: programmer art placeholders or asset pack? | Human | **Resolved** — Placeholder art for now; real assets added as available. |
| 3 | World structure: infinite scrolling, large fixed map, or room-based? | Human | **Resolved** — Infinite scrolling with parallax background layers. |

---

## Architecture Notes for Agents

- All cross-scene communication goes through `EventBus` signals — do not hold direct references to nodes in other scenes.
- `ResourceManager`, `GameState`, and `WaveManager` are autoloads — access them by name anywhere.
- Scenes must work standalone (i.e., runnable in the editor) before being added to `Main`.
- Read `CLAUDE.MD` for full conventions, node naming rules, and system descriptions.
