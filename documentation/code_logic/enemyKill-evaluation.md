# enemyKill Evaluation: How Boss/Enemy Fights Are Actually Checked

This document describes how the C++ engine (`cpp/src/enemy_kill_evaluator.cpp`,
invoked from `logical_evaluator.cpp` for any `enemyKill` requirement) decides
whether a player can complete a fight. `documentation/requirements.md`
documents the JSON *shape* of `enemyKill`; this document describes what the
engine actually *does* with it — ammo math, weapon selection, difficulty
scaling, and the special cases that don't fit the generic formula.

## The basic pipeline

For a requirement like:

```json
{
  "enemyKill": {
    "enemies": [["Zazabi-X (Blue)", "Zazabi-X (Red)", "Zazabi Core-X"]],
    "excludedWeapons": ["Charge"]
  }
}
```

the engine, per unique enemy name in `enemies`:

1. Looks the enemy up in `enemies/main.json` (regular enemies) or
   `enemies/bosses/main.json` (bosses — anything referenced there is treated
   as a boss for HP/energy-cost purposes, everything else as a regular
   enemy).
2. Builds the list of "usable weapons": every weapon the player currently has
   access to (`useRequires` satisfied, ammo capacity > 0 if it costs ammo),
   filtered by this requirement's `explicitWeapons` (if present, ONLY these
   are considered) and `excludedWeapons` (these are removed regardless of
   what else is usable).
3. Filters further to weapons that can actually hit this specific enemy —
   removed if the enemy's own `invul` list (in main.json) names the weapon or
   one of its `categories`.
4. Picks the best remaining weapon (highest damage to this enemy, preferring
   ammo-free weapons when damage ties) and computes `shotsNeeded =
   ceil(enemyHP / damagePerShot)`.
5. Sums ammo cost and (for bosses) an estimated energy cost across every
   enemy named in the requirement.
6. Checks the totals against the player's **actual current** ammo/HP. If
   insufficient, the requirement is **not satisfied** — this is a hard gate,
   not a suggestion.

If a name in `enemies` isn't found in either enemies file, the engine assumes
it's killable rather than breaking the seed — so a typo'd enemy name fails
silently open, not closed. Always double check enemy names against
`enemies/main.json` / `enemies/bosses/main.json` when writing a new
`enemyKill` requirement.

## Ammo sufficiency and Reserve-X

Ammo cost is compared against the player's **real, current** ammo count —
not an inflated "assume you'll find more later" number — with one
exception: **if the player has at least one Reserve-X, ammo is treated as
effectively unlimited** for reachability purposes. This models the fact that
Reserve-X implies the player can always return to a save station, refill,
and grind back up, which makes any *finite* ammo requirement eventually
satisfiable given enough time. Without at least one Reserve-X, the player's
literal current missile/Power Bomb count is what's checked — a fight sitting
right at the start of a route with only starting ammo (typically 5 missiles)
and no Reserve-X will correctly fail an `enemyKill` that needs more shots
than that.

Boss **HP/energy** checks do *not* get the Reserve-X-is-infinite treatment —
Reserve-X HP is discounted for boss fights instead (first tank counts as 60
HP, each additional tank as 40 HP, reflecting the slow pause-menu regen
mechanic), since a boss fight is a sustained real-time HP check, not
something you can pause and grind indefinitely.

## Missile levels vs. missile items — naming

The item you pick up is `SuperMissile` / `IceMissile` / `DiffusionMissile`.
The **weapon** you fire once you have it is named `Super` / `IceMissile` /
`Diffusion` (yes, `IceMissile` is spelled the same both ways — only `Super`
and `Diffusion` differ from their item names). When writing
`explicitWeapons` / `excludedWeapons`, use the **weapon** names (`Super`,
`Diffusion`), not the item names — `weapons.json`'s `name` field is
authoritative.

Missile level is derived, not a separate flag:

| Level | Granted by | Weapon name |
|-------|-----------|-------------|
| 1 | (base game) | `Missile` |
| 2 | `SuperMissile` | `Super` |
| 3 | `IceMissile` | `IceMissile` |
| 4 | `DiffusionMissile` | `Diffusion` |

Each level *requires* the previous ones (having `DiffusionMissile` implies
levels 1-3 too) and increases per-shot damage — higher missile level makes
every missile-family fight easier, not just ones that explicitly ask for the
higher-tier weapon. A fight that lists `explicitWeapons: ["Super"]` is
requiring missile level ≥ 2, and the ammo-sufficiency math automatically
uses `Super`'s higher per-shot damage once the player has it.

## invul / excludedWeapons / explicitWeapons — where each one lives

There are two independent layers, and they combine (both must allow a
weapon for it to be usable):

- **`invul`** lives on the enemy in `enemies/main.json` /
  `enemies/bosses/main.json`. It's a property of the *enemy* — permanent,
  applies to every `enemyKill` requirement that references this enemy
  anywhere in the game. Use it for "this enemy is immune to bombs" style
  facts.
- **`excludedWeapons` / `explicitWeapons`** live on the *requirement itself*
  (the specific `enemyKill` block in a room's `requires`). They're
  situational to that specific fight/room — use them for "in this particular
  arena, the arena geometry/script makes this weapon impractical or
  disabled," not for permanent enemy properties (that's what `invul` is
  for). A boss can have different `explicitWeapons` in different rooms if
  its accessible geometry changes the practical weapon set (rare, but the
  schema allows it).

`damageMultipliers` (also on the enemy) is a third, separate axis — it
doesn't gate whether a weapon works, it scales how much damage it does once
it's already past `invul` and the requirement's weapon filters.

## Difficulty scaling

Two independent settings affect the math, and both are read live at
evaluation time (not baked into the data):

- **`itemProgression`** (logic difficulty: normal/tricky/technical/
  challenge) scales the *safety margin* added to estimated boss energy
  cost — `resourceLeniency_` in the evaluator, 1.5× at normal down to 1.0×
  at challenge. Easier logic assumes a less skilled player who takes more
  hits, so the same fight is estimated to cost more HP at `normal` than at
  `challenge`.
- **`ingameDifficulty`** (easy/normal/hard/omega) scales incoming boss
  damage via `state.difficulty.dmgMul` (Hard/Omega hit harder) — this is the
  in-game damage multiplier already used everywhere else (environmental
  damage, contact damage), applied identically here.

Both apply on top of suit reduction (Varia halves incoming damage, Gravity
quarters it) and are already wired through — no per-boss configuration
needed for ordinary difficulty scaling.

## Boss scenarios (mostly unused)

`enemies/bosses/scenarios.json` supports a much more detailed fight model
(damage windows, dodge rate, incoming-attack frequency, energy farming from
drops) for bosses that have an entry there — see
`schema/mxf-bossScenarios.schema.json`. As of this writing it only has
entries for a handful of vanilla-named bosses (Draygon, Ridley, Phantoon,
Botwoon, Mother Brain 2) and none of this hack's renamed/custom bosses use
it. Every actual boss in this data set (Zazabi, Neo-Ridley, Meta Draygon-X,
Spikespawn, etc.) falls through to the **generic fallback**: estimate shots
from `boss.hp` and the best available weapon's damage, estimate energy cost
from the boss's highest `baseDamage` attack scaled by a flat ~5%-of-HP
heuristic, suit reduction, difficulty, and logic leniency. This fallback is
deliberately simple — if a specific fight needs more accurate modeling than
"total HP ÷ damage per shot," don't reach for `scenarios.json` (it's not
consumed by anything besides those 5 legacy entries) — see the next section
instead.

## When the generic formula isn't enough: hard-coded special cases

Some fights have mechanics the generic per-enemy HP/damage model can't
represent — most notably anything where a weapon matters for reasons other
than raw damage-per-shot (suppressing a barrier, interrupting an attack
pattern, etc.). These are handled as explicit, named checks in
`EnemyKillEvaluator::evaluateBossKill()` in the C++ source, gated on
`boss.name`, rather than as generic JSON fields — the formula has no concept
of "this weapon enables that weapon" so there's no schema-level way to
express it.

**Current special case: Meta Draygon-X.** Charge Beam deals no real damage
to Draygon's HP (hence it's in `excludedWeapons` for the `enemyKill` block —
picking it as "the kill weapon" would be wrong), but it suppresses a barrier
that otherwise blocks missile damage from landing at all, and the barrier's
required suppression rate isn't something the flat HP/damage formula models.
Rather than extend the schema for one boss, the engine hard-requires
`state.hasItem("Charge")` for this specific boss name, gated by difficulty:

- Logic difficulty `normal` or `tricky`: Charge is required outright,
  regardless of in-game difficulty.
- Logic difficulty `technical` or `challenge`: Charge is only required if
  in-game difficulty is `hard` or `omega` (a skilled player can compensate
  without it at Easy/Normal in-game difficulty, but the barrier's growth
  outpaces missile-only DPS at Hard/Omega regardless of skill).

If a future boss needs similar treatment, follow this pattern: add a named
check near the top of `evaluateBossKill()`, keyed on `boss.name`, before it
falls through to the generic formula — don't try to force it into
`damageMultipliers` or `invul` if the mechanic isn't actually about raw
damage.
