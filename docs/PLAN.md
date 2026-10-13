# Neon Graze — Build Plan for Claude Code

A step-by-step plan for building Neon Graze with Claude Code, from an empty folder to a game on Google Play. Each task is sized for one or two Claude Code sessions. Claude reads the task from this file, so you only need to say which task to do.

**Who does what:** tasks marked **(Claude)** are coding work you hand to Claude Code. Tasks marked **(You)** are things only you can do: installing tools, playing, accounts, stores. **(Both)** means Claude writes code and you test or decide.

**Time estimates** assume you're new to coding, working 10–15 hours a week. They include your playtesting and learning time, not just Claude's coding time.

## How to run a session

1. Open a terminal in the project folder and run `claude`.
2. Starting a new task? Type `/clear` first so Claude starts with a clean context (CLAUDE.md is still loaded).
3. Type `/plan Let's do task 1.3 from docs/PLAN.md` (or press Shift+Tab to switch into plan mode, then ask). Claude reads the task and proposes a plan without changing files.
4. Read the plan. Ask about anything you don't understand. Then approve it.
5. Claude builds it, writes tests, and runs the unit and smoke tests.
6. You play it: press F5 in the Godot editor, or ask Claude to run it through the Godot MCP server. Use the "You check" list for the task.
7. Ask Claude to explain the change (template below). Write one line in "What I learned" in `docs/PROGRESS.md`.
8. Say "It works, tick it off and commit." Claude ticks the task below, logs it and commits.

**Handy commands:** `/diff` shows what changed · `/rewind` rolls code and conversation back to a checkpoint if something went wrong · `/compact` shrinks a long conversation · `/context` shows what's using space · `/usage` shows your plan limits · `/code-review` reviews the current changes for bugs (use it at the end of each phase).

**Plan limits:** on the Pro plan you have a 5-hour session limit and a weekly limit. Start big tasks at the beginning of a session window, and use `/clear` between tasks so you don't spend usage on old context.

## Prompt templates

**Start a task**

```
/plan Let's do task X.Y from docs/PLAN.md. Read the docs it references first. List the files you'll create or change, the tests you'll add, and anything you need me to decide.
```

**Explain what you did**

```
Explain what you just built as if I'm new to Godot. For each new file, one or two sentences on what it does. Then teach me the one Godot concept that matters most here, with a tiny example.
```

**Report a bug**

```
Bug: I did [what you did]. I saw [what happened]. I expected [what should happen]. Here's the output: [paste error]. Find the root cause and explain it before changing any code.
```

**Tune the feel**

```
[Thing] feels [too slow / too easy / floaty]. Suggest 2–3 number changes in the data files only, tell me which and why, and update docs/DESIGN.md to match.
```

**Keep the code healthy (every few tasks)**

```
Check the files we've changed recently against the rules in CLAUDE.md and docs/ARCHITECTURE.md. List problems (files over 300 lines, missing types, magic numbers, instantiating during a run, missing tests) and propose fixes, smallest first. Don't change anything yet.
```

**Run and look**

```
Use the godot MCP server to run the project for 20 seconds, then read the debug output and fix any errors or warnings.
```

## Checklist

Claude ticks these off as tasks are finished.

### Phase 0 — Setup (week 1)

- [ ] 0.1 Install the tools (You)
- [ ] 0.2 Create the project and add this kit (You)
- [ ] 0.3 Project settings and skeleton (Claude)
- [ ] 0.4 Test harness: GUT, smoke test, perf test (Claude)
- [ ] 0.5 Connect the Godot MCP server (You)
- [ ] 0.6 Start your Google Play developer account (You)

### Phase 1 — Prototype: is it fun? (weeks 2–4)

- [ ] 1.1 Player ship and relative drag (Claude)
- [ ] 1.2 Pools, Stats and the Pulse Cannon (Claude)
- [ ] 1.3 Enemies from data: Drifter and Charger (Claude)
- [ ] 1.4 Hull, damage and the restart loop (Claude)
- [ ] 1.5 BulletField, bullet patterns and the Spinner Turret (Claude)
- [ ] 1.6 Graze and Overdrive (Claude)
- [ ] 1.7 XP shards and level-up cards (Claude)
- [ ] 1.8 HUD and pause (Claude)
- [ ] 1.9 Neon look and juice, version 1 (Claude)
- [ ] 1.10 Web build and the fun test (Both)

### Phase 2 — Vertical slice: one polished sector on a phone (weeks 5–8)

- [ ] 2.1 Android setup and first phone build (Both)
- [ ] 2.2 Seeded randomness (Claude)
- [ ] 2.3 Waves and Sector 1 data (Claude)
- [ ] 2.4 Swarm and Splitter enemies (Claude)
- [ ] 2.5 Boss framework and the Gatekeeper (Claude)
- [ ] 2.6 Run flow: menu, sector select, results (Claude)
- [ ] 2.7 Save system and credits (Claude)
- [ ] 2.8 Audio and haptics (Both)
- [ ] 2.9 Performance pass on the phone (Both)
- [ ] 2.10 Juice, version 2 (Claude)
- [ ] 2.11 Start recruiting testers (You)

### Phase 3 — MVP: content complete (weeks 9–16)

- [ ] 3.1 Scatter, Arc Laser and Seeker Missiles (Claude)
- [ ] 3.2 All passives, card rules and evolutions (Claude)
- [ ] 3.3 Sniper, Shielded and Mine-layer (Claude)
- [ ] 3.4 Sectors 2 and 3 (Both)
- [ ] 3.5 Hydra Core and Null Sovereign bosses (Claude)
- [ ] 3.6 Ships and ship select (Claude)
- [ ] 3.7 Meta upgrade shop (Claude)
- [ ] 3.8 Endless and Daily modes (Claude)
- [ ] 3.9 Settings and accessibility (Claude)
- [ ] 3.10 Menu polish, first-run hints, credits and privacy screen (Claude)
- [ ] 3.11 Services layer with fakes (Claude)
- [ ] 3.12 Rewarded placements: revive, double credits, reroll (Claude)
- [ ] 3.13 Real ads: AdMob and consent (Both)
- [ ] 3.14 Supporter Pack with Google Play Billing (Both)
- [ ] 3.15 Balance simulation and tuning (Both)
- [ ] 3.16 Release build setup (Both)
- [ ] 3.17 Optional: daily leaderboard and analytics (Both)
- [ ] 3.18 Bug bash and content freeze (Both)

### Phase 4 — Closed testing (weeks 17–20)

- [ ] 4.1 Play Console app setup and policy forms (You)
- [ ] 4.2 Privacy policy and credits page (Both)
- [ ] 4.3 Store assets: icon, screenshots, video (Both)
- [ ] 4.4 Closed test: 12+ testers for 14 days (You)
- [ ] 4.5 Crash and feedback loop (Both)
- [ ] 4.6 Apply for production access (You)

### Phase 5 — Launch (weeks 21–22)

- [ ] 5.1 Release candidate (Both)
- [ ] 5.2 Staged rollout and launch posts (You)

### Phase 6 — Live updates (week 23 onward)

- [ ] 6.1 First update (Both)

---

## Phase 0 — Setup (week 1)

### 0.1 Install the tools (You · 1–2 h)

- Subscribe to Claude Pro and install Claude Code by following the official quickstart at code.claude.com/docs.
- Install Git, Node.js 18 or newer (needed by the Godot MCP server) and the standard Godot 4.6+ build (not the .NET version).
- Set the `GODOT` environment variable to the full path of the Godot executable in your shell profile, then open a new terminal and check that `"$GODOT" --version` prints the version.
- Android tools come later (task 2.1).

**Done when:** `claude`, `git --version`, `node --version` and `"$GODOT" --version` all work in a terminal.

### 0.2 Create the project and add this kit (You · 30 min)

1. In the Godot Project Manager, create a project called `neon-graze` with the Compatibility renderer and Git version control metadata. Close Godot.
2. Copy `CLAUDE.md` into the project folder (next to `project.godot`) and the `docs/` folder alongside it.
3. In a terminal in that folder: `git init` (if Godot didn't), `git add -A`, `git commit -m "Initial project and plan"`.
4. Run `claude`, then type `/memory` to confirm CLAUDE.md is loaded. You don't need `/init`; CLAUDE.md is already written.

**Done when:** Claude can answer "What are the rules for bullets in this project?" correctly.

### 0.3 Project settings and skeleton (Claude · 1 h)

**Goal:** a correctly configured, empty project with the folder layout and autoload stubs.

- Apply every setting in ARCHITECTURE.md §1.
- Create the folders from ARCHITECTURE.md §2 (with a `.gitkeep` in empty ones).
- Create stub autoloads `Events`, `Save`, `Audio`, `Services` and register them.
- Create `main.tscn` with a tiny `ScreenRouter` and a placeholder main menu showing "NEON GRAZE — tap to start".
- Check `.gitignore` covers `.godot/`, `build/` and `*.keystore`.

**Done when:** F5 opens a portrait window showing the placeholder menu; the headless import command prints no errors. **You check:** the window is tall and narrow; clicking works.

### 0.4 Test harness (Claude · 1–2 h)

**Goal:** the three test commands in CLAUDE.md work, so every later task can be checked automatically.

- Ask me before downloading GUT (the Godot 4 version, 9.x) into `addons/gut/`, enable it, and add `.gutconfig.json` pointing at `res://tests/unit`.
- Add one real test (for example, that the XP formula in DESIGN.md §6 gives 6, 32 and 91 for levels 1, 5 and 10). Put the formula in a small pure function.
- `tests/smoke/smoke_run.tscn`: for now it loads `main.tscn`, waits 120 frames, checks nothing crashed, and quits with exit code 0 (or 1 on failure). Later tasks make it run real bot gameplay.
- `tests/perf/stress.tscn`: a stub that prints "perf: not implemented yet" and quits.

**Done when:** the unit test command and smoke command both pass, with no `ERROR:` lines. **Learn:** ask Claude "What's a unit test and why is it useful when an AI writes the code?"

### 0.5 Connect the Godot MCP server (You · 15 min)

In a terminal (not inside Claude Code), run:

```
claude mcp add godot -e GODOT_PATH=/full/path/to/godot -- npx @coding-solo/godot-mcp
```

Restart Claude Code, type `/mcp` and check that `godot` is connected. Then ask Claude: "Use the godot MCP server to run the project and show me the debug output."

**Done when:** Claude can launch your game and read its output.

### 0.6 Start your Google Play developer account (You · 30 min + waiting)

Register a Play Console developer account and start identity verification now; it can take days. Decide personal vs organization (personal is fine for most solo devs, but remember the 12-tester / 14-day closed test). Write the status in `docs/PROGRESS.md`.

**Also this week:** do the official Godot "Your first 2D game" tutorial by hand, without AI (about 3 hours). It makes everything Claude writes easier to follow.

---

## Phase 1 — Prototype: is it fun? (weeks 2–4)

**Goal:** the core loop (drag, auto-fire, dodge, graze, level up) is playable on desktop and in a browser. Placeholder visuals are fine.

### 1.1 Player ship and relative drag (Claude · 2 h)

**Goal:** a ship you can move with one thumb (or the mouse).

- `run/run.tscn` with Arena and World (ARCHITECTURE.md §5); the main menu's tap starts it.
- `run/player/`: Player scene drawn as a simple cyan arrow (placeholder `_draw()`), with a visible hitbox dot.
- `player_input.gd`: relative drag exactly as ARCHITECTURE.md §5 describes, sensitivity and smoothing read from `data/tuning.tres`, clamped to the arena.
- Create `TuningData` (resource script) and `data/tuning.tres` with player values from DESIGN.md §4.

**Tests:** a pure function `compute_target(ship_start, finger_start, finger_now, sensitivity, bounds)` covering normal movement, sensitivity and clamping. **Done when:** tests pass; smoke test passes. **You check:** drag anywhere and the ship follows the movement of your finger, not its position; releasing and touching somewhere else doesn't make it jump; it can't leave the screen.

### 1.2 Pools, Stats and the Pulse Cannon (Claude · 2–3 h)

**Goal:** the ship auto-fires at a target.

- `core/pool.gd` as described in ARCHITECTURE.md §5.
- `core/stats.gd`: base values plus a list of modifiers; for now only ship base values.
- `WeaponData` resource and `data/weapons/pulse_cannon.tres` (DESIGN.md §7, levels included even if unused yet).
- `run/weapons/weapon.gd` (cooldown, asks Run for the nearest enemy) and `pulse.gd`; pooled `projectile.tscn` on the player_projectiles physics layer.
- A temporary "training dummy" target that drifts around so there is something to shoot.

**Tests:** pool acquire/release/grow; cooldown fires the right number of times over 3 s; Stats returns modified values correctly. **You check:** bolts fire every 0.3 s toward the dummy; there are no pauses or stutters after a minute. **Learn:** "Why do we pool bullets instead of creating and deleting them?"

### 1.3 Enemies from data: Drifter and Charger (Claude · 3 h)

**Goal:** real enemies built from data files.

- `EnemyData` resource; `enemy.tscn` (pooled Area2D on the enemies layer) that draws its shape from `shape_points` and colour.
- Behaviour scripts `drift.gd` and `charge.gd` (Charger: approach, flash for 0.6 s, dash to the locked position).
- `data/enemies/drifter.tres` and `charger.tres` from DESIGN.md §9.
- Enemies take damage from projectiles, flash white on hit and die at 0 HP (emitting `Events.enemy_killed`).
- A temporary spawner: one drifter per second from the top, a charger every 5 s (replaced in task 2.3). Remove the dummy.

**Tests:** enemy damage and death; charger state changes (approach → telegraph → dash). **You check:** enemies come in, get shot, flash and die; chargers clearly warn before dashing.

### 1.4 Hull, damage and the restart loop (Claude · 1–2 h)

**Goal:** you can lose, and get straight back in.

- `RunState` (ARCHITECTURE.md §5) with hull, time, kills, score.
- Contact damage from enemies through the player hurtbox; 1 s invulnerability with blinking.
- At 0 hull: a simple "SIGNAL LOST" overlay with time survived and kills; tap to restart the run.

**Tests:** damage ignored while invulnerable; hull never below 0; RunState reset on restart. **You check:** getting touched loses one hull and the ship blinks; losing all hull shows the overlay; tap restarts instantly.

### 1.5 BulletField, bullet patterns and the Spinner Turret (Claude · 3–4 h)

**Goal:** enemy bullets that are fast to process even at 1,500 on screen.

- `run/bullets/bullet_field.gd` exactly as ARCHITECTURE.md §5 describes (arrays, free-list, one MultiMesh, cull, hit check). Bullets have a white core and coloured rim.
- `bullet_patterns.gd` pure functions: `ring`, `fan`, `aimed_burst`, `spiral_step`.
- `turret.gd` behaviour and `data/enemies/spinner_turret.tres` (DESIGN.md §9). Add spinners to the temporary spawner every 12 s.
- Fill in `tests/perf/stress.tscn`: 1,500 bullets + 150 enemies for 10 s; print the average script time per frame.

**Tests:** pattern maths (ring of 8 gives 8 evenly spaced angles; fan centred on the aim angle); free-list reuse; bullets culled off-screen; hit detected at the right distance. **Done when:** tests pass; the perf test prints a number (write it in PROGRESS.md as the baseline). **You check:** spinners stop near the top and spray a rotating stream; getting hit by a bullet costs one hull. **Learn:** "Why are enemy bullets stored in arrays instead of being nodes?"

### 1.6 Graze and Overdrive (Claude · 2–3 h)

**Goal:** the signature mechanic.

- Graze detection inside BulletField (rules in DESIGN.md §5): once per bullet, not while invulnerable; emits `Events.grazed`.
- Graze chain and graze score in a first version of `ScoreSystem`.
- `OverdriveSystem`: meter from grazes and kills, double-tap detection from player input, the pulse (clear bullets, damage on-screen enemies, 0.5 s invulnerability). For now cleared bullets just disappear (they become shards in 1.7).
- A temporary text readout of the meter and chain on screen (the real HUD comes in 1.8).

**Tests:** a bullet passing within the graze radius counts once; a bullet that hits doesn't also graze; no graze while invulnerable; chain grows and resets after 1.5 s; meter caps at 100; double-tap timing rules. **You check:** brushing past bullets fills the meter and you can feel it; double-tap when full wipes the screen satisfyingly.

### 1.7 XP shards and level-up cards (Claude · 3–4 h)

**Goal:** the survivors-like growth loop.

- `PickupField` (like BulletField: arrays + MultiMesh) for XP shards: dropped on kill, pulled in within the pickup radius, merged above the cap. Overdrive-cleared bullets become 1-XP shards.
- XP curve from DESIGN.md §6 (reuse the function from 0.4).
- `UpgradeData` resource and five passives to start: Overclock, Hardened Rounds, Magnet Coil, Hull Plating, Graze Field. Passives add modifiers to `Stats`.
- `LevelUpSystem` and `LevelUpPanel`: pause the game, show 3 cards (Pulse Cannon levels + the five passives), apply the choice, unpause. One free reroll per run.

**Tests:** XP thresholds; card eligibility for slots and max levels; applying a passive changes Stats; reroll count. **You check:** shards fly to you; level-ups come quickly at first and slow down; picking Overclock visibly speeds up firing.

### 1.8 HUD and pause (Claude · 1–2 h)

**Goal:** the screen tells you everything at a glance.

- HUD (CanvasLayer): hull pips, XP bar with level, Overdrive meter that pulses when full, score, graze chain (only while active), timer, pause button.
- Pause panel: resume, restart, quit to menu. Pause automatically when the app loses focus.
- Respect the phone's safe area (notch / rounded corners) using the display safe-area rectangle.

**You check:** nothing important sits under your thumb in the lower middle of the screen; text is readable on a small window.

### 1.9 Neon look and juice, version 1 (Claude · 3 h)

**Goal:** it starts to look and feel like the game.

- `fx/neon_shape.gd`: draws any outline from points with a bright line and a glow. Implement both glow approaches behind a setting: real glow (WorldEnvironment, if the Compatibility renderer supports it in our Godot version — check the docs) and fake glow (additive soft halo). Use the ship, enemies and bullets.
- Background: dark animated grid shader (DESIGN.md §16 colours).
- Juice: hit flash, small screen shake on hull hit and Overdrive (scaled by a setting), death particles, graze sparks.
- Record which glow approach looks better on desktop in PROGRESS.md; the final choice is made on a phone in 2.1.

**You check:** enemy bullets are always the easiest thing to see; the screen never gets so bright you lose your ship.

### 1.10 Web build and the fun test (Both · 2 h + playtests)

**Goal:** find out if the core loop is fun before building content.

- Claude: add a "Web" export preset (turn off thread support so it runs on itch.io without extra settings) and export to `build/web/`.
- You: upload it to itch.io as a restricted (private) page and send it to at least 5 people. Watch at least two of them play if you can.
- Ask them: Did you want another run? What was confusing? Did you notice grazing?

**Go / no-go:** if at least 3 of 5 play again without being asked, continue. If not, spend a week changing the core (speed, enemy density, graze reward) before moving on. Write the result in PROGRESS.md.

**Phase 1 learning check:** you can explain the scene tree, signals, why we pool, and why the BulletField uses arrays.

---

## Phase 2 — Vertical slice: one polished sector on a phone (weeks 5–8)

**Goal:** Sector 1 is complete (5 waves and the Gatekeeper), feels good, saves your progress and runs at 60 fps on a cheap phone.

### 2.1 Android setup and first phone build (Both · 3–4 h)

- You: follow the Godot docs page "Exporting for Android" for your exact Godot version: install the JDK and Android SDK versions it lists, set their paths in Editor Settings, and install the export templates. Turn on Developer options and USB debugging on your phone.
- Claude: add an "Android" export preset (portrait, arm64 and armv7 architectures, the vibrate permission, app name and package name like `com.yourname.neongraze`) and build a debug APK with the command in CLAUDE.md.
- Test on the phone: touch feel, safe area, fps (debug overlay), both glow approaches. Record the phone model, results and glow decision in PROGRESS.md.

**Done when:** the game runs on your phone and you've chosen the glow approach.

### 2.2 Seeded randomness (Claude · 1 h)

- `core/run_rng.gd` wrapping a `RandomNumberGenerator` with a seed from `RunConfig`. Replace every random call in gameplay with it.

**Tests:** the same seed gives the same sequence; a grep for global `randf(` / `randi(` in `run/` finds nothing.

### 2.3 Waves and Sector 1 data (Claude · 3 h)

- Resources `SectorData`, `WaveData`, `SpawnEvent` (ARCHITECTURE.md §6).
- `WaveDirector`: trickle spawning, scripted events with formations (line, V, ring, sine) from screen edges, a wave banner, then the boss slot (a placeholder until 2.5).
- `data/sectors/sector_1.tres` from DESIGN.md §11. Delete the temporary spawner.
- Make the smoke test real: start a Sector 1 run with the bot pilot (ARCHITECTURE.md §9) for 60 s of game time and fail on errors, NaN positions or caps exceeded.

**Tests:** events fire at the right times; trickle rate over 10 s is within tolerance with a fixed seed; wave advances after its duration.

### 2.4 Swarm and Splitter enemies (Claude · 2 h)

- `swarm.gd` (formation movement) and `split.gd` (spawns 3 mini drifters from the pool on death). Data files from DESIGN.md §9.

**Tests:** splitter death produces exactly 3 minis; minis give the right XP.

### 2.5 Boss framework and the Gatekeeper (Claude · 4–6 h, two sessions)

- `BossData` and `BossPhase` resources; `boss.tscn`; boss HP bar; 3 s WARNING banner; small enemies retreat; phase changes at HP thresholds; defeat sequence (clear bullets, shard burst, 0.5 s slow-motion).
- `data/bosses/gatekeeper.tres` with the three phases from DESIGN.md §10.
- Debug command: jump straight to the boss.

**Tests:** phase changes at the right HP; boss defeat ends the sector. **You check:** each phase is readable and fair; there's always a gap to dodge through; grazing near the boss feels rewarding.

### 2.6 Run flow: menu, sector select, results (Claude · 3 h)

- `RunConfig`; real `ScreenRouter`; screens MainMenu, SectorSelect (Sector 1 only for now) and Results (score, time, kills, grazes, best chain, credits earned, "new best").
- Sector clear and death both lead to Results.

### 2.7 Save system and credits (Claude · 2 h)

- `Save` autoload exactly as ARCHITECTURE.md §8 (atomic write, corrupt-file recovery, version and migration hook).
- Credits formula from DESIGN.md §13; best score per sector; sectors cleared.

**Tests:** save → load round trip; corrupt file falls back to defaults and keeps a `.bak`; credits formula. **You check:** quit the game mid-menu, reopen, and your credits and best score are still there.

### 2.8 Audio and haptics (Both · 3 h)

- You: choose sound effects (Kenney CC0 packs, jsfxr) and one music track with a licence that allows commercial use. Put them in `audio/` and list each with its licence in `CREDITS.md`. Avoid "non-commercial" (NC) licences.
- Claude: `Audio` autoload with Music/SFX buses, preloaded sounds, a per-sound cooldown and slight pitch variation; wire the sounds in DESIGN.md §17 that you have so far; short haptic pulses on hull hit and Overdrive.

### 2.9 Performance pass on the phone (Both · 3 h)

- Run the stress scene and a full Sector 1 run on the phone with Godot's remote profiler.
- Target: steady 60 fps with 1,500 bullets and 150 enemies on your test phone. Claude fixes the biggest costs first and records before/after numbers in PROGRESS.md.

### 2.10 Juice, version 2 (Claude · 2 h)

- Hit-stop on big kills, Overdrive shockwave effect, card-pick animation, wave-clear flourish, a background grid that pulses with the music. Respect the screen-shake and reduced-flashing settings (stored for now, UI comes in 3.9).

### 2.11 Start recruiting testers (You · ongoing)

Open a Discord server, add a sign-up form for Android testers, and post a short gameplay clip each week. You need 15–20 people by week 17.

**Phase 2 gate:** show the slice to 5–10 people on a phone. Would they play again? Is the Gatekeeper fair? Fix the top three complaints before Phase 3.

---

## Phase 3 — MVP: content complete (weeks 9–16)

**Goal:** everything in DESIGN.md is in, monetization works, and you can build a signed release.

### 3.1 Scatter, Arc Laser and Seeker Missiles (Claude · 4–5 h)

Weapons and their five levels from DESIGN.md §7. Arc Laser uses a ray/segment check rather than projectiles. Seekers use pooled missiles with homing and a blast radius. **Tests:** level data applies correctly; laser damage ticks at 0.1 s; missile targets a new enemy if its target dies.

### 3.2 All passives, card rules and evolutions (Claude · 4 h)

All 12 passives (DESIGN.md §8), full card eligibility rules and fallbacks (§6), evolution data and the four evolved weapons (§7). **Tests:** every eligibility rule; evolution offered only when requirements are met and always included then; fallbacks when nothing is eligible.

### 3.3 Sniper, Shielded and Mine-layer (Claude · 3 h)

Behaviours and data from DESIGN.md §9. The Shielded enemy's arc blocks projectiles from the front only.

### 3.4 Sectors 2 and 3 (Both · 2 h)

Claude: `sector_2.tres` and `sector_3.tres` using the enemies and HP multipliers in DESIGN.md §11, plus their palettes. You: play each one and tune waves with the "Tune the feel" template.

### 3.5 Hydra Core and Null Sovereign bosses (Claude · 2 × 4 h)

Two boss sessions, using the framework from 2.5 and DESIGN.md §10. Hydra needs separately damageable nodes; Null Sovereign needs telegraphed laser sweeps with a safe gap.

### 3.6 Ships and ship select (Claude · 2 h)

`ShipData` for Vector, Razor and Bastion (DESIGN.md §12), ship select screen, unlock with credits.

### 3.7 Meta upgrade shop (Claude · 2–3 h)

`MetaUpgradeData` and the shop screen (DESIGN.md §13). Ranks feed into `Stats`. **Tests:** costs and ranks; can't buy without enough credits; stats include meta ranks.

### 3.8 Endless and Daily modes (Claude · 3 h)

Endless rules and Daily runs with date seed, seeded ship and mutator (DESIGN.md §2 and §14). Keep a local best per day. **Tests:** the same date always gives the same ship, mutator and spawn sequence.

### 3.9 Settings and accessibility (Claude · 3 h)

Every setting in DESIGN.md §18, saved in the profile and applied live. Colour palettes swap through one palette resource.

### 3.10 Menu polish, first-run hints, credits and privacy screen (Claude · 3 h)

Main menu with animated background; three first-run hints (drag to move, graze the bullets, double-tap for Overdrive) shown once; credits screen listing every asset licence; a privacy policy link.

### 3.11 Services layer with fakes (Claude · 2 h)

`Services` autoload with `AdService`, `StoreService`, `LeaderboardService`, `AnalyticsService` interfaces and fake implementations (ARCHITECTURE.md §7). Desktop and tests always use the fakes.

### 3.12 Rewarded placements: revive, double credits, reroll (Claude · 2 h)

Revive panel at death (once per run, 5 s countdown), double credits on Results, +1 reroll on the level-up panel. Supporter owners get rewards instantly. Built and tested with the fake services only. **Tests:** each placement can be used once per run; supporter skips the ad; game logic never waits forever if an ad fails.

### 3.13 Real ads: AdMob and consent (Both · 3–4 h)

- You: create an AdMob account, register the app and create rewarded ad units. In AdMob's privacy & messaging section, create a GDPR consent message.
- Claude: ask me before installing the Poing Studios AdMob plugin version that matches our Godot version. Implement `android_ad_service.gd`: request consent info and show the consent form when needed before any ad loads, preload rewarded ads, and add "Privacy options" to Settings. Use Google's test ad unit IDs during development; real IDs go in only for release.
- Test on your phone. Never tap your own live ads.

### 3.14 Supporter Pack with Google Play Billing (Both · 3–4 h)

- You: in Play Console, create the app, upload a build to the internal testing track (billing only works for apps in Play Console), create the one-time product `supporter_pack` at $3.99, and add yourself as a licence tester.
- Claude: ask me before installing the official Godot Google Play Billing plugin for our Godot version. Implement `android_store_service.gd`: query, purchase, acknowledge every purchase (Google refunds purchases that aren't acknowledged within 3 days), restore on start and from Settings.
- Test buy, restore after reinstall, and cancel.

### 3.15 Balance simulation and tuning (Both · 3 h)

- Claude: a headless balance script that runs 50 bot runs per sector at fixed fps and reports average level at the boss, time to death, boss fight length and how often the bot clears the sector.
- You: compare with the targets (DESIGN.md §6 and §10) and tune data with Claude. Remember the bot is worse than a human: it's for spotting big outliers, not final tuning.

### 3.16 Release build setup (Both · 2 h)

- You: create an upload keystore (`keytool -genkeypair -v -keystore neon-graze-upload.keystore -alias upload -keyalg RSA -keysize 2048 -validity 10000`). Store it and its passwords somewhere safe outside the project and back it up.
- Claude: release export preset producing an AAB, version name/code scheme, target SDK 36 (check the preset and Gradle template), and a script that lists every native `.so` in the AAB and checks 16 KB alignment (each plugin must be compliant too). Never commit the keystore or passwords.

### 3.17 Optional: daily leaderboard and analytics (Both · 3–4 h)

Only if a maintained Play Games Services plugin and analytics plugin exist for our Godot version and pass the 16 KB check. Otherwise skip: ship with local daily bests and Play Console statistics, and add these in an update. Any analytics must be declared in the Data safety form.

### 3.18 Bug bash and content freeze (Both · 1 week)

Play every mode on every ship. Claude runs `/code-review` and the "Keep the code healthy" template. After this task, no new features until launch; new ideas go to "Open issues / ideas for later" in PROGRESS.md.

**Phase 3 learning check:** you can explain Resources and data-driven design, how the save file works, and why services have fake implementations.

---

## Phase 4 — Closed testing (weeks 17–20)

### 4.1 Play Console app setup and policy forms (You · 2–3 h)

Store listing draft, content rating questionnaire, target audience 13+ (not under 13, to stay out of the Families programme), Data safety form (AdMob, purchases, any analytics), "Contains ads" declaration, privacy policy URL, Play App Signing on. If you monetize in the EU, complete the trader status declaration (your contact details become public, so consider a business address). AdMob will ask for an `app-ads.txt` file on your developer website.

### 4.2 Privacy policy and credits page (Both · 1 h)

Claude drafts a plain-English privacy policy covering local saves, AdMob (and consent), Google Play Billing and any analytics. You check it's accurate and host it (GitHub Pages or a simple website). Claude isn't a lawyer; if you're unsure, get it reviewed.

### 4.3 Store assets: icon, screenshots, video (Both · 3 h)

- Claude: a debug "screenshot mode" (hide debug overlay, optional HUD toggle, save the screen as a PNG), Android launcher icons from one neon ship design, a 512×512 store icon and a 1024×500 feature graphic.
- You: 6–8 portrait screenshots with short captions ("One thumb. Endless swarms." "No forced ads." "Daily challenge."), and a 15–30 s gameplay video showing a graze streak and an Overdrive in the first seconds.

### 4.4 Closed test: 12+ testers for 14 days (You)

Create a closed testing track with a Google Group of testers, upload the AAB and share the opt-in link. Keep at least 12 testers opted in for 14 days in a row (aim for 15–20). Message them on day 1 and day 7.

### 4.5 Crash and feedback loop (Both · ongoing)

Check Android vitals and the pre-launch report. For bugs, use the "Report a bug" template with `adb logcat` output from the phone. Ship fixes to the closed track.

### 4.6 Apply for production access (You)

After 14 days, answer Google's questions about your test honestly, using the real feedback you gathered.

---

## Phase 5 — Launch (weeks 21–22)

### 5.1 Release candidate (Both)

Bump the version, switch to real ad unit IDs, build the release AAB, run all tests, check 16 KB alignment, test the exact AAB on your phone through internal testing.

### 5.2 Staged rollout and launch posts (You)

Roll out to 20% of users, watch crashes and reviews for 2–3 days, then 100%. Post your launch clip on Reddit (follow each subreddit's rules), TikTok, YouTube Shorts and your Discord. Claude can draft the posts.

---

## Phase 6 — Live updates (week 23 onward)

### 6.1 First update (Both)

Plan updates every 2–4 weeks. Start each one with:

```
/plan Read docs/PROGRESS.md "Open issues / ideas" and the latest reviews I'll paste below. Propose a small update (1–2 weeks of my time) with tasks written in the same format as docs/PLAN.md, and add them as Phase 6 tasks.
```

Good first updates: the online daily leaderboard, a fourth sector, cosmetic skin packs, weekly mutators.
