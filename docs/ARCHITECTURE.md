# Neon Graze — Architecture

How the code is organised. `docs/PLAN.md` tasks point at the sections below (for example "ARCHITECTURE.md §5"). Game rules and numbers live in `docs/DESIGN.md`; this file describes the structure that holds them.

> **Status:** first draft, written before any code exists. When building a task shows something here is wrong, propose a change to this file in the same task.

**Contents:** §1 Project settings · §2 Folders · §3 Autoloads and events · §4 Screens and run flow · §5 Inside a run · §6 Data resources · §7 Services · §8 Save · §9 Testing and the bot pilot · §10 Performance budget · §11 Builds and exports

---

## 1. Project settings

Set these in Project Settings (Advanced Settings on), or in `project.godot`. Task 0.3 applies them.

| Setting | Value | Why |
|---|---|---|
| `application/config/name` | `Neon Graze` | |
| `application/run/main_scene` | `res://main.tscn` | |
| `application/run/max_fps` | `60` | Gameplay ticks at 60 Hz. On 90/120 Hz phones, capping keeps motion smooth without physics interpolation. Revisit in task 2.9. |
| `application/boot_splash/bg_color` | `#05060D` | Matches the background, so no white flash. |
| `display/window/size/viewport_width` × `viewport_height` | `720` × `1280` | Design resolution (portrait 9:16). |
| `display/window/size/window_width_override` × `window_height_override` | `405` × `720` | Desktop window fits on a laptop screen. |
| `display/window/stretch/mode` | `canvas_items` | Crisp lines at any screen size. |
| `display/window/stretch/aspect` | `keep_width` | Width is always 720; taller phones see a slightly taller arena. |
| `display/window/handheld/orientation` | `Portrait` | |
| `input_devices/pointing/emulate_touch_from_mouse` | `true` | Mouse drags act like touch on desktop, so one input path works everywhere. |
| `input_devices/pointing/emulate_mouse_from_touch` | `true` (default) | UI buttons work on the phone. |
| `physics/common/physics_ticks_per_second` | `60` | All gameplay runs here. |
| `rendering/renderer/rendering_method` and `.mobile` | `gl_compatibility` | Works on cheap phones and in browsers. |
| `rendering/environment/defaults/default_clear_color` | `#05060D` | |
| `rendering/textures/vram_compression/import_etc2_astc` | `true` | Needed for Android export. |
| `debug/gdscript/warnings/untyped_declaration` | `Warn` | Enforces static typing (CLAUDE.md treats warnings as things to fix). |
| `audio/buses/default_bus_layout` | `res://audio/default_bus_layout.tres` | Buses: Master → Music, SFX. |

**2D physics layer names:** 1 `player`, 2 `enemies`, 3 `player_projectiles`. Enemy bullets and pickups don't use physics (§5).

**Input actions:** `pause` (Escape, P), `debug_overlay` (F3), `debug_boss` (F9), `debug_screenshot` (F12). Debug actions only do anything when `OS.is_debug_build()` is true. Touch is handled directly from `InputEventScreenTouch` / `InputEventScreenDrag`, not through actions. The Android back button arrives as `NOTIFICATION_WM_GO_BACK_REQUEST` and is handled by the ScreenRouter (§4).

**Autoloads**, in this order: `Events`, `Save`, `Audio`, `Services` (§3).

**`.gitignore`** must include `.godot/`, `build/`, `*.keystore`, `*.jks`, `.env`, `export_credentials.cfg`, `*.tmp`.

---

## 2. Folders

```
res://
├── main.tscn / main.gd          ScreenRouter: the only scene loaded at start (§4)
├── project.godot, export_presets.cfg, .gutconfig.json, CLAUDE.md, CREDITS.md
├── autoload/                    events.gd, save.gd, audio.gd, services.gd
├── core/                        Engine-independent helpers, no scenes
│   ├── pool.gd                  Node pool (§5)
│   ├── stats.gd                 Base values + modifiers (§5)
│   ├── run_rng.gd               Seeded RNG wrapper (§5)
│   └── formulas.gd              Pure maths: XP curve, credits, costs, score (DESIGN.md)
├── defs/                        Resource class scripts (class_name ...Data) (§6)
├── data/                        .tres files built from defs/ — all gameplay numbers
│   ├── tuning.tres
│   ├── weapons/  upgrades/  evolutions/  enemies/  bosses/
│   ├── sectors/  ships/  meta/  mutators/  palettes/
├── run/                         Everything that exists only during a run
│   ├── run.tscn / run.gd        Run root: owns the tick order (§5)
│   ├── run_state.gd             Hull, time, kills, score...
│   ├── player/                  player.tscn, player.gd, player_input.gd, bot_input.gd, input_source.gd
│   ├── weapons/                 weapon.gd, pulse.gd, scatter.gd, arc_laser.gd, seekers.gd,
│   │                            projectile.tscn/.gd, missile.tscn/.gd
│   ├── enemies/                 enemy.tscn, enemy.gd, behaviours/ (drift.gd, charge.gd, turret.gd, ...)
│   ├── bosses/                  boss.tscn, boss.gd, boss_part.tscn, attacks/
│   ├── bullets/                 bullet_field.gd, bullet_patterns.gd
│   ├── pickups/                 pickup_field.gd
│   └── systems/                 wave_director.gd, score_system.gd, overdrive_system.gd,
│                                level_up_system.gd, card_picker.gd, graze_chain.gd
├── ui/
│   ├── screens/                 main_menu, sector_select, ship_select, shop, settings, results, credits
│   ├── hud/                     hud.tscn and its widgets
│   ├── panels/                  level_up_panel, pause_panel, revive_panel, signal_lost
│   └── theme/                   neon_theme.tres, fonts/
├── fx/                          neon_shape.gd, shaders/ (grid.gdshader, glow), particles, screen_shake.gd
├── services/                    ad_service.gd, store_service.gd, leaderboard_service.gd,
│                                analytics_service.gd, fake_*.gd, android_*.gd (§7)
├── audio/                       music/, sfx/, default_bus_layout.tres
├── tests/
│   ├── unit/                    test_*.gd (GUT)
│   ├── smoke/                   smoke_run.tscn / .gd
│   ├── perf/                    stress.tscn / .gd
│   └── helpers/                 shared test fixtures
├── tools/                       balance.gd, check_16kb.sh, other dev scripts
├── addons/                      gut/, store and ad plugins (added with permission)
├── build/                       export output (git-ignored)
└── docs/
```

Rules: one scene and its script sit side by side with the same name. A folder holds one feature. Empty folders get a `.gitkeep` until they have files.

---

## 3. Autoloads and events

Autoloads are global singletons. Keep them few and small.

| Autoload | Job | Must not |
|---|---|---|
| `Events` | Holds signals only, for things that cross systems or screens. | Hold state or logic. |
| `Save` | Loads/saves the profile, owns the in-memory `profile` dictionary, applies migrations (§8). | Know about gameplay rules. |
| `Audio` | Music and SFX playback, buses, per-sound cooldowns, haptics (DESIGN.md §17). | Be called in a tight loop without cooldowns. |
| `Services` | Holds `ads`, `store`, `leaderboard`, `analytics` and the shared reward flow (§7). | Be bypassed: nothing else talks to plugins. |

**`Events` signals** (add more only when two systems that don't know each other need to talk):

```gdscript
signal enemy_killed(data: EnemyData, position: Vector2)
signal player_hit(hull_left: int)
signal grazed(position: Vector2, chain: int)
signal graze_chain_broken(final_chain: int)
signal overdrive_ready
signal overdrive_fired
signal xp_collected(amount: int)
signal level_up(new_level: int)
signal wave_started(index: int, banner: String)
signal boss_warning(data: BossData)
signal boss_phase_changed(phase_index: int)
signal boss_defeated(data: BossData)
signal run_ended(result: RunResult)
signal settings_changed
signal supporter_changed(owned: bool)
```

Inside a run, systems that `run.gd` owns get direct references from it. `Events` is for listeners that shouldn't be coupled: HUD, audio, juice, score.

---

## 4. Screens and run flow

`main.tscn` contains a `ScreenRouter` (Node, `PROCESS_MODE_ALWAYS`) with a `Screen` holder and a fade `CanvasLayer`.

- `ScreenRouter.go_to(screen: StringName, args: Dictionary = {})`: fades out, frees the current screen, instantiates the new one from a preloaded `PackedScene`, and calls `setup(args)` on it. Screens: `main_menu`, `sector_select`, `ship_select`, `shop`, `settings`, `credits`, `run`, `results`.
- Freeing and instantiating whole screens is allowed: it happens *between* runs, never during one.
- Android back / Escape: in a run → pause; on a menu screen → previous screen; on the main menu → quit confirmation.
- When the app loses focus (`NOTIFICATION_APPLICATION_FOCUS_OUT` / `PAUSED`), a running game pauses and `Save` writes any pending changes.

**`RunConfig`** (`RefCounted`) is everything needed to start a run, built by the menus and passed to `run` in `args`:

```gdscript
class_name RunConfig extends RefCounted
enum Mode { SECTOR, ENDLESS, DAILY }
var mode: Mode
var sector: SectorData
var ship: ShipData
var seed: int
var mutator: MutatorData          # null unless Daily
var use_meta: bool = true         # false in Daily
var start_at_boss: bool = false   # debug only
var bot: bool = false             # smoke test / balance tool
```

**`RunResult`** (`RefCounted`) is what the run hands to `results`: mode, sector id, cleared, score, time, kills, grazes, best chain, level, credits earned, revive used, new best flags. `Results` applies credits and bests through `Save`.

Flow: MainMenu → SectorSelect (or Endless / Daily button) → ShipSelect → Run → (SIGNAL LOST → Revive offer) → Results → MainMenu or Retry (same config, new seed unless Daily).

---

## 5. Inside a run

### Scene tree

```
Run (Node2D, run.gd)                 owns tick order, RunState, RunRng, pools
├── Camera2D                         screen shake = camera offset
├── Arena (Node2D)                   fixed playfield: arena Rect2 + background
│   └── Background (ColorRect)       grid shader (DESIGN.md §16)
├── World (Node2D)                   everything that moves, in draw order:
│   ├── Pickups (PickupField)        XP shards
│   ├── PlayerProjectiles (Node2D)   pool container
│   ├── Enemies (Node2D)             pool container (enemies, minis, mines)
│   ├── Boss (boss.tscn, inactive until the boss slot)
│   ├── Fx (Node2D)                  pooled particles, graze sparks
│   ├── Bullets (BulletField)        enemy bullets — above enemies and pickups
│   └── Player
├── Systems (Node)                   WaveDirector, ScoreSystem, OverdriveSystem, LevelUpSystem
├── Hud (CanvasLayer)
└── Panels (CanvasLayer)             LevelUpPanel, PausePanel, RevivePanel, SignalLost
```

The **arena** is the visible viewport rectangle at run start (720 wide, at least 1280 tall). Everything clamps, spawns and culls relative to it.

### Tick order

Only `run.gd` has gameplay `_physics_process`. Every tick it calls, in order:

1. `input_source.update()`, then `player.tick(dt)`: move toward the drag target.
2. `weapons.tick(dt)`: cooldowns and firing.
3. Pooled projectiles and missiles `tick(dt)`.
4. Each active enemy `tick(dt)` (its behaviour), then `boss.tick(dt)` if active.
5. `bullet_field.tick(dt, player)`: move, cull, hit, graze.
6. `pickup_field.tick(dt, player)`: pull and collect.
7. `wave_director.tick(dt)`, `overdrive_system.tick(dt)`, `score_system.tick(dt)`.
8. `run_state.time += dt`; the level-up check runs last and may pause the tree.

`dt = delta * time_scale`, where `time_scale` belongs to the run (hit-stop and boss slow-motion set it). `Engine.time_scale` stays 1 so UI and tweens are unaffected. A fixed order plus `RunRng` means the same seed and the same inputs give the same run.

**Pausing:** `get_tree().paused = true` for the pause and level-up panels. Run nodes are `PROCESS_MODE_PAUSABLE`; panels use `PROCESS_MODE_WHEN_PAUSED`.

### Player and relative drag

The Player reads movement from an **input source** (`input_source.gd`, base class) so the bot can replace the finger:

- `PlayerInput` (real touch/mouse) and `BotInput` (§9) both provide `get_target() -> Vector2` and the signal `overdrive_requested`.
- `PlayerInput` handles `InputEventScreenTouch` and `InputEventScreenDrag` in `_unhandled_input`, so UI buttons get touches first. It tracks **one finger only**: the first one down. Other fingers are ignored for movement but can still count as taps.

**Relative drag** (the ship follows the finger's movement, not its position):

```gdscript
# On touch press: remember where the ship and the finger were.
ship_start = player.position
finger_start = touch.position
# On drag: a pure static function in player_input.gd (tested in task 1.1).
static func compute_target(ship_start: Vector2, finger_start: Vector2, finger_now: Vector2,
        sensitivity: float, bounds: Rect2) -> Vector2:
    var target := ship_start + (finger_now - finger_start) * sensitivity
    return Vector2(clampf(target.x, bounds.position.x, bounds.end.x),
                   clampf(target.y, bounds.position.y, bounds.end.y))
```

- `bounds` is the arena shrunk by `tuning.arena_margin`.
- On release the target stays where it is, so a new touch anywhere never makes the ship jump.
- Each tick the ship moves toward the target with frame-rate-independent smoothing, capped at `max_move_speed`: `position = position.lerp(target, 1.0 - exp(-tuning.move_smoothing * dt))`.
- `sensitivity = tuning.drag_sensitivity * settings.drag_sensitivity`.
- **Taps:** a touch released within `tap_max_duration` and moved less than `tap_max_move` px is a tap. A press that starts within `double_tap_window` of the end of a previous tap emits `overdrive_requested`. That second press still starts a normal drag.

The Player has a `Hurtbox` (`Area2D`, layer `player`, mask `enemies`, circle radius = hitbox radius). Because `area_entered` fires only once, contact damage is checked each tick with `get_overlapping_areas()` when not invulnerable. Bullets don't use the hurtbox; BulletField checks the hitbox point directly.

### Pool (`core/pool.gd`)

```gdscript
class_name Pool extends RefCounted
func _init(scene: PackedScene, parent: Node, initial_size: int) -> void
func acquire() -> Node     # returns an inactive instance, activated
func release(node: Node) -> void
var active: Array[Node]     # what systems iterate; release removes with swap-remove
var grow_count: int         # how many times it had to grow
```

- All instances are created and added to `parent` when the run loads, then deactivated.
- Pooled scenes implement `activate()` / `deactivate()`. Deactivate hides the node, stops its processing, and disables collision with `set_deferred("monitoring", false)` / `set_deferred("monitorable", false)` (never toggle collision directly inside a physics callback).
- If the pool is empty, `acquire()` doubles the pool size, increments `grow_count` and calls `push_warning`. This is a safety net, not a plan: the smoke test fails if any `grow_count > 0`.
- Pooled nodes are never `queue_free`d during a run.
- Pool sizes come from `tuning.tres` (§10).

### Stats (`core/stats.gd`)

Every number that upgrades can change is a **stat**, identified by a `StringName` (list in DESIGN.md §8).

```gdscript
class_name Stats extends RefCounted
func set_base(stat: StringName, value: float) -> void
func add_modifier(mod: StatModifier, source: StringName) -> void
func remove_source(source: StringName) -> void
func get_value(stat: StringName) -> float   # cached; cache cleared when modifiers change
```

`value = (base + sum of ADD modifiers) × (1 + sum of MULT modifiers)`. Percent bonuses add together (+10% and +10% = +20%), they don't compound. Bases come from tuning and the ship. Passives, meta ranks and mutators add modifiers with a source id (`passive:overclock`, `meta:firepower`, `mutator:glass_cannon`).

### RunState (`run/run_state.gd`)

A `RefCounted` holding the facts about the current run: `hull`, `max_hull`, `time`, `kills`, `score`, `grazes`, `best_chain`, `level`, `xp`, `xp_to_next`, `rerolls_left`, `revive_used`, `invulnerable_until`, `ads_used: Dictionary`. `reset(config, stats)` restores everything for a restart. Rules such as "hull never below 0" and "damage ignored while invulnerable" live in its methods (`take_hit()`, `is_invulnerable()`) so they can be unit tested.

### RunRng (`core/run_rng.gd`)

Wraps `RandomNumberGenerator` seeded from `RunConfig.seed`. Methods: `randf()`, `randf_range()`, `randi_range()`, `chance(p)`, `pick(array)`, `pick_weighted(items, weights)`, `shuffle(array)`. Systems that need their own stream (waves, cards, loot) get a child RNG seeded from the main one, so adding a random call in one system doesn't change the others.

### Weapons

- `weapon.gd` (base, a `Node` under `Player/Weapons`): holds `WeaponData`, current `level`, a cooldown timer, and a reference to the run. Each tick it adds `dt × fire_rate` to its timer; each time the timer passes the level's cooldown it calls `fire()`, so long frames still fire the right number of times.
- `run.nearest_enemy(from: Vector2) -> Node2D` searches active enemies and boss parts. A weapon with no target doesn't fire, except Pulse, which fires straight up.
- `pulse.gd`, `scatter.gd`: acquire `projectile.tscn` from the pool. Projectiles move in a straight line, have a lifetime and pierce count, and hit enemies through `area_entered`.
- `arc_laser.gd`: no projectiles. It keeps a beam segment and every 0.1 s does a segment-vs-circle test against active enemies; drawn with `NeonShape`.
- `seekers.gd`: acquires `missile.tscn`. Missiles steer toward their target (limited turn rate); if the target is released, they pick the nearest live enemy; they explode in a radius on contact or at the end of their lifetime.
- Evolution replaces the weapon's data with the evolved `WeaponData` in place.

### Enemies

- `enemy.tscn`: a pooled `Area2D` (layer `enemies`, mask `player_projectiles`) with a circle shape whose radius comes from data. `activate(data: EnemyData, position: Vector2, hp_mult: float)` sets it up (`hp_mult` = wave × sector × mutator, DESIGN.md §9).
- It draws its outline from `data.shape_points` with `NeonShape` in `_draw()`. It redraws only when its look changes (hit flash, telegraph), not every frame.
- Per-instance state lives on the enemy in generic fields: `hp`, `velocity`, `state: int`, `timer: float`, `anchor: Vector2`, `locked_target: Vector2`, `angle: float`.
- **Behaviours** (`run/enemies/behaviours/*.gd`) extend `EnemyBehaviour`, which is a `Resource` with exported tuning fields and `func tick(enemy: Enemy, dt: float, run: Run) -> void`. The `.tres` enemy file embeds its behaviour. **Behaviours are shared between all enemies of a type, so they must be stateless**: all state goes in the enemy's fields.
- `take_damage(amount)` flashes white for 0.06 s; at 0 HP it emits `Events.enemy_killed`, asks PickupField to drop shards, calls `behaviour.on_death()` (the Splitter spawns its minis here) and releases itself to the pool.
- Enemies leaving the arena by more than 96 px are released without reward.

### BulletField (`run/bullets/bullet_field.gd`)

A `Node2D` that owns every enemy bullet. Rules in CLAUDE.md "Rules for bullets".

- **Storage** (size = `bullet_cap`, 2,000): `pos: PackedVector2Array`, `vel: PackedVector2Array`, `radius: PackedFloat32Array`, `color_idx: PackedByteArray`, `flags: PackedByteArray` (ALIVE, INSIDE_GRAZE, HIT), `free_slots: PackedInt32Array` used as a stack, `alive_count`, `high_water` (highest slot used + 1), `dropped_count`.
- `spawn(position, velocity, radius, color_idx) -> void` pops a free slot; with no free slot it increments `dropped_count` and returns.
- **Drawing:** one `MultiMeshInstance2D` with `instance_count = bullet_cap`, `use_colors = true`, and a quad mesh with a bullet shader (white core, coloured rim, soft edge). Each tick it fills one `PackedFloat32Array` (8 transform floats + 4 colour floats per instance) and assigns it to `multimesh.buffer` once. Dead slots get a zero-scale transform; `visible_instance_count = high_water`.
- **tick(dt, player)**, for every alive slot:
  1. `pos += vel * dt`.
  2. Outside the arena by more than 64 px → free the slot.
  3. Hit: `dist² < (r + hitbox_radius)²` and the player isn't invulnerable → set HIT, free the slot, `player.take_hit()`.
  4. Graze (DESIGN.md §5): inside `r + graze_radius` → set INSIDE_GRAZE. When a bullet with INSIDE_GRAZE (and not HIT) is now outside that ring and the player isn't invulnerable → clear the flag, emit `Events.grazed`. A bullet freed while inside the ring never grazes.
- `clear_all() -> PackedVector2Array` frees every bullet and returns their positions (Overdrive turns them into shards; boss defeats too).
- `clear_near(position, radius)` erases bullets in a circle without making shards (Hellfire Swarm blasts). Revive uses `clear_all()` and discards the positions.
- Lasers (Null Sovereign) are not bullets: the boss checks segment vs. hitbox itself.

### PickupField (`run/pickups/pickup_field.gd`)

The same pattern as BulletField for XP shards: packed arrays (`pos`, `vel`, `value`, `flags`), a free-list, one MultiMesh, cap `shard_cap` (300).

- `drop(position, value)`: when the field is full, the value is added to the nearest live shard instead (merging), and that shard is drawn larger. No XP is ever lost.
- In the tick, shards within the `pickup_radius` stat accelerate toward the player at `shard_pull_speed`; touching the ship collects them and adds XP × `xp_gain`.
- Shards don't despawn. At boss start and sector clear, every shard is pulled in.

### Systems (`run/systems/`)

- `GrazeChain` (`RefCounted`): chain count, timeout and best chain. Pure, unit tested.
- `ScoreSystem`: listens to kills, grazes and boss events and adds to `RunState.score` (formulas in `core/formulas.gd`).
- `OverdriveSystem`: the meter (0–100) and gains from grazes and kills. On `overdrive_requested` with a full meter: `bullet_field.clear_all()` → shards, damages on-screen enemies and the boss, gives invulnerability, empties the meter.
- `LevelUpSystem` + `CardPicker` (`RefCounted`, pure): builds hands of cards from what the player owns, the slots and the rules in DESIGN.md §6, using its own RNG stream. The system pauses the tree and shows `LevelUpPanel`; level-ups that arrive together queue up.
- `WaveDirector`: runs the sector's waves (§6, DESIGN.md §11), trickle spawns and scripted events, the wave banners, then the boss slot. In Endless it scales spawn rates and HP over time.

### Bosses (`run/bosses/`)

- `boss.tscn`: an `Area2D` like an enemy, plus optional `BossPart` children (separately damageable `Area2D`s, used by the Hydra Core). HP is the sum of the core and its parts.
- `boss.gd` keeps the phase index and runs the current phase's attacks in a loop. Attack resources (`attacks/*.gd`, extending `BossAttack`) are stateless like behaviours; per-attack timers live in the boss.
- Sequence: WARNING banner (3 s) and small enemies retreat off-screen → entry (invulnerable) → phases at HP thresholds, with a 1 s firing pause and `clear_all()` → shards on each phase change → defeat: `clear_all()` → shards, shard burst, run `time_scale = 0.25` for 0.5 s → sector clear.
- Debug (F9, or `RunConfig.start_at_boss`): jump straight to the boss with a level-appropriate build.

### Juice (`fx/`)

- `NeonShape`: static helpers that draw an outline from points: a bright core line plus a glow. Glow mode (real `WorldEnvironment` glow vs. a fake additive halo) is a setting, decided on a phone in task 2.1.
- Screen shake: a trauma value on the run's `Camera2D`; offset = trauma² × max × noise. Scaled by the shake setting; 0 when it's off.
- Hit-stop: sets the run's `time_scale` to 0.05 for a few frames. It never touches `Engine.time_scale`.
- Particles: a pool of `CPUParticles2D` bursts. Reduced flashing disables white full-screen flashes.

---

## 6. Data resources

Every class here is a `Resource` script in `defs/` with `class_name` and typed `@export` fields. Data files in `data/` are `.tres` files of these classes. Code reads data and never hard-codes the values. Edit `.tres` files in the Godot inspector or carefully as text, then run the import command to check they parse.

| Class | Key fields | Files |
|---|---|---|
| `TuningData` | Player and global numbers from DESIGN.md §4–§6, pool sizes and caps (§10) | `data/tuning.tres` |
| `StatModifier` | `stat: StringName`, `kind: ADD/MULT`, `value: float` | embedded |
| `WeaponLevel` | `damage`, `cooldown`, `count`, `spread_deg`, `speed`, `pierce`, `radius`, `lifetime`, `area`, `duration`, `chain`, `description` (fields a weapon doesn't use stay 0) | embedded |
| `WeaponData` | `id`, `display_name`, `icon_points`, `color`, `script_kind` (PULSE / SCATTER / ARC / SEEKER), `levels: Array[WeaponLevel]`, `weight`, `is_evolution` | `data/weapons/*.tres` |
| `UpgradeData` | `id`, `display_name`, `description`, `max_level`, `per_level: Array[StatModifier]`, `weight` | `data/upgrades/*.tres` |
| `EvolutionData` | `base_weapon`, `required_passive`, `result: WeaponData` | `data/evolutions/*.tres` |
| `EnemyData` | `id`, `hp`, `speed`, `radius`, `xp`, `score`, `color`, `shape_points`, `bullet_color_idx`, `behaviour: EnemyBehaviour` | `data/enemies/*.tres` |
| `EnemyBehaviour` (+ subclasses) | Behaviour-specific tuning, e.g. `ChargeBehaviour.telegraph_time` | embedded in enemy files |
| `BossData` | `id`, `display_name`, `max_hp`, `radius`, `color`, `shape_points`, `parts: Array[BossPartData]`, `phases: Array[BossPhase]`, `entry_time`, `xp_burst`, `score` | `data/bosses/*.tres` |
| `BossPhase` | `hp_ratio_start` (the phase begins when HP ratio ≤ this), `attacks: Array[BossAttack]`, `move_pattern`, `move_speed` | embedded |
| `BossAttack` (+ subclasses) | Ring / Fan / AimedBurst / Spiral / LaserSweep / Charge / Summon with their timings | embedded |
| `SectorData` | `id`, `number`, `display_name`, `palette: PaletteData`, `hp_mult`, `bullet_speed_mult`, `waves: Array[WaveData]`, `boss: BossData`, `music`, `unlock_requires: StringName` | `data/sectors/*.tres` |
| `WaveData` | `duration`, `banner`, `hp_mult`, `trickle: Array[TrickleEntry]`, `events: Array[SpawnEvent]` | embedded |
| `TrickleEntry` | `enemy: EnemyData`, `per_second`, `edge` | embedded |
| `SpawnEvent` | `time` (seconds into the wave), `enemy`, `count`, `formation` (SINGLE / LINE / V / RING / SINE), `edge` (TOP / LEFT / RIGHT / SIDES), `spacing`, `x_ratio` (−1 = random) | embedded |
| `ShipData` | `id`, `display_name`, `description`, `shape_points`, `color`, `start_weapon`, `base_stats: Array[StatModifier]`, `unlock_cost` | `data/ships/*.tres` |
| `MetaUpgradeData` | `id`, `display_name`, `max_rank`, `base_cost`, `cost_growth`, `per_rank: Array[StatModifier]` | `data/meta/*.tres` |
| `MutatorData` | `id`, `display_name`, `description`, `modifiers: Array[StatModifier]`, `enemy_hp_mult`, `enemy_speed_mult`, `enemy_fire_rate_mult`, `spawn_rate_mult`, `score_mult`, `graze_score_mult`, `weapon_slots`, `max_hull_override` | `data/mutators/*.tres` |
| `PaletteData` | `background`, `grid`, `player`, `player_shot`, `enemy_colors`, `bullet_rims: PackedColorArray`, `shard`, `overdrive`, `ui_text`, `ui_accent` | `data/palettes/*.tres` |

Ids are lowercase `snake_case` and never change once saved data refers to them (best scores, unlocks, meta ranks).

---

## 7. Services

The game never calls an ad, billing or leaderboard plugin directly. It calls the `Services` autoload, which holds one implementation of each interface:

```gdscript
# services/ad_service.gd
class_name AdService extends Node
signal rewarded_finished(placement: StringName, granted: bool)
func initialize() -> void                     # Android: consent first, then load ads
func is_rewarded_ready(placement: StringName) -> bool
func show_rewarded(placement: StringName) -> void
func privacy_options_required() -> bool
func show_privacy_options() -> void

# services/store_service.gd
class_name StoreService extends Node
signal purchase_finished(product_id: StringName, ok: bool, message: String)
signal ownership_changed(product_id: StringName, owned: bool)
func initialize() -> void                      # Android: connect, query, restore
func owns(product_id: StringName) -> bool
func purchase(product_id: StringName) -> void
func restore() -> void

# services/leaderboard_service.gd — is_available(), submit_daily(date_key, score)
# services/analytics_service.gd   — log_event(name, params); the default does nothing
```

| Implementation | Used when |
|---|---|
| `fake_ad_service.gd`, `fake_store_service.gd`, `fake_leaderboard_service.gd`, `fake_analytics_service.gd` | Desktop, web, tests, and Android when the `--fake-services` user arg is passed. Fakes can be set to succeed, fail or never answer, so tests cover every path. |
| `android_ad_service.gd` (AdMob plugin), `android_store_service.gd` (Google Play Billing plugin) | Android when the plugin singleton exists. |

**Shared reward flow** (`Services.request_reward(placement) -> bool`, awaited):

1. Supporter Pack owned → return `true` immediately, no ad.
2. Ad not ready → return `false`; the UI shows "No ad available right now".
3. Otherwise `show_rewarded` and await `rewarded_finished`, with a **15 s timeout** → `false`.
4. Placements: `revive`, `double_credits`, `extra_reroll`. Each is allowed once per run (tracked in `RunState.ads_used`).

The game is paused while an ad shows, and gameplay never waits on a service without a timeout.

---

## 8. Save

One JSON profile in `user://`. The `Save` autoload owns it.

```json
{
  "version": 1,
  "credits": 0,
  "selected_ship": "vector",
  "ships_unlocked": ["vector"],
  "sectors_cleared": [],
  "best_scores": { "sector_1": 0 },
  "endless_best": { "score": 0, "time": 0.0 },
  "daily_bests": { "2026-10-13": 0 },
  "meta_ranks": { "firepower": 0 },
  "hints_seen": [],
  "supporter_cached": false,
  "settings": { "music_volume": 0.8, "sfx_volume": 1.0, "haptics": true, "...": "DESIGN.md §18" },
  "totals": { "runs": 0, "kills": 0, "grazes": 0, "play_time": 0.0 }
}
```

- **Defaults:** `Save.default_profile()` returns a complete profile. Loading merges the file into the defaults, so new keys always exist.
- **Atomic write:** write `profile.json.tmp`, rename the current `profile.json` to `profile.json.bak`, then rename the tmp file to `profile.json`. A crash mid-write never destroys the last good save.
- **Load and recovery:** try `profile.json`. If it's missing or doesn't parse as a Dictionary with a `version`, try `profile.json.bak`. If both fail, use defaults. A corrupt main file is copied to `profile.corrupt.json` first, so nothing is silently lost. Recovery is logged with `push_warning`.
- **Version and migrations:** `CURRENT_VERSION` is a constant. `_migrate(data)` runs step functions `_migrate_1_to_2(data)` and so on until the data is current. Every format change adds a step and a test.
- **When to save:** after Results is applied, after a purchase or unlock, after settings change (debounced 1 s), and on focus out / pause / quit. Never during a run's update loop.
- `supporter_cached` lets the game show supporter perks offline; the store's restore on start is the source of truth.
- Daily bests older than 30 days are pruned on load.

---

## 9. Testing and the bot pilot

**Unit tests (GUT 9.x):** `tests/unit/test_*.gd`, each extending `GutTest`. `.gutconfig.json` points `dirs` at `res://tests/unit` and sets `include_subdirs: true`. Test the pure functions and `RefCounted` classes: formulas, Stats, Pool, RunState, GrazeChain, CardPicker, bullet patterns, the BulletField maths, Save round trips (in a temporary `user://test_*` path), WaveDirector timing with a fixed seed. Tests must not depend on frame timing or on each other.

**Smoke test** (`tests/smoke/smoke_run.tscn`, run with `--fixed-fps 60`):

- Until task 2.3: load `main.tscn`, wait 120 frames, quit 0.
- From task 2.3: build a `RunConfig` (Sector 1, Vector, seed 12345, `bot = true`), start the run and simulate 60 s of game time. Each second it checks and fails (`get_tree().quit(1)`, after printing the reason) on: a NaN or infinite position (player, enemies, bullets), any `Pool.grow_count > 0`, `BulletField.dropped_count > 0`, the shard cap exceeded, the run ending in an unexpected state. Success prints `SMOKE OK` and quits 0.
- Level-up panels are auto-resolved by the bot's card choice, so the tree never stays paused.

**Bot pilot** (`run/player/bot_input.gd`, an input source like `PlayerInput`):

- Each tick it scores a few candidate positions around the ship (a ring of 8 directions at 40 px, plus staying still) by danger (enemy bullets and enemies within 140 px, weighted by distance and approach speed) and attraction (nearest shard cluster, a "home" spot at 75% of the arena height). It moves to the best one.
- It requests Overdrive when the meter is full and more than 40 bullets are within 250 px.
- It picks cards by a simple priority list (evolution > owned weapon > new weapon > passive), using the run's RNG to break ties.
- It plays worse than a human. It exists to catch crashes and big balance outliers, not to tune difficulty.

**Perf test** (`tests/perf/stress.tscn`): spawns 150 enemies (drifters that wrap around the arena) and keeps 1,500 bullets alive (a turret ring refilling), runs 10 s, and prints the average of `Performance.TIME_PHYSICS_PROCESS` and `TIME_PROCESS` in ms, plus the fps. Results go in PROGRESS.md.

**Debug overlay** (F3, debug builds): fps, physics ms, alive bullets / cap, enemies, shards, pool usage, seed.

---

## 10. Performance budget

Target: a steady **60 fps on a cheap 2020-era Android phone** (Snapdragon 6xx / Helio G-class, 3–4 GB RAM).

| Item | Budget |
|---|---|
| Frame | 16.6 ms total; gameplay script (physics tick) ≤ 6 ms with 1,500 bullets + 150 enemies |
| Enemy bullets | 1,500 typical peak, **2,000 hard cap** |
| Enemies (pool) | 220 (150 on screen at peak) |
| Player projectiles (pool) | 400; missiles 40 |
| XP shards | 300 cap (merging above) |
| Particle bursts (pool) | 48 |
| Draw calls | < 100 in a busy frame (bullets and shards are one MultiMesh each) |
| Memory | < 300 MB |
| APK/AAB size | < 60 MB |

Habits that keep it there: no allocations in loops (reuse arrays, avoid `Array`/`Dictionary` literals per frame), squared distances, cache node references, no `get_node()` in ticks, enemies redraw only when their look changes, no per-bullet signals. Measure with the perf test before and after any optimisation, and record the numbers in PROGRESS.md.

---

## 11. Builds and exports

| Preset | Output | Notes |
|---|---|---|
| `Web` | `build/web/index.html` | Thread support off (works on itch.io without special headers). Task 1.10. |
| `Android` | `build/android/neon-graze-debug.apk` | Debug, portrait, arm64-v8a + armeabi-v7a, `VIBRATE` permission, package `com.<yourname>.neongraze`. Task 2.1. |
| `Android Release` | `build/android/neon-graze.aab` | Gradle build (needed for plugins), AAB, target SDK 36, release keystore. Task 3.16. |

- **Versions:** `version_name` is `MAJOR.MINOR.PATCH` (start at `0.1.0`); `version_code = MAJOR*10000 + MINOR*100 + PATCH`, and it must go up with every upload to Play.
- **Keystore:** the upload keystore and its passwords never enter the repo. For command-line release builds, set `GODOT_ANDROID_KEYSTORE_RELEASE_PATH`, `GODOT_ANDROID_KEYSTORE_RELEASE_USER` and `GODOT_ANDROID_KEYSTORE_RELEASE_PASSWORD` in the shell. Godot keeps editor-entered credentials in `.godot/export_credentials.cfg`, which is git-ignored. Check `export_presets.cfg` has no passwords before committing.
- **16 KB pages:** `tools/check_16kb.sh build/android/neon-graze.aab` unzips the AAB, lists every native `.so` (engine and plugins) and checks that each `LOAD` segment is 16 KB aligned. Release is blocked if any fails.
- **Ad IDs:** Google's test ad unit IDs until task 5.1. Real IDs go in a release-only config read by `android_ad_service.gd`.
- The debug "screenshot mode" and debug commands are compiled out of release behaviour with `OS.is_debug_build()` checks.
