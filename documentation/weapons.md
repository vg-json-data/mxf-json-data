# Weapons Schema Guide

## What Are Weapons?

The weapons schema defines **weapon properties** - how much damage they deal, how quickly you can fire them, what ammo they cost, and what items you need to use them.

This information is used by the logic system to:
- Calculate if you can kill enemies
- Determine best weapon for each situation
- Figure out ammo costs for fights
- Choose weapons for enemy kills

## Schema Location

`schema/mxf-weapons.schema.json`

## Basic Structure

```json
{
  "weapons": [
    {
      "id": 1,
      "name": "Power Beam",
      "damage": 20,
      "cooldownFrames": 15,
      "useRequires": [],
      "shotRequires": [],
      "situational": false,
      "hitsGroup": false,
      "categories": ["Beam", "UnchargedBeam"]
    }
  ]
}
```

## Weapon Fields

### id
Unique identifier:

```json
"id": 1
```

**Note**: Must be unique across ALL weapons

### name
The weapon's name:

```json
"name": "Missile"
```

**Important**: This is what you reference in:
- Enemy damage multipliers
- Weapon requirements
- Kill requirements

**Examples**:
- `Power Beam`
- `Missile`
- `Super Missile`
- `Charge Beam`
- `Bomb`

### damage
Base damage dealt to enemies:

```json
"damage": 50
```

**Common Values**:
- Weak beams: `10-20`
- Normal weapons: `30-50`
- Strong weapons: `100-150`
- Very strong: `200+`

**Note**: This is base damage before enemy multipliers

### cooldownFrames
Frames between shots:

```json
"cooldownFrames": 15
```

**Understanding Frames**:
- 60 frames = 1 second
- 15 frames = 0.25 seconds
- 30 frames = 0.5 seconds

**Common Values**:
- Rapid fire: `10-20`
- Normal: `30-45`
- Slow: `60+`

### useRequires
Requirements to USE this weapon:

```json
"useRequires": [
  "Missiles",
  "Missile Launcher"
]
```

**Can be empty** if always available:
```json
"useRequires": []
```

**Can be complex**:
```json
"useRequires": [
  "Charge",
  {
    "or": [
      "Ice Beam",
      "Wave Beam"
    ]
  }
]
```

### shotRequires (Optional)
Requirements for EACH shot:

```json
"shotRequires": [
  {
    "ammo": {
      "type": "Missile",
      "count": 1
    }
  }
]
```

**Use For**: Weapons that consume ammo per shot

**Empty for free weapons**:
```json
"shotRequires": []
```

### situational
Is this weapon situational?

```json
"situational": false
```

**Values**:
- `false`: Always usable when requirements met
- `true`: Only usable in specific situations

**Examples of Situational Weapons**:
- Bombs (need to be in morph ball)
- Screw Attack (need to be spin jumping)
- Pseudo Screw (very specific setup)

### hitsGroup
Does this weapon hit multiple enemies?

```json
"hitsGroup": false
```

**Values**:
- `false`: Hits one enemy per shot
- `true`: Hits all enemies in a group

**Examples of Group Weapons**:
- Power Bomb (explosion hits everything)
- Plasma Beam (pierces through enemies)

**Important**: When `true`, one shot can kill entire groups

### categories
Weapon categories for grouping:

```json
"categories": [
  "Beam",
  "ChargedBeam"
]
```

**Common Categories**:
- `All`: Every weapon
- `Beam`: All beam weapons
- `ChargedBeam`: Charged beams only
- `UnchargedBeam`: Uncharged beams only
- `PowerBombBlast`: Power Bomb explosions
- `SpecialBeamAttack`: Special beam techniques

**Why Categories Matter**: Enemies can have resistances/weaknesses to categories:

```json
// Enemy resistant to all beams
"damageMultipliers": [
  {
    "weapon": "Beam",
    "value": 0.5
  }
]
```

## Complete Examples

### Example 1: Basic Beam Weapon

```json
{
  "id": 1,
  "name": "Power Beam",
  "damage": 20,
  "cooldownFrames": 15,
  "useRequires": [],
  "shotRequires": [],
  "situational": false,
  "hitsGroup": false,
  "categories": ["Beam", "UnchargedBeam"],
  "note": "Basic beam weapon, always available"
}
```

**What This Means**:
- Deals 20 damage
- Fires every 15 frames (~4 shots/second)
- No requirements (always available)
- No ammo cost
- Not situational
- Hits one enemy at a time
- Counts as Beam and UnchargedBeam

### Example 2: Ammo Weapon

```json
{
  "id": 10,
  "name": "Missile",
  "damage": 50,
  "cooldownFrames": 30,
  "useRequires": [
    "Missiles"
  ],
  "shotRequires": [
    {
      "ammo": {
        "type": "Missile",
        "count": 1
      }
    }
  ],
  "situational": false,
  "hitsGroup": false,
  "categories": ["All"],
  "note": "Standard missile weapon"
}
```

**What This Means**:
- Deals 50 damage
- Fires every 30 frames (~2 shots/second)
- Requires Missiles item
- Costs 1 missile per shot
- Not situational
- Hits one enemy

### Example 3: Charged Beam

```json
{
  "id": 20,
  "name": "Charge",
  "damage": 40,
  "cooldownFrames": 60,
  "useRequires": [
    "Charge"
  ],
  "shotRequires": [],
  "situational": false,
  "hitsGroup": false,
  "categories": ["Beam", "ChargedBeam"],
  "note": "Charged shot"
}
```

**What This Means**:
- Deals 40 damage
- Fires every 60 frames (~1 shot/second)
- Requires Charge Beam item
- No ammo cost
- Counts as charged beam category

### Example 4: Area Effect Weapon

```json
{
  "id": 30,
  "name": "Power Bomb",
  "damage": 200,
  "cooldownFrames": 120,
  "useRequires": [
    "Morph Ball",
    "Power Bomb"
  ],
  "shotRequires": [
    {
      "ammo": {
        "type": "PowerBomb",
        "count": 1
      }
    }
  ],
  "situational": true,
  "hitsGroup": true,
  "categories": ["All", "PowerBombBlast"],
  "note": "Hits all enemies in room"
}
```

**What This Means**:
- Deals 200 damage
- Fires every 120 frames (~0.5 shots/second)
- Requires Morph Ball + Power Bomb item
- Costs 1 Power Bomb per use
- Situational (need to be morphed)
- Hits entire groups with one bomb

### Example 5: Situational Weapon

```json
{
  "id": 40,
  "name": "Screw Attack",
  "damage": 60,
  "cooldownFrames": 10,
  "useRequires": [
    "Screw Attack"
  ],
  "shotRequires": [],
  "situational": true,
  "hitsGroup": false,
  "categories": ["SpecialBeamAttack"],
  "note": "Only while spin jumping"
}
```

**What This Means**:
- Deals 60 damage
- Very fast (10 frames between hits)
- Requires Screw Attack item
- No ammo cost
- Situational (only during spin jumps)
- Hits one enemy at a time

## How Weapons Are Used in Logic

### In Enemy Kills

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [
        ["Sidehopper"]
      ]
    }
  }
]
```

**Logic Process**:
1. Find all available weapons (check useRequires)
2. Calculate damage to enemy (base damage * multipliers)
3. Figure out how many shots needed (enemy HP / damage)
4. Check ammo availability (shots * shotRequires)
5. Choose best weapon (most efficient)

### With Explicit Weapons

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [["Boss"]],
      "explicitWeapons": ["Missile", "Super Missile"]
    }
  }
]
```

**Means**: Can ONLY use Missiles or Super Missiles

### With Excluded Weapons

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [["Enemy"]],
      "excludedWeapons": ["Power Bomb"]
    }
  }
]
```

**Means**: Cannot use Power Bombs (maybe room doesn't allow it)

## Damage Calculations

### Example 1: Simple Damage

**Weapon**: Missile (50 damage)
**Enemy**: 100 HP, no multipliers

```
Damage per shot = 50
Shots needed = 100 / 50 = 2 missiles
```

### Example 2: With Weakness

**Weapon**: Missile (50 damage)
**Enemy**: 100 HP, weakness to missiles (2.0x)

```
Damage per shot = 50 * 2.0 = 100
Shots needed = 100 / 100 = 1 missile
```

### Example 3: With Resistance

**Weapon**: Beam (20 damage)
**Enemy**: 100 HP, resistant to beams (0.5x)

```
Damage per shot = 20 * 0.5 = 10
Shots needed = 100 / 10 = 10 shots
```

### Example 4: Multi-Category

**Weapon**: Charge Beam (40 damage)
**Categories**: Beam, ChargedBeam
**Enemy**: Resistant to Beam (0.5x), weak to ChargedBeam (2.0x)

```
Apply most specific category: ChargedBeam
Damage per shot = 40 * 2.0 = 80
```

## Weapon Selection Logic

When multiple weapons work, the logic chooses the **most efficient**:

```
Efficiency = Damage per ammo cost
```

### Example Selection

**Enemy**: 100 HP

**Option A - Missiles**:
- Damage: 50
- Cost: 1 missile per shot
- Shots needed: 2
- Total cost: 2 missiles

**Option B - Super Missiles**:
- Damage: 150
- Cost: 1 super per shot
- Shots needed: 1
- Total cost: 1 super

**Logic Chooses**: Super Missile (fewer shots, less total ammo)

## Cooldown and DPS

### Damage Per Second

```
DPS = (Damage * 60) / CooldownFrames
```

**Example**:
- Damage: 50
- Cooldown: 30 frames

```
DPS = (50 * 60) / 30 = 100 damage/second
```

### Comparing Weapons

**Power Beam**:
- Damage: 20
- Cooldown: 15
- DPS: (20 * 60) / 15 = 80

**Charge Beam**:
- Damage: 40
- Cooldown: 60
- DPS: (40 * 60) / 60 = 40

**Power Beam has higher DPS** despite lower damage per shot

## How to Add a New Weapon

### Step 1: Decide Properties

**Name**: "Plasma Beam"
**Damage**: 60
**Cooldown**: 20 frames
**Requirements**: Plasma Beam item
**Ammo**: None (beam weapon)
**Situational**: No
**Hits Group**: Yes (pierces)

### Step 2: Find Next ID

Check existing weapons:
- ID 50: Last weapon
- Next: **ID 51**

### Step 3: Choose Categories

Is it a beam? Charged? Special?
- Yes, it's a beam
- It's uncharged
- Categories: `Beam`, `UnchargedBeam`

### Step 4: Write the JSON

```json
{
  "id": 51,
  "name": "Plasma Beam",
  "damage": 60,
  "cooldownFrames": 20,
  "useRequires": [
    "Plasma Beam"
  ],
  "shotRequires": [],
  "situational": false,
  "hitsGroup": true,
  "categories": ["Beam", "UnchargedBeam"],
  "note": "Piercing beam that hits multiple enemies"
}
```

### Step 5: Add to weapons Array

```json
{
  "weapons": [
    // ... existing weapons ...
    {
      "id": 51,
      "name": "Plasma Beam",
      // ... rest of weapon data ...
    }
  ]
}
```

### Step 6: Use in Enemy Data

Enemies can now reference it:

```json
"damageMultipliers": [
  {
    "weapon": "Plasma Beam",
    "value": 0.5
  }
]
```

## Weapon Categories in Detail

### All
Every weapon in the game:

```json
"categories": ["All"]
```

**Use For**: Weapons that don't fit other categories

### Beam
All beam-type weapons:

```json
"categories": ["Beam", ...]
```

**Includes**: Power Beam, Wide Beam, Ice Beam, etc.

### ChargedBeam
Only charged beam shots:

```json
"categories": ["Beam", "ChargedBeam"]
```

**Includes**: Charge Beam, Charged Wide, Charged Ice, etc.

### UnchargedBeam
Only uncharged beam shots:

```json
"categories": ["Beam", "UnchargedBeam"]
```

**Includes**: Power Beam, Wide Beam, Ice Beam, etc.

### PowerBombBlast
Power Bomb explosions:

```json
"categories": ["PowerBombBlast"]
```

**Special**: Area-of-effect damage

### SpecialBeamAttack
Special beam attacks:

```json
"categories": ["SpecialBeamAttack"]
```

**Includes**: Screw Attack, Pseudo Screw, etc.

## Common Patterns

### Pattern 1: Free Beam

```json
{
  "name": "Power Beam",
  "damage": 20,
  "useRequires": [],
  "shotRequires": [],
  "categories": ["Beam", "UnchargedBeam"]
}
```

**Use**: Always-available weapons

### Pattern 2: Ammo Weapon

```json
{
  "name": "Missile",
  "damage": 50,
  "useRequires": ["Missiles"],
  "shotRequires": [
    {"ammo": {"type": "Missile", "count": 1}}
  ],
  "categories": ["All"]
}
```

**Use**: Weapons that consume ammo

### Pattern 3: Charged Version

```json
{
  "name": "Charge",
  "damage": 40,
  "useRequires": ["Charge"],
  "shotRequires": [],
  "categories": ["Beam", "ChargedBeam"]
}
```

**Use**: Upgraded beam versions

### Pattern 4: Bomb Weapon

```json
{
  "name": "Bomb",
  "damage": 30,
  "useRequires": ["Morph Ball", "Bombs"],
  "shotRequires": [],
  "situational": true,
  "categories": ["All"]
}
```

**Use**: Situational weapons

## Common Mistakes

### ❌ Mistake 1: Missing shotRequires

```json
// WRONG - missile with no ammo cost
{
  "name": "Missile",
  "useRequires": ["Missiles"],
  "shotRequires": []  // ← Should cost ammo!
}
```

**Fix**: Add ammo cost for ammo weapons

### ❌ Mistake 2: Wrong Categories

```json
// WRONG - charged beam not in ChargedBeam category
{
  "name": "Charge",
  "categories": ["Beam"]  // ← Missing ChargedBeam!
}
```

**Fix**: Include all applicable categories

### ❌ Mistake 3: Duplicate IDs

```json
// WRONG - same ID
{"id": 10, "name": "WeaponA"},
{"id": 10, "name": "WeaponB"}  // ← Duplicate!
```

**Fix**: Each weapon needs unique ID

### ❌ Mistake 4: Unrealistic Cooldown

```json
// WRONG - fires 60 times per second
{
  "name": "Super Rapid Beam",
  "cooldownFrames": 1  // ← Too fast!
}
```

**Fix**: Use reasonable cooldowns (10-60 frames typical)

## Testing Your Weapon

### Test Requirements

```json
"useRequires": ["Your New Item"]
```

Verify weapon is only available when you have the item

### Test Damage

Kill test enemy:
- Enemy: 100 HP
- Weapon: 50 damage
- Expected: 2 shots

### Test Ammo Cost

```json
"shotRequires": [
  {"ammo": {"type": "Missile", "count": 2}}
]
```

Verify ammo is consumed correctly

### Test in Enemy Kill

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [["TestEnemy"]]
    }
  }
]
```

Verify logic can select your weapon

## Quick Reference

### Required Fields

- ✅ `id`: Unique number
- ✅ `name`: Weapon name
- ✅ `damage`: Base damage
- ✅ `cooldownFrames`: Frames between shots
- ✅ `useRequires`: Requirements to use
- ✅ `situational`: Situational flag
- ✅ `hitsGroup`: Group hit flag
- ✅ `categories`: Weapon categories

### Optional Fields

- `shotRequires`: Per-shot requirements
- `note`: Description

### Common Cooldowns

| Speed | Frames | Shots/Second |
|-------|--------|--------------|
| Very Fast | 10 | 6.0 |
| Fast | 15 | 4.0 |
| Normal | 30 | 2.0 |
| Slow | 60 | 1.0 |
| Very Slow | 120 | 0.5 |

### Damage Ranges

| Power | Damage |
|-------|--------|
| Weak | 10-20 |
| Normal | 30-50 |
| Strong | 60-100 |
| Very Strong | 100-150 |
| Extreme | 150+ |

## Next Steps

- Read [Enemies-Schema.md](Enemies-Schema.md) for how weapons interact with enemies
- Check [Requirements-Schema.md](Requirements-Schema.md) for enemyKill usage
- See [Logic-System-Overview.md](Logic-System-Overview.md) for weapon selection logic