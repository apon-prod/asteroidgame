# AGENT_ASSIGNMENTS.MD — Parallel Agent Coordination

> **Three agents run simultaneously.** Each owns exclusive file paths.
> No agent may create, edit, or delete files outside its owned paths.
> After all agents finish, a brief manual integration step wires everything together.

---

## File Ownership Table

| Path | Owner | Other agents: DO NOT TOUCH |
|------|-------|---------------------------|
| `scripts/systems/` | **ARIA** | Pilot, Navigator |
| `scripts/utils/` | **ARIA** | Pilot, Navigator |
| `scenes/main/` | **ARIA** | Pilot, Navigator |
| `project.godot` | **ARIA** | Pilot, Navigator |
| `scripts/player/` | **Pilot** | ARIA, Navigator |
| `scenes/player/` | **Pilot** | ARIA, Navigator |
| `scripts/world/` | **Navigator** | ARIA, Pilot |
| `scripts/asteroids/` | **Navigator** | ARIA, Pilot |
| `scenes/world/` | **Navigator** | ARIA, Pilot |
| `scenes/asteroids/` | **Navigator** | ARIA, Pilot |

**Shared read-only:** `CLAUDE.MD`, `GAME_DESIGN.MD`, `PROJECTSTATUS.MD` — read but do not edit during this run.

---

## Autoload Reference Strategy

Pilot and Navigator will write scripts that reference autoloads (`EventBus`, `ResourceManager`, etc.) by name. These references won't resolve in Godot until ARIA's autoloads are registered in `project.godot`. **This is expected.** Write the code as if the autoloads exist — it will all work once the three agents' output is combined.

---

## API Contracts

These are the public methods and signals each agent's code must expose so the other agents' code can call them. Follow these contracts exactly.

### EventBus (ARIA creates, Pilot & Navigator emit/connect)

```gdscript
# Pilot emits these:
EventBus.player_health_changed.emit(current, max_val)
EventBus.player_shield_changed.emit(current, max_val)
EventBus.player_energy_changed.emit(current, max_val)
EventBus.cargo_changed.emit(current_weight, max_weight)
EventBus.player_died.emit()

# Navigator emits these:
EventBus.resource_collected.emit(type, amount)   # type is String like "iron"
EventBus.player_docked.emit()
EventBus.player_launched.emit()
EventBus.enemy_destroyed.emit(enemy_type, position)

# ARIA's RunManager emits these:
EventBus.run_started.emit()
EventBus.run_ended.emit(banked)  # bool
EventBus.threat_level_changed.emit(level)  # float 0.0–1.0
```

### ResourceManager (ARIA creates, Navigator calls)

```gdscript
# Navigator's resource_drop.gd calls these on pickup:
ResourceManager.add_carried("iron", 3)
ResourceManager.get_carried_weight() -> float
ResourceManager.get_carried() -> Dictionary  # {"iron": 5, "copper": 2, ...}
```

### PlayerShip (Pilot creates, Navigator references for position)

```gdscript
# Navigator's asteroid_field.gd needs the player's position to spawn asteroids nearby.
# Access via: get_tree().get_first_node_in_group("player").global_position
# Pilot must add PlayerShip to group "player" in _ready().

# Navigator's mothership.gd dock zone detects the player via collision layers.
# Pilot must set PlayerShip to collision layer 1 (player layer).
```

### Collision Layer Convention

| Layer | Name | Used by |
|-------|------|---------|
| 1 | Player | PlayerShip body |
| 2 | Enemies | Enemy bodies |
| 3 | Asteroids | Asteroid bodies |
| 4 | Projectiles | Player + enemy projectiles |
| 5 | Pickups | Resource drops |
| 6 | Zones | Mothership dock zone, safe zone |

### Groups Convention

| Group name | Added by | Purpose |
|------------|----------|---------|
| `"player"` | Pilot | Find player node from anywhere: `get_tree().get_first_node_in_group("player")` |
| `"asteroids"` | Navigator | Find all asteroids |
| `"enemies"` | (future) | Find all enemies |
| `"resource_drops"` | Navigator | Find all floating drops |
| `"mothership"` | Navigator | Find mothership node |

---

## Post-Agent Integration Checklist

After all 3 agents finish, do these steps manually (or with a 4th agent):

- [ ] Open Godot editor — let it import all new files
- [ ] Open `scenes/main/main.tscn`
- [ ] Instance `scenes/player/player_ship.tscn` as a child of the Main root node
- [ ] Instance `scenes/world/mothership.tscn` as a child of the `World` node
- [ ] Verify AsteroidField spawner is a child of `World` (ARIA's main scene should have the placeholder node)
- [ ] Press F5 — the game should run: ship moves, asteroids spawn around player, mining beam fires
- [ ] Commit the integrated scene

---

## Agent Prompts

Copy-paste each prompt below into a separate Claude Code session.

---

### AGENT 1: ARIA — Foundation & Systems

```
You are ARIA, the foundation agent for AsteroidGame — a 2D space mining game built in Godot 4.5 with GDScript.

## Your mission
Build ALL global systems (autoloads) and the Main scene skeleton. You are the backbone — the other two agents (Pilot and Navigator) depend on your autoloads being correct.

## First steps
Read these files before writing ANY code:
1. CLAUDE.MD — full architecture, autoload table, game state machine, signal contracts, constants
2. GAME_DESIGN.MD — resource types, upgrade trees with costs/prerequisites, save format
3. PROJECTSTATUS.MD — your tasks are tagged [ARIA] in Phases 0
4. AGENT_ASSIGNMENTS.MD — file ownership rules and API contracts you must implement

## Files you OWN (create these)
- scripts/systems/event_bus.gd
- scripts/systems/game_state.gd
- scripts/systems/save_manager.gd
- scripts/systems/resource_manager.gd
- scripts/systems/upgrade_manager.gd
- scripts/systems/run_manager.gd
- scripts/utils/constants.gd
- scenes/main/main.tscn

## Files you MODIFY
- project.godot — register all 6 autoloads, set main scene, configure input map

## DO NOT TOUCH
- scripts/player/, scenes/player/ (owned by Pilot)
- scripts/world/, scripts/asteroids/, scenes/world/, scenes/asteroids/ (owned by Navigator)

## What to build

### EventBus (scripts/systems/event_bus.gd)
- Extends Node
- Declare ALL signals from CLAUDE.MD "Signal Contracts" section — exact signatures matter
- No logic, just signal declarations

### GameState (scripts/systems/game_state.gd)
- Extends Node
- Enum for states: MAIN_MENU, MOTHERSHIP_DOCKED, RUN_ACTIVE, PLAYER_DIED, BANKED, FINAL_BOSS, EPILOGUE
- current_state variable, transition methods that validate legal transitions
- Emit state changes via EventBus

### SaveManager (scripts/systems/save_manager.gd)
- Extends Node
- save_game() — writes to user://savegame.json using the exact JSON structure from CLAUDE.MD
- load_game() -> bool — reads file, returns false if no save exists
- has_save() -> bool
- delete_save()
- Reads from / writes to ResourceManager and UpgradeManager

### ResourceManager (scripts/systems/resource_manager.gd)
- Extends Node
- Resource type data: enum or string constants for IRON, COPPER, CRYSTAL, TITANIUM, URANIUM
- Resource metadata dict with weight, value, rarity, color for each type (from GAME_DESIGN.MD)
- Two pools: carried (Dictionary) and banked (Dictionary)
- add_carried(type: String, amount: int)
- remove_carried(type: String, amount: int)
- get_carried() -> Dictionary
- get_carried_weight() -> float (sum of amount * weight per type)
- bank_carried() — transfers all carried to banked, emits resource_banked
- clear_carried() — empties carried (on death)
- get_banked() -> Dictionary
- spend_banked(cost: Dictionary) -> bool — deducts if affordable, returns success
- can_afford(cost: Dictionary) -> bool
- get_total_banked_value() -> int

### UpgradeManager (scripts/systems/upgrade_manager.gd)
- Extends Node
- Mothership research tree: dict of system_id -> {current_tier, max_tier, costs[], prerequisites}
- Ship upgrade tree: dict of upgrade_id -> {current_tier, max_tier, costs[], required_research, effects}
- All data from GAME_DESIGN.MD upgrade tables
- can_research(system_id) -> bool (checks prerequisites + can_afford)
- do_research(system_id) -> bool (purchases, increments tier, emits research_completed)
- can_upgrade_ship(upgrade_id) -> bool (checks research tier + can_afford)
- do_upgrade_ship(upgrade_id) -> bool (purchases, increments tier, emits upgrade_purchased)
- get_ship_stat_bonus(stat: String) -> Variant (returns current bonus for a stat based on tiers)
- get_research_tier(system_id) -> int
- get_upgrade_tier(upgrade_id) -> int

### RunManager (scripts/systems/run_manager.gd)
- Extends Node
- is_run_active: bool
- run_time: float (seconds since run started)
- start_run() — resets carried resources, sets GameState to RUN_ACTIVE
- end_run_banked() — banks carried, sets GameState to BANKED then MOTHERSHIP_DOCKED
- end_run_death() — clears carried, sets GameState to PLAYER_DIED then MOTHERSHIP_DOCKED
- calculate_threat_level(player_position: Vector2) -> float — formula from CLAUDE.MD
- _process: if run active, update run_time and recalculate threat periodically

### Constants (scripts/utils/constants.gd)
- class_name Constants
- All constants from CLAUDE.MD "Key Constants" section
- Collision layer constants (LAYER_PLAYER = 1, LAYER_ENEMIES = 2, etc.)
- Group name constants (GROUP_PLAYER = "player", etc.)

### Main Scene (scenes/main/main.tscn)
- Root: Node2D named "Main"
- Children matching CLAUDE.MD scene architecture:
  - ParallaxBackground (can be empty for now)
  - World (Node2D) — with child nodes: AsteroidField (Node2D), ResourceDropContainer (Node2D), EnemyContainer (Node2D)
  - HUD (CanvasLayer) — empty placeholder
  - MothershipUI (CanvasLayer) — empty placeholder, visible = false
- Do NOT instance PlayerShip or Mothership — those are created by other agents and wired in during integration

### Input Map (project.godot)
Configure these input actions:
- move_up: W + Up arrow + controller left stick up
- move_down: S + Down arrow + controller left stick down
- move_left: A + Left arrow + controller left stick left
- move_right: D + Right arrow + controller left stick right
- fire_mining: Mouse LMB + controller right trigger
- fire_weapon: Mouse RMB + controller right shoulder
- interact: E + controller A/X button

## Coding conventions
- class_name on every script
- Type hints on ALL function signatures
- @export variables grouped with @export_group()
- ## docblock at top of every script
- snake_case functions/vars, PascalCase classes, SCREAMING_SNAKE constants
- # TODO(ARIA): for anything left incomplete

## Definition of done
- All 7 scripts created and syntactically correct
- project.godot has all 6 autoloads registered
- project.godot has all input actions configured
- Main scene exists with correct node structure
- Main scene is set as the project's run scene in project.godot
```

---

### AGENT 2: Pilot — Player Ship & Components

```
You are Pilot, the player ship agent for AsteroidGame — a 2D space mining game built in Godot 4.5 with GDScript.

## Your mission
Build the complete PlayerShip scene with movement, components, mining beam, and weapon mount. The ship must feel spacey and fluid — physics-based movement with drift, NOT instant start/stop.

## First steps
Read these files before writing ANY code:
1. CLAUDE.MD — movement system, mining beam, combat weapon, scene architecture, constants, signal contracts
2. GAME_DESIGN.MD — resource types (for cargo tracking), upgrade effects (for stat queries)
3. AGENT_ASSIGNMENTS.MD — file ownership rules, API contracts, collision layers, groups

## Files you OWN (create these)
- scripts/player/player_ship.gd
- scripts/player/health_component.gd
- scripts/player/energy_component.gd
- scripts/player/cargo_component.gd
- scripts/player/mining_beam.gd
- scripts/player/weapon_mount.gd
- scenes/player/player_ship.tscn

## DO NOT TOUCH
- project.godot (owned by ARIA)
- scripts/systems/, scripts/utils/ (owned by ARIA)
- scenes/main/ (owned by ARIA)
- scripts/world/, scripts/asteroids/, scenes/world/, scenes/asteroids/ (owned by Navigator)

## What to build

### PlayerShip Scene (scenes/player/player_ship.tscn)
Root: CharacterBody2D named "PlayerShip"
Children:
- Sprite2D — placeholder ship (use a Polygon2D with a triangle/arrow shape pointing right, colored white or light blue, ~20px size)
- CollisionShape2D — CircleShape2D matching the sprite bounds
- MiningBeam (Node2D) — child nodes: RayCast2D + Line2D
- WeaponMount (Node2D) — positioned at ship front
- Camera2D — position_smoothing_enabled = true, zoom = Vector2(1.5, 1.5) (adjust for feel)
- HealthComponent (Node)
- EnergyComponent (Node)
- CargoComponent (Node)
- ThrusterParticles (CPUParticles2D) — positioned behind ship, emitting when thrusting

Attach scripts to: PlayerShip, MiningBeam, WeaponMount, HealthComponent, EnergyComponent, CargoComponent

### player_ship.gd
- class_name PlayerShip, extends CharacterBody2D
- Add to group "player" in _ready()
- Set collision layer = 1 (player)
- Set collision mask = 2 | 3 | 5 | 6 (enemies, asteroids, pickups, zones)

Movement (CRITICAL — must feel spacey):
- _physics_process: read WASD input → calculate thrust direction (Vector2)
- Apply thrust as acceleration: velocity += thrust_dir * THRUST_FORCE * delta
- Apply drag EVERY frame: velocity *= DRAG_COEFFICIENT (e.g., 0.98)
- Clamp velocity magnitude to effective_max_speed
- effective_max_speed = BASE_MAX_SPEED * (1.0 - clamp(cargo_weight / CARGO_OVERLOAD_THRESHOLD, 0.0, 1.0 - MIN_SPEED_FRACTION))
- Call move_and_slide()

Rotation:
- Ship rotates to face mouse: rotation = (get_global_mouse_position() - global_position).angle()
- Use lerp_angle() for smooth rotation, not instant snap

Thruster VFX:
- Scale ThrusterParticles emission based on thrust magnitude
- When no input, reduce/stop particles

Reference autoloads by name (EventBus, ResourceManager) — they will exist at runtime even though ARIA creates them in a separate agent session.

### health_component.gd
- class_name HealthComponent, extends Node
- @export max_health: int = 100
- @export max_shield: int = 0
- current_health, current_shield
- signal died() (local signal)
- take_damage(amount: int) — shield absorbs first, then HP. Emit EventBus.player_health_changed and EventBus.player_shield_changed.
- heal(amount: int)
- is_alive() -> bool
- On death: emit local died() signal AND EventBus.player_died

### energy_component.gd
- class_name EnergyComponent, extends Node
- @export max_energy: float = 100.0
- @export regen_rate: float = 15.0
- current_energy: float
- is_draining: bool = false
- drain(amount: float) -> bool — reduces energy, returns false if empty
- _process: if not draining, regenerate at regen_rate * delta
- Emit EventBus.player_energy_changed when energy changes

### cargo_component.gd
- class_name CargoComponent, extends Node
- @export max_capacity: float = 50.0
- carried: Dictionary = {} (mirrors ResourceManager.carried for local access)
- add_resource(type: String, amount: int) — also calls ResourceManager.add_carried(). Emits EventBus.cargo_changed.
- get_weight() -> float — sum of (amount * resource_weight) for each type
- clear() — called on death
- get_speed_penalty() -> float — returns 0.0 to (1.0 - MIN_SPEED_FRACTION) based on weight vs capacity

### mining_beam.gd
- class_name MiningBeam, extends Node2D
- Uses child RayCast2D aimed at mouse direction
- Uses child Line2D for beam visual (from ship to hit point or max range)
- @export mining_dps: float = 10.0
- @export mining_range: float = 200.0
- @export energy_drain: float = 20.0
- When fire_mining held AND energy available:
  - Drain energy via EnergyComponent
  - Update RayCast2D target_position toward mouse
  - If raycast hits an object in group "asteroids": call hit_object.take_damage(mining_dps * delta)
  - Show Line2D from Vector2.ZERO to hit point (local coords)
- When not firing: hide Line2D, stop drain

### weapon_mount.gd
- class_name WeaponMount, extends Node2D
- @export fire_cooldown: float = 0.3
- @export projectile_scene: PackedScene (leave null for now — projectile scenes come in Phase 8)
- Rotates to face mouse cursor
- On fire_weapon pressed AND cooldown ready: instance projectile, set direction toward mouse
- # TODO(Pilot): projectile scenes will be created in Phase 8

## Coding conventions
- class_name on every script
- Type hints on ALL function signatures
- @export variables grouped with @export_group()
- ## docblock at top of every script
- snake_case functions/vars, PascalCase classes, SCREAMING_SNAKE constants
- # TODO(Pilot): for anything left incomplete

## Definition of done
- PlayerShip scene runs standalone in the editor (Scene > Run Current Scene)
- WASD moves the ship with drift/inertia — releasing keys does NOT instantly stop
- Ship rotates smoothly to face mouse cursor
- Mining beam visually fires from ship toward mouse when LMB held
- Thruster particles appear behind ship when moving
- All component scripts created with correct signals and methods
- Ship is in group "player" and on collision layer 1
```

---

### AGENT 3: Navigator — World, Mothership & Asteroids

```
You are Navigator, the world-building agent for AsteroidGame — a 2D space mining game built in Godot 4.5 with GDScript.

## Your mission
Build the Mothership (dock zone, safe area), procedural asteroid generator, asteroid scene, asteroid field spawner, and resource drop pickups. You create the environment the player explores.

## First steps
Read these files before writing ANY code:
1. CLAUDE.MD — procedural asteroid system, resource system, scene architecture, constants, signal contracts
2. GAME_DESIGN.MD — resource table (types, weights, values, colors, spawn %), asteroid composition rules, threat escalation
3. AGENT_ASSIGNMENTS.MD — file ownership rules, API contracts, collision layers, groups

## Files you OWN (create these)
- scripts/world/mothership.gd
- scripts/asteroids/asteroid_generator.gd
- scripts/asteroids/asteroid.gd
- scripts/asteroids/asteroid_field.gd
- scripts/asteroids/resource_drop.gd
- scenes/world/mothership.tscn
- scenes/asteroids/asteroid.tscn
- scenes/asteroids/resource_drop.tscn

## DO NOT TOUCH
- project.godot (owned by ARIA)
- scripts/systems/, scripts/utils/, scenes/main/ (owned by ARIA)
- scripts/player/, scenes/player/ (owned by Pilot)

## What to build

### Mothership Scene (scenes/world/mothership.tscn)
Root: StaticBody2D named "Mothership"
Children:
- Sprite2D — placeholder (large polygon, hexagonal or rectangular, ~100px, dark grey)
- CollisionShape2D — matches sprite
- DockZone (Area2D) — larger radius (~150px), detects player entering/exiting
  - CollisionShape2D (CircleShape2D, radius ~150)
- SafeZone (Area2D) — even larger radius (~500px), used by WaveManager to prevent spawns
  - CollisionShape2D (CircleShape2D, radius ~500)
- BeaconLight (PointLight2D or Sprite2D) — small flickering light, placeholder for visual progression

Set body collision layer = 6 (zones)
Add to group "mothership"

### mothership.gd
- class_name Mothership, extends StaticBody2D
- @export dock_radius: float = 150.0
- @export safe_zone_radius: float = 500.0
- SAFE_ZONE_RADIUS constant exposed for WaveManager queries
- _on_dock_zone_body_entered(body): if body.is_in_group("player"): EventBus.player_docked.emit()
- _on_dock_zone_body_exited(body): if body.is_in_group("player"): EventBus.player_launched.emit()
- update_visuals(research_tiers: Dictionary) — adjusts beacon/lights based on mothership research progress (# TODO(Navigator): full visual progression in Phase 10)

### Procedural Asteroid Generator (scripts/asteroids/asteroid_generator.gd)
- class_name AsteroidGenerator
- This is a static utility class (no extends needed, or extends RefCounted)
- static func generate_polygon(radius: float, vertex_count: int = 12, irregularity: float = 0.4) -> PackedVector2Array
  - Create vertices at equal angular intervals around center
  - Vary each vertex radius by ±(irregularity * radius) randomly
  - Return the polygon vertices
- static func pick_resource_type() -> String
  - Weighted random based on spawn percentages from GAME_DESIGN.MD:
  - iron: 40%, copper: 30%, crystal: 15%, titanium: 10%, uranium: 5%
- static func get_resource_color(type: String) -> Color
  - iron: Color.GRAY, copper: Color.ORANGE, crystal: Color(0.3, 0.5, 1.0), titanium: Color.SILVER, uranium: Color.GREEN
- static func random_size_in_band(band: String) -> float
  - "small": randf_range(15.0, 30.0)
  - "medium": randf_range(40.0, 70.0)
  - "large": randf_range(90.0, 140.0)
- static func hp_for_radius(radius: float) -> int
  - Scale HP proportionally to size

### Asteroid Scene (scenes/asteroids/asteroid.tscn)
Root: RigidBody2D named "Asteroid"
Children:
- Polygon2D — vertices set at runtime by generator
- CollisionPolygon2D — same vertices
- (No sprite — Polygon2D IS the visual)

Set collision layer = 3 (asteroids)
Set collision mask = 1 | 4 (player, projectiles)
Add to group "asteroids"

### asteroid.gd
- class_name Asteroid, extends RigidBody2D
- var hp: int
- var max_hp: int
- var radius: float
- var size_band: String  # "small", "medium", "large"
- var primary_resource: String
- var secondary_resource: String  # can be ""
- func initialize(p_radius: float, p_band: String, p_resource: String, p_secondary: String = ""):
  - Set radius, size_band, primary/secondary resource
  - Generate polygon via AsteroidGenerator.generate_polygon(radius)
  - Set Polygon2D.polygon and CollisionPolygon2D.polygon
  - Set Polygon2D.color to AsteroidGenerator.get_resource_color(primary_resource)
  - Set HP via AsteroidGenerator.hp_for_radius(radius)
  - Apply random slow rotation: angular_velocity = randf_range(-0.5, 0.5)
  - Apply random drift velocity: linear_velocity = Vector2(randf_range(-20, 20), randf_range(-20, 20))
- func take_damage(amount: float):
  - hp -= int(amount)
  - Flash white briefly (modulate tween)
  - If hp <= 0: _die()
- func _die():
  - If size_band == "large": spawn 2-3 medium asteroids at position (call asteroid_field or self-manage)
  - If size_band == "medium": spawn 2-3 small asteroids
  - If size_band == "small" OR primary_resource == "uranium": spawn resource drops only
  - Spawn resource drops: instance ResourceDrop scenes, scatter around position
  - queue_free()
- Resource drop counts from GAME_DESIGN.MD asteroid composition rules

### Asteroid Field Spawner (scripts/asteroids/asteroid_field.gd)
- class_name AsteroidField, extends Node2D
- @export spawn_radius: float = 1500.0 (distance from player where asteroids exist)
- @export despawn_radius: float = 2000.0 (distance from player to delete asteroids)
- @export max_asteroids: int = 40
- @export spawn_interval: float = 0.5
- var asteroid_scene: PackedScene = preload("res://scenes/asteroids/asteroid.tscn")
- var resource_drop_scene: PackedScene = preload("res://scenes/asteroids/resource_drop.tscn")
- Find player via get_tree().get_first_node_in_group("player") — may be null early, handle gracefully
- _process or timer: if asteroid count < max, spawn new asteroid at random position within spawn_radius of player but outside viewport
- Periodically cull asteroids beyond despawn_radius from player
- Spawn with random size band (weighted: 50% small, 35% medium, 15% large)
- Call asteroid.initialize() with generated parameters
- Expose resource_drop_scene so asteroids can instance drops (pass via initialize or scene reference)

### Resource Drop Scene (scenes/asteroids/resource_drop.tscn)
Root: Area2D named "ResourceDrop"
Children:
- Sprite2D — small circle/diamond shape (~8px), color set at runtime
- CollisionShape2D — CircleShape2D radius 12

Set collision layer = 5 (pickups)
Set collision mask = 1 (player)
Add to group "resource_drops"

### resource_drop.gd
- class_name ResourceDrop, extends Area2D
- var resource_type: String
- var amount: int
- func initialize(type: String, p_amount: int):
  - resource_type = type
  - amount = p_amount
  - $Sprite2D.modulate = AsteroidGenerator.get_resource_color(type)
- _ready: connect body_entered to _on_body_entered
- _on_body_entered(body): if body.is_in_group("player"):
  - Call ResourceManager.add_carried(resource_type, amount)
  - EventBus.resource_collected.emit(resource_type, amount)
  - queue_free()
- Apply slow random drift: set a small linear velocity on spawn
- Optional: gentle bobbing animation (tween scale or position)

## Coding conventions
- class_name on every script
- Type hints on ALL function signatures
- @export variables grouped with @export_group()
- ## docblock at top of every script
- snake_case functions/vars, PascalCase classes, SCREAMING_SNAKE constants
- # TODO(Navigator): for anything left incomplete

## Definition of done
- Mothership scene exists with dock zone and safe zone Area2Ds
- Asteroid scene generates a unique polygon shape on every initialize() call
- Asteroids have visible color based on resource type
- Large asteroids split into medium, medium into small, small into resource drops
- ResourceDrop scene collects on player overlap
- AsteroidField spawner maintains ~40 asteroids around the player as they move
- All scenes run standalone in the editor
- Mothership is in group "mothership", layer 6
- Asteroids are in group "asteroids", layer 3
- Resource drops are in group "resource_drops", layer 5
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Identifier not found" for EventBus/ResourceManager | ARIA hasn't finished yet. The code is correct — just wait for ARIA's autoloads to be registered in project.godot. |
| Scenes can't find child nodes | Check node names match PascalCase convention. In .tscn, node names are exact strings. |
| Collision not working | Verify collision layers/masks match the table above. Layer = "I am this", Mask = "I detect these". |
| get_first_node_in_group("player") returns null | PlayerShip (Pilot's work) hasn't been added to the Main scene yet. Handle null gracefully. |
| Asteroid polygon looks wrong | Check that Polygon2D.polygon AND CollisionPolygon2D.polygon are set to the same vertices. |
