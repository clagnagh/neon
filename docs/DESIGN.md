# Neon Graze — Game Design

What the game is, with every rule and number. Tasks in `docs/PLAN.md` point at sections here (for example "DESIGN.md §9"). Code reads these numbers from data files (`data/*.tres`); when tuning changes a number, update this file too (the "Tune the feel" template does that).

> **Status:** first draft. Every number is a starting guess to be tuned by playing (task 1.10 onward) and by the balance tool (task 3.15). Distances are in pixels at the 720 × 1280 design resolution; times are in seconds.

**Contents:** §1 Pitch · §2 Modes · §3 Controls and core loop · §4 Player · §5 Graze and Overdrive · §6 XP, levels and cards · §7 Weapons · §8 Passives and stats · §9 Enemies and bullet patterns · §10 Bosses · §11 Sectors and waves · §12 Ships · §13 Score, credits and meta upgrades · §14 Daily runs and mutators · §15 Monetization and privacy · §16 Look and colours · §17 Audio and haptics · §18 Settings and accessibility

---

## 1. Pitch

**Neon Graze** is a one-thumb, portrait, neon shoot-'em-up with survivors-like growth. Drag anywhere to fly, your weapons fire on their own, and every bullet you *almost* touch charges an Overdrive that wipes the screen. A run lasts about 8 minutes: five waves and a boss.

**Pillars**

1. **One thumb.** Everything is playable with one thumb on a phone held in one hand. No virtual joystick and no buttons during play, except pause.
2. **Risk is reward.** Grazing bullets is the fastest way to power. Playing safe is valid; playing close is better.
3. **Readable chaos.** Hundreds of bullets, but the player can always see the threats, their own hitbox and a way out.
4. **Short and replayable.** 8-minute runs, quick restarts, a new build each time, and permanent progress through credits.
5. **Fair to players' wallets.** No forced ads, no energy, no paid currency. Ads are opt-in rewards; a single Supporter Pack removes them.

**Audience:** 13+, mobile players who like Vampire Survivors-style growth and arcade shooters. Android first, plus a web build for playtests.

---

## 2. Modes

| Mode | Unlock | Summary |
|---|---|---|
| **Sector run** | Sector 1 from the start; each later sector unlocks by clearing the one before | 5 waves (~7:15) then the sector boss. Clearing it is a win. This is the main mode. |
| **Endless** | Clear Sector 1 | Survive as long as you can. Score includes time. |
| **Daily** | Clear Sector 1 | One seeded run per day, the same for everyone: fixed sector, ship and mutator (§14). |

**Endless rules**

- Starts with the Sector 1 enemy mix, adds the Sector 2 enemies at 5:00 and the Sector 3 enemies at 10:00.
- Spawn rates grow 6% per minute (linear: `1 + 0.06 × minutes`). Enemy HP multiplier is `1 + 0.25 × minutes`.
- A boss every 5 minutes, cycling Gatekeeper → Hydra Core → Null Sovereign. Boss HP × `(1 + 0.5 × cycle)`, where cycle counts from 0.
- Score gets +10 per second survived. The run ends only on death (one revive allowed, §15).

---

## 3. Controls and core loop

**Controls**

- **Drag anywhere** to move. The ship copies your finger's *movement*, not its position (relative drag, §4), so your thumb never covers the ship.
- **Weapons fire automatically** at the nearest enemy.
- **Double-tap anywhere** when the Overdrive meter is full to fire Overdrive (§5).
- **Pause** with the button in the top corner, Android back, or by leaving the app (auto-pause).
- On the level-up panel, **tap a card** to pick it.

**Core loop (one run):** fly and dodge → enemies die and drop XP shards → collect shards → level up and pick 1 of 3 cards → build gets stronger → waves get harder → graze bullets to charge Overdrive → clear the screen when it gets too dense → beat the boss → Results → spend credits → next run.

**HUD layout (portrait).** Everything important sits at the top, away from the thumb.

- Top-left: hull pips, with the Overdrive meter underneath (glows and pulses when full).
- Top-centre: score; the graze chain "×N GRAZE" shows under it while the chain is 2 or more.
- Top-right: timer and pause button (side can be swapped in Settings).
- A full-width XP bar under the top bar, with the level number.
- During a boss: the boss HP bar replaces the wave banner area under the XP bar.
- Wave banners appear in the upper third for 2 s.
- Nothing permanent in the lower 40% of the screen.

**First-run hints** (each shown once, remembered in the save):

1. Run start: "DRAG ANYWHERE TO MOVE". Hides after 200 px of total drag.
2. The first time 10+ enemy bullets are on screen: "BRUSH PAST BULLETS TO GRAZE". Hides after the first graze or 6 s.
3. The first time the meter is full: "DOUBLE-TAP FOR OVERDRIVE". Hides when Overdrive fires.

**Death:** at 0 hull, "SIGNAL LOST" shows time survived, kills and level. The revive offer appears (§15) if available, then Results. Tapping Retry from Results starts a new run with the same setup instantly.

---

## 4. Player

These go in `data/tuning.tres` (`TuningData`). Ship-specific values (§12) change some of them through stats.

| Value | Field | Start value | Notes |
|---|---|---|---|
| Drag sensitivity | `drag_sensitivity` | 1.0 | Finger movement × this. Multiplied by the Settings slider (0.6–1.6). |
| Move smoothing | `move_smoothing` | 28 /s | Exponential follow toward the target; higher = snappier. |
| Max move speed | `max_move_speed` | 2,600 px/s | Stops teleport-like jumps on big flicks. |
| Arena margin | `arena_margin` | 24 px | The ship's centre stays this far inside the arena edges. |
| Start position | `start_position_ratio` | (0.5, 0.78) | Fraction of arena width/height. |
| Hitbox radius | `hitbox_radius` | 5 px | Base value of the `hitbox_radius` stat. The ship drawing is ~40 px tall; the hitbox is the bright dot at its centre. |
| Graze radius | `graze_radius` | 30 px | Base value of the `graze_radius` stat. Extra distance beyond touching (§5). |
| Starting hull | `base_hull` | 3 | Ship overrides this. |
| Max hull cap | `max_hull_cap` | 6 | No source can raise max hull above this. |
| Invulnerability after a hit | `invuln_after_hit` | 1.0 s | Base value of the `invuln_time` stat. |
| Blink rate while invulnerable | `blink_hz` | 12 Hz | |
| Pickup radius | `pickup_radius` | 90 px | Base value of the `pickup_radius` stat. |
| Shard pull speed | `shard_pull_speed` | 900 px/s | Shards accelerate to this once inside the pickup radius. |
| Tap: max duration | `tap_max_duration` | 0.20 s | |
| Tap: max movement | `tap_max_move` | 16 px | |
| Double-tap window | `double_tap_window` | 0.30 s | From the end of the first tap to the start of the second. |

**Damage:** any enemy contact or enemy bullet costs **1 hull**, then `invuln_time` seconds of invulnerability with blinking. Hits during invulnerability are ignored. Hull never goes below 0. A hit also breaks the graze chain and gives a small screen shake and haptic pulse.

---

## 5. Graze and Overdrive

### Graze

A **graze** is when an enemy bullet passes close to the hitbox without hitting it.

- The **graze ring** for a bullet is `bullet radius + graze_radius` around the hitbox centre. The hit ring is `bullet radius + hitbox_radius`.
- When a bullet enters the graze ring it's marked *inside*. The graze is awarded **when that bullet leaves the ring** without having hit. Deciding on exit is what guarantees that a bullet which hits never also grazes.
- **Once per bullet.** A bullet can't graze twice, even if it re-enters the ring.
- **No grazing while invulnerable** (after a hit, during Overdrive's invulnerability, after a revive). A bullet that leaves the ring during invulnerability gives nothing.
- A bullet removed while inside the ring (Overdrive, culling, missile blast) gives nothing.

Each graze gives:

| Reward | Amount |
|---|---|
| Graze chain | +1 |
| Score | `10 + min(chain, 90)` (10 → 100 points per graze as the chain grows) |
| Overdrive meter | +2 × `overdrive_gain` |
| Feedback | A spark on the ring edge where the bullet left, and a soft tick whose pitch rises with the chain (§17) |

**Graze chain:** grows by 1 per graze and resets 1.5 s (`graze_chain_timeout`) after the last graze, or immediately on a hull hit. The run records the best chain.

### Overdrive

- **Meter:** 0–100, starts at 0 (plus the Overdrive Primer meta upgrade, §13), never decays. Gains: +2 per graze and +1 per kill, both × `overdrive_gain`, capped at 100.
- **When full:** the HUD meter glows and pulses, a ready sound plays, and `Events.overdrive_ready` fires once.
- **Firing:** double-tap anywhere (§4). With a full meter:
  1. Every enemy bullet is cleared and becomes a **1-XP shard** where it was (merging above the shard cap, so no XP is lost).
  2. Every enemy whose centre is inside the arena takes `150 × overdrive_damage` damage. Bosses and boss parts instead take 3% of their max HP.
  3. The player gets 0.5 s of invulnerability.
  4. A shockwave ring expands from the ship, with screen shake, a haptic pulse and a sound.
  5. The meter resets to 0.
- A double-tap with a meter that isn't full does nothing except a small "not ready" flicker on the meter.

---

## 6. XP, levels and cards

### XP curve

XP needed to go **from level L to L+1**:

```
xp_to_next(L) = round(4 + 2 × L^1.64)
```

The constants (4, 2, 1.64) live in `tuning.tres`. The pure function is in `core/formulas.gd` (tested in task 0.4: levels 1, 5 and 10 give **6, 32 and 91**).

| Level | XP to next | Total XP to reach next | | Level | XP to next | Total XP to reach next |
|---|---|---|---|---|---|---|
| 1 | 6 | 6 | | 11 | 106 | 521 |
| 2 | 10 | 16 | | 12 | 122 | 643 |
| 3 | 16 | 32 | | 13 | 138 | 781 |
| 4 | 23 | 55 | | 14 | 156 | 937 |
| 5 | 32 | 87 | | 15 | 174 | 1,111 |
| 6 | 42 | 129 | | 16 | 193 | 1,304 |
| 7 | 53 | 182 | | 17 | 212 | 1,516 |
| 8 | 65 | 247 | | 18 | 233 | 1,749 |
| 9 | 77 | 324 | | 19 | 254 | 2,003 |
| 10 | 91 | 415 | | 20 | 276 | 2,279 |

**XP sources:** shards dropped by kills (value = the enemy's `xp`), 1-XP shards from Overdrive-cleared bullets, and boss defeat bursts. All XP is × `xp_gain`.

**Pacing targets** (checked by the balance tool, task 3.15):

| Target | Value |
|---|---|
| First level-up | within 10 s |
| Level-ups in the first 2 minutes | every 10–25 s |
| Level-ups near the end of wave 5 | every 40–60 s |
| Level when the Sector 1 boss appears | 13–15 |
| Level when the Sector 2 boss appears | 15–17 |
| Level when the Sector 3 boss appears | 17–19 |

### Slots

- **3 weapon slots** (the ship's starting weapon fills one).
- **5 passive slots.**

### Card rules

Each level-up shows a **hand of 3 different cards**. A card is **eligible** if it is one of these:

1. **Weapon level-up:** an owned weapon below its max level (5).
2. **New weapon:** an unowned base weapon, and a weapon slot is free.
3. **Passive level-up:** an owned passive below its max level.
4. **New passive:** an unowned passive, and a passive slot is free.
5. **Evolution:** an owned weapon at level 5 whose required passive (§7) is owned at any level, and that hasn't evolved yet.

Never eligible: evolved weapons (they have no levels), anything the current mutator bans.

**Drawing a hand**

- If any evolution is eligible, one evolution card is always in the hand (if several, the RNG picks one). The rest of the hand is drawn normally.
- Normal cards are drawn without replacement, weighted by their data `weight` (default 10). Owned items get ×1.5 weight, to help builds come together.
- **Fallbacks:** if fewer than 3 cards are eligible, the hand is filled with fallback cards, which appear only when needed:
  - **Repair:** +1 hull (or +500 score if hull is full).
  - **Data Cache:** +300 score and +20 Overdrive meter.
  - With no eligible cards at all, the hand is just Repair + Data Cache.

**Rerolls:** each run starts with `rerolls` = 1 (+ meta upgrades). A reroll draws a fresh hand (cards can repeat). With no rerolls left, the panel offers **+1 reroll** for a rewarded ad once per run (§15).

**Flow:** reaching the XP threshold pauses the game and opens the panel. Picking a card applies it and unpauses. Several level-ups at once open panels one after another. Leftover XP carries over.

---

## 7. Weapons

Four base weapons with 5 levels each (level 1 is getting it) and one evolution each. Level stats below are *before* the player's stats are applied:

- damage × `damage`
- cooldown ÷ `fire_rate`
- count + `projectile_count` (Scatter gets +2 pellets per point, Arc Laser +1 chain)
- speed × `projectile_speed`
- pierce + `pierce`
- radius / blast / beam width × `area`

All weapons auto-aim at the nearest enemy (bosses and boss parts count).

### Pulse Cannon

Fast single bolts. Vector's starting weapon. With no enemy on screen it fires straight up.

| Level | Damage | Cooldown | Bolts | Other |
|---|---|---|---|---|
| 1 | 10 | 0.30 | 1 | Speed 1,100, radius 6, pierce 0, lifetime 1.2 s |
| 2 | 13 | 0.30 | 1 | |
| 3 | 13 | 0.30 | 2 | Parallel, 14 px apart |
| 4 | 15 | 0.24 | 2 | |
| 5 | 18 | 0.24 | 3 | 8° spread, pierce 1 |

### Scatter

A short-range shotgun fan. Razor's starting weapon. Rewards flying close.

| Level | Damage per pellet | Cooldown | Pellets | Other |
|---|---|---|---|---|
| 1 | 6 | 0.90 | 5 in 40° | Speed 900, radius 5, lifetime 0.45 s (~400 px range) |
| 2 | 6 | 0.90 | 7 in 46° | |
| 3 | 8 | 0.90 | 7 | |
| 4 | 8 | 0.70 | 7 | |
| 5 | 9 | 0.70 | 9 in 54° | Pierce 1 |

### Arc Laser

A beam from the ship toward the nearest enemy that damages everything along it. Not projectiles: a segment-vs-circle check every **0.1 s** tick. The beam turns toward its target at up to 120°/s. "Cooldown" is the off-time between bursts.

| Level | Damage per tick | On time | Cooldown | Beam | Other |
|---|---|---|---|---|---|
| 1 | 4 | 1.2 | 2.0 | Width 12, length 700 | |
| 2 | 5 | 1.2 | 2.0 | | |
| 3 | 5 | 1.2 | 2.0 | Width 20, length 900 | |
| 4 | 6 | 1.6 | 2.0 | | |
| 5 | 7 | 1.6 | 2.0 | | Chains from the beam's end to 2 more enemies within 180 px |

### Seeker Missiles

Homing missiles that explode. Bastion's starting weapon. Blasts damage everything in the radius and ignore shields.

| Level | Damage | Cooldown | Missiles | Other |
|---|---|---|---|---|
| 1 | 22 | 1.6 | 2 | Speed 520, turn rate 240°/s, blast radius 50, lifetime 3 s (explodes at the end) |
| 2 | 22 | 1.6 | 3 | |
| 3 | 22 | 1.6 | 3 | Blast radius 70 |
| 4 | 30 | 1.6 | 3 | |
| 5 | 30 | 1.3 | 4 | |

If a missile's target dies, it retargets to the nearest living enemy. With no enemy, it flies straight and explodes at the end of its lifetime.

### Evolutions

An evolution replaces the weapon (same slot) with a stronger single-level weapon. Requirement: the weapon at level 5 plus the paired passive at any level (§6).

| Evolution | Needs | Result |
|---|---|---|
| **Photon Storm** | Pulse Cannon 5 + Overclock | 5 bolts in a 16° spread, damage 20, cooldown 0.18, pierce 2 |
| **Flak Bloom** | Scatter 5 + Hardened Rounds | 11 pellets in 60°, damage 10, cooldown 0.6, pierce 1. Each pellet bursts into 3 fragments (damage 4, radius 4) on its first hit |
| **Storm Arc** | Arc Laser 5 + Capacitor | Always on (no cooldown), damage 8 per tick, width 24, chains to 4 enemies |
| **Hellfire Swarm** | Seeker Missiles 5 + Payload | 6 missiles, cooldown 1.1, damage 34, blast radius 90. Blasts also erase enemy bullets inside them (no shards from these) |

---

## 8. Passives and stats

### Stats

Every number that upgrades can change is a stat (`core/stats.gd`): `value = (base + Σ ADD) × (1 + Σ MULT)`. Percent bonuses add together.

| Stat | Base | Meaning |
|---|---|---|
| `damage` | 1.0 | Multiplies all player damage |
| `fire_rate` | 1.0 | Divides weapon cooldowns |
| `projectile_speed` | 1.0 | Multiplies projectile and missile speed |
| `projectile_count` | 0 | Extra projectiles (§7) |
| `pierce` | 0 | Extra enemies a projectile passes through |
| `area` | 1.0 | Multiplies projectile radius, blast radius and beam width |
| `pickup_radius` | 90 | px |
| `graze_radius` | 30 | px |
| `hitbox_radius` | 5 | px |
| `max_hull` | 3 | Capped at 6 |
| `invuln_time` | 1.0 | Seconds after a hit |
| `xp_gain` | 1.0 | Multiplies XP collected |
| `overdrive_gain` | 1.0 | Multiplies meter gains |
| `overdrive_damage` | 1.0 | Multiplies Overdrive damage to normal enemies |
| `overdrive_start` | 0 | Meter at run start |
| `rerolls` | 1 | Rerolls per run |
| `credit_gain` | 1.0 | Multiplies credits earned |

### Passives

Twelve passives. The first five (bold) arrive in task 1.7; the rest in task 3.2. Taking Hull Plating also repairs 1 hull.

| Id | Name | Max level | Per level | Pairs with |
|---|---|---|---|---|
| `overclock` | **Overclock** | 5 | `fire_rate` +10% | Pulse Cannon → Photon Storm |
| `hardened_rounds` | **Hardened Rounds** | 5 | `damage` +12% | Scatter → Flak Bloom |
| `magnet_coil` | **Magnet Coil** | 5 | `pickup_radius` +25% | |
| `hull_plating` | **Hull Plating** | 3 | `max_hull` +1 | |
| `graze_field` | **Graze Field** | 3 | `graze_radius` +15% | |
| `capacitor` | Capacitor | 5 | `overdrive_gain` +12% | Arc Laser → Storm Arc |
| `payload` | Payload | 5 | `area` +10% | Seeker Missiles → Hellfire Swarm |
| `velocity_coils` | Velocity Coils | 3 | `projectile_speed` +15% | |
| `phase_tips` | Phase Tips | 3 | `pierce` +1 | |
| `multiplexer` | Multiplexer | 2 | `projectile_count` +1 | |
| `salvage` | Salvage | 5 | `xp_gain` +8% | |
| `afterimage` | Afterimage | 3 | `invuln_time` +0.25 s | |

Card text shows the change, e.g. "Overclock 2 → 3: fire rate +10% (total +30%)".

---

## 9. Enemies and bullet patterns

Enemy HP in play = data HP × the wave's `hp_mult` (§11) × the sector's `hp_mult` × any mutator. Contact with any enemy costs 1 hull (§4). Enemies that leave the arena by more than 96 px vanish without reward.

| Enemy | HP | Speed | Radius | XP | Score | Colour | Shape | First appears |
|---|---|---|---|---|---|---|---|---|
| Drifter | 20 | 90 | 18 | 1 | 10 | Magenta | Triangle pointing down | S1 W1 |
| Charger | 35 | 120 (dash 900) | 20 | 3 | 30 | Orange | Diamond | S1 W1 |
| Spinner Turret | 120 | 160 (entry) | 26 | 8 | 100 | Yellow | Hexagon with spokes | S1 W3 |
| Swarm | 8 | 220 | 12 | 1 | 5 | Violet | Chevron | S1 W2 |
| Splitter | 50 | 70 | 24 | 3 | 40 | Pink | Pentagon | S1 W4 |
| Mini Drifter | 8 | 150 | 10 | 1 | 5 | Pink | Small triangle | From Splitters |
| Sniper | 40 | 200 | 18 | 5 | 60 | Red | Long thin triangle | S2 |
| Shielded | 80 | 60 | 24 | 6 | 80 | Blue + white shield arc | Square | S2 |
| Mine-layer | 60 | 140 | 22 | 6 | 70 | Amber | Oval | S3 |
| Mine | 10 | 0 | 12 | 0 | 5 | Amber, blinking core | Circle | From Mine-layers |

### Behaviours

- **Drifter** (`drift.gd`): moves down at its speed with a sine sway (amplitude 40 px, 0.5 Hz, random phase). Doesn't shoot.
- **Charger** (`charge.gd`), a state machine:
  1. *Approach:* moves toward the player until within 380 px, or for at most 2.5 s.
  2. *Telegraph (0.6 s):* stops, flashes, and shows a thin line to the **locked position** (the player's position when the telegraph starts).
  3. *Dash:* 900 px/s to the locked position, then 150 px past it.
  4. *Recover (1.2 s):* slow drift.
  5. Back to approach.
- **Spinner Turret** (`turret.gd`): enters from the top and stops at a random height 14–28% down the arena. Then it repeats: fire a spiral (`spiral_step`) with 4 arms, one bullet per arm every 0.12 s, rotating 90°/s, bullet speed 200, small bullets, for 4.0 s; pause 1.5 s. After 20 s it leaves upward at 120 px/s.
- **Swarm** (`swarm.gd`): spawned only in formations of 6–10 and follows the formation's path (§11) at 220 px/s. Doesn't shoot.
- **Splitter** (`split.gd`): drifts like a Drifter at 70 px/s. On death it spawns **exactly 3 Mini Drifters** from the pool in a 120° fan pointing down, which then drift down at 150 px/s. Minis give 1 XP each.
- **Sniper** (`sniper.gd`): enters from the left or right edge and stops 10% in from that edge, 15–45% down. It aims a red telegraph line at the player for 1.0 s (tracking for 0.7 s, then locked), then fires an `aimed_burst` of 3 medium bullets 0.08 s apart at speed 520. It moves to a new height and repeats every 3.5 s, and leaves after 3 bursts.
- **Shielded** (`shield.gd`): descends at 60 px/s to 35% of the arena height and holds there, drifting toward the player's x. It turns to face the player at 60°/s. Its **120° front arc blocks player projectiles and the Arc Laser** (projectile consumed with a spark, no damage). Missile blasts and Overdrive ignore the shield. Every 2.5 s it fires a `fan` of 3 small bullets (30° spread) at speed 180.
- **Mine-layer** (`mine_layer.gd`): crosses the screen horizontally at 140 px/s, at 18–50% height, entering from one side and leaving by the other. It drops a Mine every 0.9 s.
- **Mine** (`mine.gd`): stationary, fuse 3.0 s (blinking faster in the last second), then bursts into a `ring` of 8 medium bullets at speed 160. Shooting it first stops the burst.

### Bullets

| Size | Radius | Used by |
|---|---|---|
| Small | 6 | Turrets, Shielded, Gatekeeper spirals |
| Medium | 8 | Snipers, Mines, boss rings |
| Large | 12 | Boss accents only |

Bullet speed in play = data speed × the sector's `bullet_speed_mult` × any mutator.

### Patterns (`bullet_patterns.gd`, pure functions returning unit directions)

| Function | Arguments | Result |
|---|---|---|
| `ring(count, offset_angle)` | | `count` directions evenly spaced around a circle, starting at `offset_angle` |
| `fan(count, spread_deg, aim_angle)` | | `count` directions spread evenly across `spread_deg`, centred on `aim_angle` (one bullet = straight at the aim) |
| `aimed_burst(count, aim_angle, jitter_deg)` | | `count` directions at the aim angle (± jitter, 0 by default). The caller spaces them in time. |
| `spiral_step(arms, base_angle)` | | `arms` evenly spaced directions starting at `base_angle`. The caller advances `base_angle` each step. |

### Fairness rules

- No enemy bullet ever spawns within 120 px of the player.
- Rings have at most 24 bullets, so gaps stay at least ~45 px wide 200 px from the source.
- Aimed attacks are either telegraphed (Charger, Sniper, lasers) or slow enough to take at least 0.35 s to reach the player.
- Enemies entering from the sides show an edge marker 0.5 s before they appear.

---

## 10. Bosses

### Shared rules

- **Arrival:** when the last wave ends, a "WARNING" banner shows for 3 s, small enemies retreat off-screen (no XP, no score), and every shard is pulled to the player.
- **Entry:** the boss enters from the top over 2 s; it can't be damaged during entry.
- **Phases** change at HP thresholds. On each change: a 1 s firing pause, a flash, and every enemy bullet becomes a shard.
- Boss HP is fixed in its data. The sector's `hp_mult` doesn't apply, but Endless scaling does.
- Bosses take 3% of max HP from Overdrive (§5). Touching the boss costs 1 hull.
- **Defeat:** all bullets cleared (→ shards), a shard burst (the boss's XP), 0.5 s of slow motion at 0.25×, then "SECTOR CLEAR" and Results after 2 s.
- **Target fight length:** 60–90 s for a typical build at the target level (§6). HP values below are first guesses for the balance tool to tune.
- **Always a way out:** every pattern leaves a gap at least 44 px wide (lasers: 120 px) the player can reach in time.

### Gatekeeper (Sector 1)

HP **24,000** · radius 70 · octagon with a rotating outer ring · magenta/white · XP burst 120 · score 5,000. Moves slowly side to side between 30% and 70% of the arena width, at 20% height.

| Phase | HP | Attacks |
|---|---|---|
| 1 | 100–66% | Ring of 16 medium bullets at speed 170 every 1.4 s, each ring rotated 11.25°. Every third ring is replaced by an aimed burst of 5 small bullets (speed 300, 0.08 s apart). |
| 2 | 66–33% | Two counter-rotating spirals (3 arms each, step 0.10 s, ±70°/s, speed 190) for 5 s, then 2 s of rest with one aimed fan of 7 (50°, speed 220). Summons 2 Drifters from the top every 6 s. |
| 3 | 33–0% | "The Gate": a ring of 24 medium bullets at speed 200 every 1.1 s, with a 4-bullet gap aimed at the player. Every 8 s: telegraph 0.8 s, charge down to 55% height at 700 px/s, return at 300 px/s. No firing while charging. |

### Hydra Core (Sector 2)

Total HP **40,000** = three heads × 8,000 (60%) + core 16,000 (40%) · core radius 60, heads radius 34 at 90 px from the core (left, right, below) · teal/orange · XP burst 160 · score 10,000. **The heads are separately damageable nodes** (boss parts). The core is shielded until every head is destroyed.

| Phase | HP | Attacks |
|---|---|---|
| 1 | 100–40% (heads alive) | Heads take turns (one every 0.7 s) firing an aimed fan of 5 (40°, speed 210). Every 4 s all heads fire a ring of 10 small bullets. Each destroyed head bursts 30 XP and makes the others fire 20% faster. |
| 2 | 40–15% (core exposed) | Core moves in a slow figure-eight. Spiral of 5 arms (step 0.09 s, 60°/s, speed 180), plus an aimed burst of 5 every 2.5 s. |
| 3 | 15–0% | Alternating rings of 20 at speeds 160 and 220 (offset half a gap) every 0.8 s. Summons a V of 4 Swarm every 6 s. |

### Null Sovereign (Sector 3)

HP **60,000** · radius 80 · large hollow diamond · violet/white · XP burst 220 · score 20,000.

**Laser rules:**

- Every laser shows a thin telegraph line for 1.0 s before it fires.
- Beams are 26 px wide and sweep at most 40°/s.
- Collision is segment vs. the hitbox.
- There is always a safe lane at least 120 px wide that the player can reach within 0.8 s.

| Phase | HP | Attacks |
|---|---|---|
| 1 | 100–60% | One laser sweeps 70° across the lower screen over 1.8 s, alternating direction, every 3.5 s. Between sweeps: a ring of 14 medium bullets at speed 160. |
| 2 | 60–30% | Two lasers in a V with a 40° safe lane between them rotate together through 50° over 2.0 s, every 4 s. Plus a spiral of 4 arms (step 0.12 s, speed 170). |
| 3 | 30–0% | "Collapse": four lasers in an X rotate slowly (20°/s) around the boss; the safe lanes are between them. Every 1.5 s, aimed fans of 3 (speed 200) from both side edges, with edge markers. |

---

## 11. Sectors and waves

| # | Name | `hp_mult` | `bullet_speed_mult` | Enemies | Boss | Palette | Unlock |
|---|---|---|---|---|---|---|---|
| 1 | Outer Grid | 1.0 | 1.0 | Drifter, Charger, Spinner Turret, Swarm, Splitter | Gatekeeper | Cyan / magenta | Start |
| 2 | Signal Reef | 1.6 | 1.1 | + Sniper, Shielded | Hydra Core | Teal / orange | Clear Sector 1 |
| 3 | Null Core | 2.4 | 1.2 | + Mine-layer (all enemies) | Null Sovereign | Violet / red | Clear Sector 2 |

**Every sector:** 5 waves, then the boss. Wave `hp_mult`: **1.0, 1.3, 1.7, 2.2, 2.8** (waves 1–5). Wave durations: 80, 85, 90, 90, 90 s (7:15 in total). A banner shows each wave's name for 2 s at its start.

### Spawning

- **Trickle:** an enemy type spawns at `per_second` on average. The time between spawns is randomised ±40% with the run's RNG, so the average holds over 10 s.
- **Edges:** `TOP` (just above the arena at a random x from 10–90%), `LEFT` / `RIGHT` (just outside, at a random y from 5–40%), `SIDES` (alternating left/right).
- **Formations** for scripted events:
  - `SINGLE`.
  - `LINE`: `count` enemies side by side, `spacing` apart.
  - `V`: a chevron pointing down.
  - `RING`: `count` enemies on a 120 px circle above the arena that move down together.
  - `SINE`: a column entering one by one 0.25 s apart, each following a sine path (amplitude 120 px, 0.4 Hz).

### Sector 1 — Outer Grid

| Wave | Duration | hp_mult | Trickle (per second) | Scripted events (time into wave) |
|---|---|---|---|---|
| 1 "OUTER GRID" | 80 | 1.0 | Drifter 1.0 | 0:20 LINE of 5 Drifters · 0:45 V of 7 Drifters · 1:05 2 Chargers from SIDES |
| 2 "CONTACT" | 85 | 1.3 | Drifter 1.1, Charger 0.2 | 0:15 SINE of 8 Swarm · 0:40 LINE of 3 Chargers · 1:00 V of 8 Swarm |
| 3 "CROSSFIRE" | 90 | 1.7 | Drifter 1.0, Charger 0.15 | Spinner Turret at 0:05, 0:35, 1:05 · SINE of 8 Swarm at 0:20, 0:50, 1:20 |
| 4 "FRACTURE" | 90 | 2.2 | Drifter 1.3, Splitter 0.25, Charger 0.15 | 0:05 2 Spinner Turrets · 0:30 RING of 12 Drifters · 0:50 V of 10 Swarm · 1:10 2 Spinner Turrets |
| 5 "OVERLOAD" | 90 | 2.8 | Drifter 1.5, Splitter 0.3, Charger 0.25 | Spinner Turret at 0:05, 0:23, 0:41, 0:59 · Swarm every 12 s from 0:10 (alternating SINE 8 / V 8) · 1:00 "RUSH": 3 V of 7 Drifters 1 s apart |

Then the **Gatekeeper**.

### Sector 2 — Signal Reef (outline, finished in task 3.4)

| Wave | Mix |
|---|---|
| 1 | Drifters and Chargers; Swarm formations |
| 2 | + Snipers (one every ~8 s) |
| 3 | + Shielded (one every ~10 s), Spinner Turret events |
| 4 | Splitters, Snipers and Shielded together |
| 5 | Everything, denser; a Sniper pair event every 20 s |

Then the **Hydra Core**.

### Sector 3 — Null Core (outline, finished in task 3.4)

| Wave | Mix |
|---|---|
| 1 | Drifters, Swarm, Mine-layers |
| 2 | + Chargers and Snipers |
| 3 | + Shielded and Spinner Turrets |
| 4 | + Splitters, Mine-layer pairs |
| 5 | Everything, densest; RING events with Shielded at the centre |

Then the **Null Sovereign**.

---

## 12. Ships

| Ship | Unlock | Start weapon | Hull | Stat changes | Style |
|---|---|---|---|---|---|
| **Vector** | Free | Pulse Cannon | 3 | None | All-rounder. Cyan arrow. |
| **Razor** | 1,500 credits | Scatter | 2 | `hitbox_radius` 4, `graze_radius` +20%, `overdrive_gain` +25% | Graze specialist: fragile and close-range. Thin magenta-white dart. |
| **Bastion** | 2,500 credits | Seeker Missiles | 5 | `fire_rate` −10%, `pickup_radius` +20%, `overdrive_damage` +50% | Tank: forgiving, slower to kill. Wide green-cyan wedge. |

The selected ship is remembered. Locked ships show their cost and a short preview of their perks.

---

## 13. Score, credits and meta upgrades

### Score

| Source | Points |
|---|---|
| Kill | The enemy's `score` |
| Graze | `10 + min(chain, 90)` |
| Boss defeat | The boss's `score` |
| Sector clear | 1,000 × hull left |
| Endless | +10 per second survived |
| Fallback cards | Repair (at full hull) 500, Data Cache 300 |

Mutators can multiply score (§14). The best score is kept per sector, for Endless, and per Daily date.

### Credits

```
credits = floor( ( floor(score / 200) + floor(kills / 10) + clear_bonus ) × credit_gain )
clear_bonus = 100 × sector number if the sector was cleared, else 0
```

- Endless: `clear_bonus` = 40 per boss defeated.
- Daily: the first finished Daily run each day gets +100.
- The rewarded "double credits" doubles the final number once per run (§15).

Examples:

- Died at 4:00 in Sector 1 with 9,000 score and 260 kills → 45 + 26 = **71 credits**.
- Cleared Sector 1 with 28,000 score and 650 kills → 140 + 65 + 100 = **305 credits**.

### Meta upgrades (shop)

Ranks are permanent and add stat modifiers to every run except Daily. Costs follow `round_to_10(base_cost × 1.6^(rank − 1))`.

| Id | Name | Ranks | Per rank | Costs |
|---|---|---|---|---|
| `firepower` | Firepower | 5 | `damage` +4% | 100, 160, 260, 410, 660 |
| `cycling` | Rapid Cycling | 5 | `fire_rate` +3% | 100, 160, 260, 410, 660 |
| `reinforced_hull` | Reinforced Hull | 2 | `max_hull` +1 | 400, 640 |
| `magnetism` | Magnetism | 3 | `pickup_radius` +10% | 80, 130, 200 |
| `graze_sensors` | Graze Sensors | 3 | `graze_radius` +5% | 120, 190, 310 |
| `overdrive_primer` | Overdrive Primer | 2 | `overdrive_start` +25 | 200, 320 |
| `salvage_rights` | Salvage Rights | 5 | `credit_gain` +5% | 150, 240, 380, 610, 980 |
| `spare_reroll` | Spare Reroll | 2 | `rerolls` +1 | 300, 480 |

Everything in the shop plus both ships costs about 13,000 credits: roughly 45–60 runs. You can't buy without enough credits, and ranks can't be refunded.

---

## 14. Daily runs and mutators

**Daily seed:** the UTC date as an integer, `YYYYMMDD` (e.g. 20261013). From that seed, in this fixed order:

1. Pick the **ship** from all three (even locked ones; Daily lends it).
2. Pick the **sector** (1–3).
3. Pick the **mutator**.
4. Seed the run itself.

The same date always gives the same ship, sector, mutator and spawn sequence.

**Daily rules:** meta upgrades off, revive off, 1 reroll, unlimited attempts. The best score per date is saved locally (an online leaderboard is optional, task 3.17).

### Mutators

| Id | Name | Effect |
|---|---|---|
| `glass_cannon` | Glass Cannon | `damage` +50%, max hull is 1, score ×1.3 |
| `bullet_storm` | Bullet Storm | Enemies fire 40% faster, graze score ×2 |
| `magnet_storm` | Magnet Storm | `pickup_radius` ×3, `xp_gain` −20% |
| `swarm_season` | Swarm Season | Spawn rates ×1.5, enemy HP ×0.7 |
| `overcharged` | Overcharged | `overdrive_gain` ×2, Overdrive damage ×0.5 |
| `fast_forward` | Fast Forward | Enemy and bullet speed ×1.25, score ×1.4 |
| `lone_gun` | Lone Gun | 1 weapon slot, `damage` +30% |
| `thin_margins` | Thin Margins | `graze_radius` ×0.7, graze score ×3 |

`MutatorData` holds player stat modifiers plus game-side multipliers: `enemy_hp_mult`, `enemy_speed_mult`, `enemy_fire_rate_mult`, `spawn_rate_mult`, `score_mult`, `graze_score_mult`, `weapon_slots` (−1 = unchanged), `max_hull_override` (−1 = unchanged).

---

## 15. Monetization and privacy

**Promises (shown in the store listing too):** free to play, **no forced ads** (no banners, no interstitials, ever), no energy or timers, no paid currency, no loot boxes.

### Rewarded ads (opt-in, each at most once per run)

| Placement | Where | Reward |
|---|---|---|
| `revive` | SIGNAL LOST screen, with a 5 s countdown | Continue with 1 hull; all enemy bullets cleared (no shards); 2 s invulnerability. Not in Daily. |
| `double_credits` | Results | Credits from this run ×2 |
| `extra_reroll` | Level-up panel, when rerolls are 0 | +1 reroll |

If an ad isn't available or fails, the button says "No ad available right now" and the game carries on. It never waits more than 15 s (ARCHITECTURE.md §7).

### Supporter Pack

`supporter_pack`, a one-time purchase at **$3.99**.

- Every rewarded placement is granted instantly, without an ad.
- A "Supporter Gold" colour palette and a gold ship trail (cosmetic).
- No other gameplay advantage: everyone can get the same rewards by watching ads.
- Restorable on reinstall (restore on start and from Settings).

### Privacy

- Progress is saved only on the device. No accounts, no own servers.
- AdMob shows rewarded ads. Google's consent form (UMP) is shown where the law requires it, before any ad loads, and "Privacy options" is in Settings.
- Google Play Billing handles the purchase.
- No analytics at launch. If added (task 3.17), it must be in the privacy policy and the Data safety form.
- Target audience 13+, not designed for children.

---

## 16. Look and colours

**Style:** glowing vector outlines on a near-black animated grid, in the spirit of Geometry Wars. Every object is a polygon outline drawn in code (`fx/neon_shape.gd`): line width 3 px, plus a glow.

**Brightness order (most to least visible):** enemy bullets → player hitbox dot → enemies → player ship → XP shards → player shots → background. If anything breaks this order, it's a bug.

### Default palette (Sector 1)

| Element | Colour |
|---|---|
| Background | `#05060D` |
| Grid lines | `#1B1F5C` at 35% alpha; pulse brighter on the beat (task 2.10) |
| Player ship | Cyan `#29F3FF` |
| Player hitbox dot | White `#FFFFFF` with a 1 px dark outline |
| Player shots | Pale cyan `#9FFBFF` at 70% alpha (deliberately dimmer than enemy bullets) |
| Enemy bullets | White core `#FFFFFF`, rim hot pink `#FF2E88` (default), orange `#FF7A00`, yellow `#FFE600` |
| Drifter / Mini | Magenta `#FF2BD6` / pink `#FF5C8A` |
| Charger | Orange `#FF8A1F` |
| Spinner Turret | Yellow `#FFE53B` |
| Swarm | Violet `#A45CFF` |
| Splitter | Pink `#FF5C8A` |
| Sniper | Red `#FF3B3B` |
| Shielded | Blue `#3B7BFF`, white shield arc |
| Mine-layer / Mine | Amber `#FFB13B` |
| XP shards | Green `#5CFF7A` |
| Overdrive | Gold `#FFD23B` |
| UI text / accent | `#E8F6FF` / cyan `#29F3FF` |
| Telegraph lines | The enemy's colour at 50% alpha, 2 px |

### Sector palettes

| Sector | Grid | Accent | Default bullet rim |
|---|---|---|---|
| 1 Outer Grid | `#1B1F5C` | Magenta | Hot pink `#FF2E88` |
| 2 Signal Reef | `#0E3A3A` | Orange | Orange `#FF7A00` |
| 3 Null Core | `#2A0E3A` | Red | Red-white `#FF4D6A` |

Accessibility palettes (§18) swap colours through the same `PaletteData`.

### Typography and UI

- One display font for titles and numbers (suggestion: **Orbitron**, SIL Open Font License, free for commercial use; ask before downloading). Body text uses the same font or Godot's default.
- Minimum text size on a phone: 28 px at design resolution for body text, 22 px for small labels.
- Buttons are at least 96 × 96 px.

### Juice

- **Version 1 (task 1.9):** hit flash (white, 0.06 s), small screen shake on hull hit and Overdrive, death particles in the enemy's colour, graze sparks.
- **Version 2 (task 2.10):** hit-stop (0.05 s on Charger and larger kills, 0.12 s on boss parts), Overdrive shockwave, card-pick animation, wave-clear flourish, grid pulsing with the music.
- All of it respects the screen shake and reduced flashing settings.

---

## 17. Audio and haptics

Sounds come from CC0 or commercially licensed sources, each listed in `CREDITS.md` with its licence. Each sound has a cooldown so a hundred kills in a second don't become a hundred sounds, and a slight random pitch (±5%) unless noted.

| Id | When | Cooldown | Notes |
|---|---|---|---|
| `player_shoot` | Any player weapon fires | 0.06 s | Very soft, low in the mix |
| `enemy_hit` | Enemy takes damage | 0.04 s | |
| `enemy_die` | Enemy dies | 0.03 s | Bigger variant for Turret and larger |
| `graze` | Graze | 0.03 s | Pitch rises with chain: +1 semitone per 5 chain, max +12 |
| `chain_break` | Chain of 10+ resets | — | |
| `hull_hit` | Player loses hull | — | Plus haptic |
| `overdrive_ready` | Meter becomes full | — | |
| `overdrive_fire` | Overdrive fires | — | Plus haptic |
| `shard_pickup` | Shard collected | 0.03 s | Pitch steps up while collecting rapidly |
| `level_up` | Level-up panel opens | — | |
| `card_pick` / `reroll` | UI | — | |
| `wave_banner` | Wave starts | — | |
| `boss_warning` | WARNING banner | — | |
| `boss_phase` | Phase change | — | |
| `boss_die` | Boss defeated | — | Plus haptic |
| `sector_clear` / `signal_lost` | End of run | — | |
| `ui_tap` / `ui_back` | Menus | — | |
| `purchase` | Supporter Pack bought | — | |

**Music:** `menu`, `sector` (one loop for the MVP; per-sector tracks later), `boss`. The Music bus defaults to 80%, SFX to 100%. Music ducks by 30% during the level-up panel.

**Haptics** (if enabled): hull hit 60 ms, Overdrive 120 ms, boss defeat 200 ms. Nothing else, so they stay meaningful.

---

## 18. Settings and accessibility

All settings are stored in the profile, applied live, and changeable during a run from the pause panel.

| Setting | Options | Default |
|---|---|---|
| Music volume | 0–100% | 80% |
| SFX volume | 0–100% | 100% |
| Haptics | On / Off | On |
| Screen shake | Off / Low (50%) / Full | Full |
| Reduced flashing | On / Off (removes white full-screen flashes, softens hit flashes) | Off |
| Drag sensitivity | 0.6–1.6 in steps of 0.1 | 1.0 |
| Glow | Off / Fast / Fancy (fake halo vs. real glow, decided in task 2.1) | Set after the phone test |
| Colour palette | Default / Deuteranopia & Protanopia (blue–orange) / Tritanopia (red–cyan) / High contrast (white bullets with thick dark outline) / Supporter Gold (if owned) | Default |
| Bullet outline boost | On / Off (thicker dark outline around enemy bullets) | Off |
| Pause button side | Right / Left | Right |
| Show FPS | On / Off | Off |
| Privacy options | Opens the consent form (shown only when required) | — |
| Restore purchases | Button | — |
| Credits / Privacy policy | Buttons (credits screen; opens the policy URL) | — |

Language: English only at launch. Keep all UI text in one place (a `strings` resource or Godot's translation CSV) so translations are easy later.
