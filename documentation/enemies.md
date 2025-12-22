# Enemies Schema Guide

## What Are Enemies?

The enemies schema defines **enemy properties** - how much health they have, what they drop, how they attack, and what damages them.

This information is used by the logic system to:
- Calculate if you can kill enemies
- Determine damage taken
- Figure out ammo costs
- Calculate farming potential

## Schema Location

`schema/mxf-enemies.schema.json`

## Basic Structure

```json
{
  "enemies": [
    {
      "id": 1,
      "name": "Sidehopper",
      "attacks": [...],
      "hp": 100,
      "amountOfDrops": 1,
      "drops": {...},
      "dims": {"w": 16, "h": 24},
      "freezable": false,
      "grapplable": false,
      "invul": [],
      "damageMultipliers": [...],
      "areas": ["MDK", "SRX"]
    }
  ]
}
```

## Enemy Fields

### id
Unique identifier for this enemy:

```json
"id": 1
```

**Note**: Must be unique across ALL enemies

### name
The enemy's name:

```json
"name": "Sidehopper"
```

**Important**: This is what you reference in requirements:

```json
"enemyDamage": {
  "enemy": "Sidehopper",
  "type": "contact",
  "hits": 2
}
```

**Naming Convention**: Use the name from the game or wiki

### attacks
Array of attacks this enemy can perform:

```json
"attacks": [
  {
    "name": "contact",
    "baseDamage": 30
  },
  {
    "name": "projectile",
    "baseDamage": 20,
    "affectedByVaria": true,
    "affectedByGravity": false
  }
]
```

**Attack Fields**:

#### name
What kind of attack:

```json
"name": "contact"
```

**Common Names**:
- `contact`: Touching the enemy
- `projectile`: Fired projectile
- `explosion`: Area explosion
- `grab`: Special grab attack
- `beam`: Beam weapon

#### baseDamage
Damage dealt with only Power Suit:

```json
"baseDamage": 30
```

**Note**: This is the base amount before Varia/Gravity reductions

#### affectedByVaria
Does Varia Suit reduce this damage?

```json
"affectedByVaria": true
```

**Default**: `true` if not specified

**Damage Calculation**:
- With Varia: `baseDamage * 0.5`
- Without: `baseDamage`

#### affectedByGravity
Does Gravity Suit reduce this damage?

```json
"affectedByGravity": true
```

**Default**: `true` if not specified

**Damage Calculation**:
- With Gravity: `baseDamage * 0.5`
- With both Varia + Gravity: `baseDamage * 0.25`

### hp
Total health points:

```json
"hp": 100
```

**Note**: Used to calculate kills and weapon effectiveness

### amountOfDrops
How many drops this enemy leaves:

```json
"amountOfDrops": 1
```

**Common Values**:
- Regular enemies: `1`
- Strong enemies: `2-5`
- Bosses: `10-20`

### drops
Drop rate table (all values out of 102):

```json
"drops": {
  "noDrop": 70.0,
  "yellowX": 20.0,
  "redX": 7.0,
  "greenX": 5.0
}
```

**Fields**:
- `noDrop`: Chance of nothing
- `yellowX`: Energy drops
- `redX`: Large energy + ammo drops
- `greenX`: Ammo drops

**Total Must Equal 102**:
```
70 + 20 + 7 + 5 = 102 ✓
```

**Example Calculation**:
- 70/102 = ~68.6% no drop
- 20/102 = ~19.6% energy
- 7/102 = ~6.9% large drops
- 5/102 = ~4.9% ammo

### farmableDrops (Optional)
For enemies that spawn farmable particles:

```json
"farmableDrops": {
  "noDrop": 0.0,
  "yellowX": 50.0,
  "redX": 30.0,
  "greenX": 22.0
}
```

**Use For**: Enemies like Metroids that you can farm repeatedly

### dims
Sprite dimensions in pixels:

```json
"dims": {
  "w": 16,
  "h": 24
}
```

**Width**: Horizontal size
**Height**: Vertical size

**Note**: Used for hitbox calculations (advanced)

### freezable
Can Ice Beam freeze this enemy?

```json
"freezable": false
```

**Values**:
- `true`: Can be frozen
- `false`: Cannot be frozen

### grapplable
Can Grapple Beam latch onto this enemy?

```json
"grapplable": false
```

**Values**:
- `true`: Can grapple
- `false`: Cannot grapple

**Use For**: Enemies you can swing from

### invul
Weapons that CANNOT damage this enemy:

```json
"invul": [
  "Power Bomb",
  "Bomb"
]
```

**Empty if vulnerable to everything**:
```json
"invul": []
```

**Common Invulnerabilities**:
- Beam-immune enemies: `["Beam", "ChargedBeam"]`
- Bomb-immune enemies: `["Bomb", "PowerBombBlast"]`
- Everything-immune: `["All"]` (rare)

### damageMultipliers
Special damage modifiers for specific weapons:

```json
"damageMultipliers": [
  {
    "weapon": "Missile",
    "value": 2.0
  },
  {
    "weapon": "Charge Beam",
    "value": 0.5
  }
]
```

**Fields**:
- `weapon`: Weapon or category name
- `value`: Multiplier applied to weapon damage

**Values**:
- `< 1.0`: Resistance (takes less damage)
- `= 1.0`: Normal damage
- `> 1.0`: Weakness (takes more damage)

**Examples**:
- `2.0`: Takes double damage (weak to this weapon)
- `0.5`: Takes half damage (resistant)
- `3.0`: Takes triple damage (very weak)
- `0.1`: Takes 10% damage (very resistant)

### dropsUpgrade (Optional)
Items dropped when killed (for bosses):

```json
"dropsUpgrade": [
  "Varia Suit",
  "Energy Tank"
]
```

**Use For**: Bosses that drop items on death

### areas
Where this enemy naturally appears:

```json
"areas": [
  "MDK",
  "SRX",
  "TRO"
]
```

**Area Codes**:
- `MDK`: Main Deck
- `SRX`: SRX Sector
- `TRO`: TRO Sector
- `PYR`: PYR Sector
- `AQA`: AQA Sector
- `ARC`: ARC Sector
- `NOC`: NOC Sector

## Complete Example: Regular Enemy

```json
{
  "id": 5,
  "name": "Sidehopper",
  "attacks": [
    {
      "name": "contact",
      "baseDamage": 30,
      "affectedByVaria": true,
      "affectedByGravity": true
    }
  ],
  "hp": 100,
  "amountOfDrops": 1,
  "drops": {
    "noDrop": 70.0,
    "yellowX": 20.0,
    "redX": 7.0,
    "greenX": 5.0
  },
  "dims": {
    "w": 16,
    "h": 24,
    "note": "Approximate sprite size"
  },
  "freezable": false,
  "grapplable": false,
  "invul": [],
  "damageMultipliers": [
    {
      "weapon": "Missile",
      "value": 2.0
    }
  ],
  "areas": ["MDK", "SRX"],
  "note": "Common hopping enemy found in early areas"
}
```

**What This Means**:
- 100 HP
- Contact does 30 damage (15 with Varia, ~7 with Varia+Gravity)
- Usually drops nothing (70%)
- Takes double damage from missiles (weak point)
- Cannot be frozen
- Cannot be grappled
- Found in MDK and SRX

## Complete Example: Boss Enemy

```json
{
  "id": 100,
  "name": "Kraid",
  "attacks": [
    {
      "name": "claw",
      "baseDamage": 50,
      "affectedByVaria": true,
      "affectedByGravity": true
    },
    {
      "name": "projectile",
      "baseDamage": 40,
      "affectedByVaria": true,
      "affectedByGravity": true
    }
  ],
  "hp": 1000,
  "amountOfDrops": 15,
  "drops": {
    "noDrop": 0.0,
    "yellowX": 60.0,
    "redX": 30.0,
    "greenX": 12.0
  },
  "dims": {
    "w": 64,
    "h": 96,
    "note": "Large boss sprite"
  },
  "freezable": false,
  "grapplable": false,
  "invul": [
    "Bomb",
    "PowerBombBlast"
  ],
  "damageMultipliers": [
    {
      "weapon": "Super",
      "value": 3.0
    },
    {
      "weapon": "Charge Beam",
      "value": 1.5
    }
  ],
  "dropsUpgrade": ["Varia Suit"],
  "areas": ["MDK"],
  "note": "Large boss. Weak to Super Missiles."
}
```

**What This Means**:
- 1000 HP (boss)
- Two attacks: claw (50 dmg) and projectile (40 dmg)
- Drops 15 items on death (100% drop rate)
- Immune to bombs
- Weak to Supers (3x damage)
- Drops Varia Suit when killed
- Only in MDK

## Damage Calculation Examples

### Example 1: Basic Contact

**Enemy**: Sidehopper (30 base damage)
**Player**: Power Suit only

```
Damage = 30
```

### Example 2: With Varia Suit

**Enemy**: Sidehopper (30 base damage)
**Player**: Varia Suit

```
Damage = 30 * 0.5 = 15
```

### Example 3: With Both Suits

**Enemy**: Sidehopper (30 base damage)
**Player**: Varia + Gravity

```
Damage = 30 * 0.5 * 0.5 = 7.5 → 8 (rounded up)
```

### Example 4: Unaffected Attack

**Enemy**: Special projectile
```json
{
  "baseDamage": 50,
  "affectedByVaria": false,
  "affectedByGravity": false
}
```

**Player**: Varia + Gravity

```
Damage = 50 (no reduction!)
```

## Damage Multipliers in Action

### Example 1: Weakness

**Enemy**: Has multiplier
```json
{
  "weapon": "Missile",
  "value": 2.0
}
```

**Weapon**: Missile (50 base damage)

```
Damage to enemy = 50 * 2.0 = 100 HP
```

### Example 2: Resistance

**Enemy**: Has multiplier
```json
{
  "weapon": "Beam",
  "value": 0.5
}
```

**Weapon**: Beam (20 base damage)

```
Damage to enemy = 20 * 0.5 = 10 HP
```

### Example 3: Category Multiplier

**Enemy**: Has multiplier
```json
{
  "weapon": "ChargedBeam",
  "value": 3.0
}
```

**Weapon**: Charge Beam (40 base damage)
**Weapon Category**: `ChargedBeam`

```
Damage to enemy = 40 * 3.0 = 120 HP
```

## Weapon Categories

Damage multipliers can apply to **weapon categories**:

| Category | Includes |
|----------|----------|
| `All` | Every weapon |
| `Beam` | All beam weapons |
| `ChargedBeam` | Only charged beams |
| `UnchargedBeam` | Only uncharged beams |
| `PowerBombBlast` | Power Bomb explosions |

**Example**: Enemy resistant to all beams
```json
"damageMultipliers": [
  {
    "weapon": "Beam",
    "value": 0.25
  }
]
```

## How Enemies Are Used in Logic

### In Requirements (Taking Damage)

```json
"requires": [
  {
    "enemyDamage": {
      "enemy": "Sidehopper",
      "type": "contact",
      "hits": 3
    }
  }
]
```

**Means**: Must take 3 contact hits from Sidehoppers

**Damage Calculation**:
1. Look up Sidehopper's "contact" attack (30 base)
2. Apply suit reductions
3. Multiply by 3 hits
4. Check if player has enough HP

### In Requirements (Killing Enemies)

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [
        ["Sidehopper"],
        ["Sidehopper"],
        ["Sidehopper"]
      ]
    }
  }
]
```

**Means**: Must kill 3 Sidehoppers

**Logic Checks**:
1. Find available weapons
2. Check weapon damages against Sidehopper
3. Calculate ammo needed
4. Verify player has enough

### In Rooms (Enemy Groups)

```json
"enemies": [
  {
    "id": "e1",
    "groupName": "Corridor Sidehoppers",
    "enemyName": "Sidehopper",
    "quantity": 3,
    "homeNodes": [1, 2]
  }
]
```

Defines where enemies appear in rooms (see Room-Schema.md)

## How to Add a New Enemy

### Step 1: Gather Information

**Name**: "Space Pirate"
**HP**: 200
**Attack**: Contact (40 damage)
**Drops**: Standard drop table
**Areas**: SRX, ARC

### Step 2: Find Next ID

Check existing enemies:
- ID 50: Last enemy
- Next: **ID 51**

### Step 3: Write the JSON

```json
{
  "id": 51,
  "name": "Space Pirate",
  "attacks": [
    {
      "name": "contact",
      "baseDamage": 40,
      "affectedByVaria": true,
      "affectedByGravity": true
    },
    {
      "name": "laser",
      "baseDamage": 30,
      "affectedByVaria": true,
      "affectedByGravity": false
    }
  ],
  "hp": 200,
  "amountOfDrops": 1,
  "drops": {
    "noDrop": 60.0,
    "yellowX": 25.0,
    "redX": 10.0,
    "greenX": 7.0
  },
  "dims": {
    "w": 24,
    "h": 32
  },
  "freezable": false,
  "grapplable": false,
  "invul": [],
  "damageMultipliers": [
    {
      "weapon": "Charge Beam",
      "value": 1.5
    }
  ],
  "areas": ["SRX", "ARC"],
  "note": "Armored enemy. Slightly weak to charged shots."
}
```

### Step 4: Add to enemies Array

```json
{
  "enemies": [
    // ... existing enemies ...
    {
      "id": 51,
      "name": "Space Pirate",
      // ... rest of enemy data ...
    }
  ]
}
```

### Step 5: Use in Rooms

Now you can reference it:

```json
"enemies": [
  {
    "groupName": "Hangar Pirates",
    "enemyName": "Space Pirate",
    "quantity": 2
  }
]
```

And in requirements:

```json
"requires": [
  {
    "enemyDamage": {
      "enemy": "Space Pirate",
      "type": "laser",
      "hits": 1
    }
  }
]
```

## Drop Tables

### Understanding Drop Rates

All values are **out of 102** (game's internal system):

```json
"drops": {
  "noDrop": 70.0,
  "yellowX": 20.0,
  "redX": 7.0,
  "greenX": 5.0
}
```

**Percentages**:
- noDrop: 70/102 = 68.6%
- yellowX: 20/102 = 19.6%
- redX: 7/102 = 6.9%
- greenX: 5/102 = 4.9%

### Common Drop Tables

**Weak Enemy** (rarely drops):
```json
{
  "noDrop": 85.0,
  "yellowX": 12.0,
  "redX": 3.0,
  "greenX": 2.0
}
```

**Regular Enemy** (moderate drops):
```json
{
  "noDrop": 70.0,
  "yellowX": 20.0,
  "redX": 7.0,
  "greenX": 5.0
}
```

**Strong Enemy** (good drops):
```json
{
  "noDrop": 50.0,
  "yellowX": 30.0,
  "redX": 15.0,
  "greenX": 7.0
}
```

**Boss** (guaranteed drops):
```json
{
  "noDrop": 0.0,
  "yellowX": 60.0,
  "redX": 30.0,
  "greenX": 12.0
}
```

## Common Mistakes

### ❌ Mistake 1: Drop Table Doesn't Total 102

```json
// WRONG - totals 100
"drops": {
  "noDrop": 70.0,
  "yellowX": 20.0,
  "redX": 5.0,
  "greenX": 5.0  // Total = 100 ❌
}
```

**Fix**: Must total exactly 102

### ❌ Mistake 2: Missing Attack Fields

```json
// WRONG - no baseDamage
"attacks": [
  {
    "name": "contact"
    // Missing baseDamage!
  }
]
```

**Fix**: All attacks need baseDamage

### ❌ Mistake 3: Duplicate IDs

```json
// WRONG - same ID
{"id": 10, "name": "EnemyA"},
{"id": 10, "name": "EnemyB"}  // ← Duplicate!
```

**Fix**: Each enemy needs unique ID

### ❌ Mistake 4: Invalid Weapon Names

```json
// WRONG - weapon doesn't exist
"damageMultipliers": [
  {
    "weapon": "Super Laser Beam",  // ← Not a real weapon!
    "value": 2.0
  }
]
```

**Fix**: Use weapon names from weapons.json

## Testing Your Enemy

### Test Damage Taken

```json
"requires": [
  {
    "enemyDamage": {
      "enemy": "Your New Enemy",
      "type": "contact",
      "hits": 1
    }
  }
]
```

Verify damage is calculated correctly

### Test Enemy Kill

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [["Your New Enemy"]]
    }
  }
]
```

Verify you can kill it with available weapons

## Quick Reference

### Required Fields

- ✅ `id`: Unique number
- ✅ `name`: Enemy name
- ✅ `attacks`: Array of attacks
- ✅ `hp`: Health points
- ✅ `amountOfDrops`: Drop quantity
- ✅ `drops`: Drop table (total = 102)
- ✅ `dims`: Sprite dimensions
- ✅ `freezable`: Can be frozen?
- ✅ `grapplable`: Can be grappled?
- ✅ `invul`: Invulnerabilities
- ✅ `damageMultipliers`: Damage modifiers

### Optional Fields

- `farmableDrops`: Farmable particle drops
- `dropsUpgrade`: Items dropped
- `areas`: Where enemy appears
- `note`: Description

## Next Steps

- Read [Weapons-Schema.md](Weapons-Schema.md) to understand weapon damage
- Check [Requirements-Schema.md](Requirements-Schema.md) for enemyDamage/enemyKill
- See [Room-Schema.md](Room-Schema.md) to place enemies in rooms