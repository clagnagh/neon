# Neon Graze — Claude Code instructions

Neon Graze is a portrait, one-thumb, survivors-like shoot-'em-up for Android (and web for playtests), built in **Godot 4.6+ with GDScript** and the **Compatibility renderer**. You drag to move, weapons fire on their own, and grazing enemy bullets fills an Overdrive meter that clears the screen.

- `docs/PLAN.md` — the task list. Work on exactly one task at a time.
- `docs/DESIGN.md` — what the game is: every number, enemy, weapon and rule. Gameplay values come from here.
- `docs/ARCHITECTURE.md` — how the code is built: settings, folders, systems, save, services, tests.
- `docs/PROGRESS.md` — task log, decisions, perf numbers, "What I learned", ideas for later.

## Who you're working with

The developer is new to coding and to Godot, and is building this to sell. Explain in plain words, define Godot terms the first time you use them, and keep changes small enough to understand. When a task needs a decision that isn't in the docs, ask; don't guess. Don't add features, settings or polish that the current task doesn't ask for; put good ideas in "Open issues / ideas for later" in PROGRESS.md instead.

## How to do a task

1. Read the task in `docs/PLAN.md` and every doc section it references, before planning.
2. In plan mode, list files to create or change, the tests you'll add, and open questions.
3. Build it. Put gameplay numbers in data files, not code. Write the tests.
4. Run the import, unit and smoke commands below. Fix every error **and warning** before saying it's done.
5. Tell the developer what to check by hand (the task's "You check" list).
6. Only when they say it works: tick the box in PLAN.md, add a row to the PROGRESS.md task log, and commit as `Task X.Y: <short title>`.

If DESIGN.md or ARCHITECTURE.md is wrong or missing something, say so and propose the doc change; keep docs and code in sync.

## Commands

`$GODOT` is the path to the Godot 4.6+ executable (set in the developer's shell profile). These work in bash, zsh and Git Bash; in PowerShell use `& $env:GODOT` instead of `"$GODOT"`.

```bash
# Import assets and check every script and resource parses. Run first, and after adding/renaming files.
"$GODOT" --headless --path . --import

# Unit tests (GUT). Exit code 0 = all passed.
"$GODOT" --headless --path . -s res://addons/gut/gut_cmdln.gd -gconfig=res://.gutconfig.json -gexit

# Smoke test: boots the game (later: a bot plays Sector 1). Exit code 0 = pass.
"$GODOT" --headless --path . --fixed-fps 60 res://tests/smoke/smoke_run.tscn

# Perf test: 1,500 bullets + 150 enemies for 10 s, prints average script ms per frame. Windowed on purpose.
"$GODOT" --path . res://tests/perf/stress.tscn

# Exports (presets are added in tasks 1.10, 2.1 and 3.16)
"$GODOT" --headless --path . --export-release "Web" build/web/index.html
"$GODOT" --headless --path . --export-debug "Android" build/android/neon-graze-debug.apk
adb install -r build/android/neon-graze-debug.apk
"$GODOT" --headless --path . --export-release "Android Release" build/android/neon-graze.aab
```

A test run only passes if the exit code is 0 **and** the output has no lines starting with `ERROR:` or `SCRIPT ERROR:`. Godot often prints errors and still exits 0, so always read the output.

## Code rules

- **Godot 4 GDScript only.** Use `@export`, `@onready`, `await`, `super()`, typed arrays (`Array[Enemy]`). Never Godot 3 syntax (`export var`, `onready var`, `yield`).
- **Static typing everywhere**: typed variables, parameters and return types (`-> void` too). Use `:=` only when the type is obvious from the right-hand side.
- **Files stay under 300 lines.** Split by responsibility before you reach that.
- **No magic numbers.** Gameplay values live in `data/*.tres` (via Resource classes in `defs/`) or `data/tuning.tres`. Named constants are fine for non-gameplay values (array strides, layer bits).
- **Logic in plain functions, nodes stay thin.** Maths and rules (drag target, XP curve, card picking, credits, patterns) go in `static func`s or `RefCounted` classes so tests can call them without a scene.
- **No creating or freeing nodes during a run.** Everything that appears in a run comes from a `Pool` filled at run start, or lives in BulletField / PickupField arrays. Never `instantiate()`, `queue_free()` or `add_child()` inside a run's update loop.
- **Seeded randomness only in gameplay.** Code under `run/` uses `RunRng`; never global `randf()`, `randi()`, `randi_range()` or `Array.shuffle()`/`pick_random()`.
- **Gameplay runs in `_physics_process`** (fixed 60 ticks), driven in a fixed order by `run.gd`. `_process` is for visuals only.
- **Signals:** cross-system or cross-screen events go through the `Events` autoload; parent↔child talk uses direct references or local signals. Use `%UniqueName` or `@export` node references, never long `../../` paths.
- **Never block on a service.** Ads, billing and leaderboards can fail or never answer; every wait has a timeout and a fallback.
- Errors: `push_error()` for bugs, `push_warning()` for recoverable problems. `print()` only in tests and tools.
- Names: `snake_case.gd` files and functions, `PascalCase` for `class_name` and nodes, `UPPER_CASE` constants, `_private` helpers.

## Rules for bullets

Enemy bullets are the game's biggest performance risk and its most important visual. These rules are fixed:

1. **Enemy bullets are never nodes.** They live in `BulletField` (`run/bullets/bullet_field.gd`): packed arrays (position, velocity, radius, colour, flags), a free-list of slots, and **one** `MultiMesh` that draws them all.
2. **Only BulletField spawns, moves, culls, draws and collides bullets.** Enemies and bosses call `bullet_field.spawn(...)`, usually with directions from the pure functions in `bullet_patterns.gd` (`ring`, `fan`, `aimed_burst`, `spiral_step`).
3. **Collision is circle vs. the player's hitbox point**, done in BulletField's update loop with squared distances, no physics engine. The hit check runs before the graze check.
4. **Graze follows DESIGN.md §5 exactly**: awarded once per bullet, never for a bullet that hit, never while invulnerable.
5. **Hard cap of 2,000 live bullets.** At the cap, `spawn` drops the bullet and counts it; the smoke test fails if that count is above 0. Bullets more than 64 px outside the arena are culled.
6. **No per-bullet allocations or signals in the update loop.** No new Arrays/Dictionaries per bullet; `Events.grazed` is the only per-bullet signal.
7. **Enemy bullets are always the easiest thing to see**: white core, saturated coloured rim, drawn above enemies and pickups. Player shots are dimmer and drawn below them.
8. Overdrive clears bullets through `bullet_field.clear_all()`, which returns their positions so they can become XP shards.
9. Player projectiles (Pulse, Scatter, Seekers) are a different thing: pooled `Area2D` nodes on the `player_projectiles` physics layer.

## Testing rules

- Every task adds or updates GUT tests in `tests/unit/test_*.gd` for its logic. Test pure functions directly; avoid timing-dependent scene tests.
- Never delete, skip or weaken a failing test to get green. Fix the code, or ask if the test is wrong.
- The smoke test must pass before every commit.

## Safety and git

- **Ask before downloading anything** (GUT, plugins, assets, fonts) and say where it comes from and its licence. Only add assets whose licence allows commercial use, and list them in `CREDITS.md`.
- Commit only when the developer asks. Never commit `.godot/`, `build/`, `*.keystore`, `*.jks`, passwords, or `.env` files. Keystore passwords go in Godot's export credentials or environment variables, never in the repo.
- Never use real AdMob ad unit IDs before task 5.1; use Google's test IDs.
- No analytics, network calls or data collection beyond what DESIGN.md §15 and the privacy policy allow.
