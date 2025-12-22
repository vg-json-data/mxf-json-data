# Items Schema Guide

## What Are Items?

Items define **what Samus can collect** throughout the game. This includes:
- **Upgrades**: Morph Ball, Bombs, Speed Booster, etc.
- **Expansions**: Energy Tanks, Missile packs, Power Bomb packs
- **Progressive Upgrades**: Items that upgrade themselves when collected multiple times
- **Special Items**: Aux Power Cells, Reserve-X items

## Schema Location

`schema/mxf-items.schema.json`

## Basic Structure

```json
{
  "$schema": "../schema/mxf-items.schema.json",
  "startingRoom": "Revival Room",
  "startingNode": 1,
  "startingItems": ["Ship"],
  "startingFlags": [],
  "startingLocks": [],
  "startingResources": [...],
  "upgradeItems": [...],
  "reserveXItems": [...],
  "expansionItems": [...],
  "progressiveUpgrades": [...],
  "gameFlags": [...]
}
```

## Starting Conditions

### startingRoom
The room where Samus begins:

```json
"startingRoom": "Revival Room"
```

**Note**: Must match a room's `name` field exactly

### startingNode
The node within that room where Samus starts:

```json
"startingNode": 1
```

**Note**: Must be a valid node ID in the starting room

### startingItems
Items Samus has at the beginning:

```json
"startingItems": [
  "Ship",
  "Power Suit"
]
```

**Common Starting Items**:
- `Ship`: The ship (always included)
- `Power Suit`: Basic suit

**Why These Matter**: These items are available from the very first moment, affecting what locations are initially reachable.

### startingFlags
Game flags that are set at the start:

```json
"startingFlags": [
  "f_IntroComplete"
]
```

**Use For**: Story flags that should be set from the beginning

### startingLocks
Doors/locks that start unlocked:

```json
"startingLocks": [
  "IntroSequenceDoor"
]
```

**Use For**: Doors that should be open from the start

### startingResources
How much energy and ammo Samus starts with:

```json
"startingResources": [
  {
    "resource": "regularEnergy",
    "maxAmount": 99
  },
  {
    "resource": "missile",
    "maxAmount": 0
  },
  {
    "resource": "super",
    "maxAmount": 0
  },
  {
    "resource": "powerBomb",
    "maxAmount": 0
  }
]
```

**Resource Types**:
- `regularEnergy`: Health (starts at 99)
- `reserveEnergy`: Reserve tanks
- `missile`: Missile capacity
- `super`: Super missile capacity
- `powerBomb`: Power bomb capacity

## Upgrade Items

These are the **main progression items** - things like Morph Ball, Bombs, Speed Booster, etc.

### Structure

```json
"upgradeItems": [
  {
    "name": "Morph Ball",
    "wram": "0x09A4",
    "data": {
      "itemType": "equipment",
      "messageBoxId": 1,
      "itemBit": "0x0004",
      "bossBit": "0x0000",
      "speedXY": "0x0000",
      "gatherX": "0x0000",
      "gatherY": "0x0000",
      "health": "0x0000",
      "damage": "0x0000"
    }
  }
]
```

### Fields Explained

#### name
The item's display name:

```json
"name": "Morph Ball"
```

**Important**: This is what you reference in requirements and room definitions

#### wram
Memory address where this item is stored:

```json
"wram": "0x09A4"
```

**Format**: Hexadecimal memory address (advanced - usually don't need to change)

#### data.itemType
What kind of item this is:

```json
"itemType": "equipment"
```

**Types**:
- `equipment`: Main progression items (Morph Ball, Bombs, etc.)
- `beam`: Beam weapons (Charge Beam, Wide Beam, etc.)
- `energytank`: Energy tanks
- `missile`: Missile expansions
- `powerbomb`: Power bomb expansions
- `xreserve`: Reserve-X items

#### data.messageBoxId
Which text box appears when you collect it:

```json
"messageBoxId": 1
```

**Note**: Different items can share message boxes (e.g., all E-Tanks use the same one)

#### data.itemBit
Binary flag for this item:

```json
"itemBit": "0x0004"
```

**Format**: Hexadecimal bit flag (advanced)

## Example: Complete Upgrade Item

```json
{
  "name": "Bombs",
  "wram": "0x09A6",
  "data": {
    "itemType": "equipment",
    "messageBoxId": 3,
    "itemBit": "0x1000",
    "bossBit": "0x0000",
    "speedXY": "0x0000",
    "gatherX": "0x0000",
    "gatherY": "0x0000",
    "health": "0x0000",
    "damage": "0x0000"
  }
}
```

**What This Means**:
- Item name: "Bombs"
- When collected, sets the Bombs flag
- Shows message box #3
- No health/damage changes (just gives the ability)

## Reserve-X Items

Special progression items that are **placed one per sector**.

### Structure

```json
"reserveXItems": [
  {
    "name": "Reserve X 1",
    "pcAddress": "0x8F8000",
    "data": {
      "itemType": "xreserve",
      "messageBoxId": 50,
      "itemBit": "0x0001",
      "bossBit": "0x0000",
      "speedXY": "0x0000",
      "gatherX": "0x0000",
      "gatherY": "0x0000",
      "health": "0x0064",
      "damage": "0x0000"
    }
  }
]
```

### Key Differences from Regular Items

- **One per sector**: Randomizer ensures one Reserve-X per area (MDK, SRX, TRO, etc.)
- **Not vanilla locations**: Can be placed in any standard item location
- **Special placement rules**: See Item-Placer.md for details

### health Field

```json
"health": "0x0064"
```

This is in hexadecimal. `0x0064` = 100 in decimal, meaning +100 energy.

## Expansion Items

Items that **increase your capacity** for energy or ammo.

### Structure

```json
"expansionItems": [
  {
    "name": "ETank",
    "data": "0x8F8000",
    "resource": "regularEnergy",
    "resourceAmount": 100
  },
  {
    "name": "Missile",
    "data": "0x8F8010",
    "resource": "missile",
    "resourceAmount": 5
  },
  {
    "name": "PowerBomb",
    "data": "0x8F8020",
    "resource": "powerBomb",
    "resourceAmount": 5
  }
]
```

### Fields Explained

#### name
Display name:

```json
"name": "ETank"
```

**Standard Names**:
- `ETank`: Energy Tank
- `Missile`: Missile expansion
- `PowerBomb`: Power Bomb expansion

#### data
Memory address:

```json
"data": "0x8F8000"
```

**Note**: Each expansion type needs a unique address

#### resource
Which resource this expands:

```json
"resource": "regularEnergy"
```

**Options**:
- `regularEnergy`: Health
- `reserveXEnergy`: Reserve energy (special)
- `reserveXAmmo`: Reserve ammo (special)
- `missile`: Missile capacity
- `powerBomb`: Power Bomb capacity

#### resourceAmount
How much it increases:

```json
"resourceAmount": 100
```

**Common Values**:
- Energy Tank: `100`
- Missile: `5`
- Power Bomb: `5`

## Progressive Upgrades

Items that **upgrade themselves** when collected multiple times. For example, collecting "Progressive Speed Booster" twice gives you both Speed Booster levels.

### Structure

```json
"progressiveUpgrades": [
  {
    "name": "Progressive Speed Booster",
    "pcAddress": "0x8F9000",
    "value": "0x01"
  },
  {
    "name": "Progressive Missile Upgrade",
    "pcAddress": "0x8F9010",
    "value": "0x01"
  }
]
```

### How Progressive Items Work

```
Collect "Progressive Speed Booster" #1
→ Unlocks "Speed Booster Level 1"

Collect "Progressive Speed Booster" #2
→ Unlocks "Speed Booster Level 2"
```

### Fields Explained

#### name
Display name:

```json
"name": "Progressive Speed Booster"
```

**Pattern**: Usually "Progressive [Item Name]"

#### pcAddress
Memory address:

```json
"pcAddress": "0x8F9000"
```

#### value
The value to store:

```json
"value": "0x01"
```

**Note**: This increments each time you collect the progressive item

## Game Flags

Named flags that can be set during gameplay.

### Structure

```json
"gameFlags": [
  "f_DefeatedKraid",
  "f_DefeatedRidley",
  "f_DefeatedPhantoon",
  "f_DefeatedDraygon",
  "f_ZebesAwake",
  "f_MaridiaOpen"
]
```

### Naming Convention

Flags usually start with `f_` to indicate they're flags:

```
f_DefeatedKraid
f_OpenedSecretPassage
f_ActivatedPowerGrid
```

### How Flags Are Used

**In Requirements**:
```json
"requires": [
  "f_DefeatedKraid"
]
```

**In Locks**:
```json
"lock": ["f_DefeatedKraid"]
```

**Set by Strats**:
```json
"setsFlags": ["f_DefeatedKraid"]
```

## Complete Example File

Here's what a complete items.json might look like:

```json
{
  "$schema": "../schema/mxf-items.schema.json",
  
  "startingRoom": "Revival Room",
  "startingNode": 1,
  
  "startingItems": [
    "Ship",
    "Power Suit"
  ],
  
  "startingFlags": [],
  "startingLocks": [],
  
  "startingResources": [
    {
      "resource": "regularEnergy",
      "maxAmount": 99
    },
    {
      "resource": "missile",
      "maxAmount": 0
    },
    {
      "resource": "powerBomb",
      "maxAmount": 0
    }
  ],
  
  "upgradeItems": [
    {
      "name": "Morph Ball",
      "wram": "0x09A4",
      "data": {
        "itemType": "equipment",
        "messageBoxId": 1,
        "itemBit": "0x0004",
        "bossBit": "0x0000",
        "speedXY": "0x0000",
        "gatherX": "0x0000",
        "gatherY": "0x0000",
        "health": "0x0000",
        "damage": "0x0000"
      }
    },
    {
      "name": "Bombs",
      "wram": "0x09A6",
      "data": {
        "itemType": "equipment",
        "messageBoxId": 3,
        "itemBit": "0x1000",
        "bossBit": "0x0000",
        "speedXY": "0x0000",
        "gatherX": "0x0000",
        "gatherY": "0x0000",
        "health": "0x0000",
        "damage": "0x0000"
      }
    },
    {
      "name": "Speed Booster",
      "wram": "0x09A8",
      "data": {
        "itemType": "equipment",
        "messageBoxId": 5,
        "itemBit": "0x2000",
        "bossBit": "0x0000",
        "speedXY": "0x0000",
        "gatherX": "0x0000",
        "gatherY": "0x0000",
        "health": "0x0000",
        "damage": "0x0000"
      }
    }
  ],
  
  "reserveXItems": [
    {
      "name": "Reserve X 1",
      "pcAddress": "0x8F8000",
      "data": {
        "itemType": "xreserve",
        "messageBoxId": 50,
        "itemBit": "0x0001",
        "bossBit": "0x0000",
        "speedXY": "0x0000",
        "gatherX": "0x0000",
        "gatherY": "0x0000",
        "health": "0x0064",
        "damage": "0x0000"
      }
    }
  ],
  
  "expansionItems": [
    {
      "name": "ETank",
      "data": "0x8F8100",
      "resource": "regularEnergy",
      "resourceAmount": 100
    },
    {
      "name": "Missile",
      "data": "0x8F8110",
      "resource": "missile",
      "resourceAmount": 5
    },
    {
      "name": "PowerBomb",
      "data": "0x8F8120",
      "resource": "powerBomb",
      "resourceAmount": 5
    }
  ],
  
  "progressiveUpgrades": [
    {
      "name": "Progressive Speed Booster",
      "pcAddress": "0x8F9000",
      "value": "0x01"
    }
  ],
  
  "gameFlags": [
    "f_DefeatedKraid",
    "f_DefeatedRidley",
    "f_ZebesAwake"
  ]
}
```

## How to Add a New Item

### Adding a Regular Upgrade

**Step 1**: Decide what the item is
- Name: "Hi-Jump"
- Type: equipment

**Step 2**: Find an unused memory address
- Check existing items
- Use next available: e.g., "0x09AA"

**Step 3**: Choose a message box
- Same as similar items (e.g., other suit upgrades)

**Step 4**: Write the JSON

```json
{
  "name": "Hi-Jump",
  "wram": "0x09AA",
  "data": {
    "itemType": "equipment",
    "messageBoxId": 7,
    "itemBit": "0x0100",
    "bossBit": "0x0000",
    "speedXY": "0x0000",
    "gatherX": "0x0000",
    "gatherY": "0x0000",
    "health": "0x0000",
    "damage": "0x0000"
  }
}
```

**Step 5**: Add to upgradeItems array

### Adding an Expansion Item

**Step 1**: Decide the details
- Name: "Super Missile"
- Resource type: "super"
- Amount: 5 per pack

**Step 2**: Write the JSON

```json
{
  "name": "Super",
  "data": "0x8F8130",
  "resource": "super",
  "resourceAmount": 5
}
```

**Step 3**: Add to expansionItems array

### Adding a Progressive Item

**Step 1**: Define the progression
- "Progressive Charge Beam"
- Level 1: Charge Beam
- Level 2: Ice Beam
- Level 3: Wave Beam

**Step 2**: Write the JSON

```json
{
  "name": "Progressive Charge Beam",
  "pcAddress": "0x8F9020",
  "value": "0x01"
}
```

**Step 3**: Add to progressiveUpgrades array

**Step 4**: Define what each level unlocks (in logic/code)

## Common Patterns

### Pattern 1: Suit Upgrades

```json
{
  "name": "Varia Suit",
  "wram": "0x09A2",
  "data": {
    "itemType": "equipment",
    "messageBoxId": 6,
    "itemBit": "0x0001",
    "bossBit": "0x0000",
    "speedXY": "0x0000",
    "gatherX": "0x0000",
    "gatherY": "0x0000",
    "health": "0x0000",
    "damage": "0x0000"
  }
}
```

**Pattern**: No health/damage, just sets a flag

### Pattern 2: Beam Weapons

```json
{
  "name": "Wide Beam",
  "wram": "0x09A8",
  "data": {
    "itemType": "beam",
    "messageBoxId": 10,
    "itemBit": "0x0008",
    "bossBit": "0x0000",
    "speedXY": "0x0000",
    "gatherX": "0x0000",
    "gatherY": "0x0000",
    "health": "0x0000",
    "damage": "0x0014"
  }
}
```

**Pattern**: Sets damage value for weapon strength

### Pattern 3: Items with Energy Bonus

```json
{
  "name": "Aux Power Cell 1",
  "wram": "0x09B0",
  "data": {
    "itemType": "equipment",
    "messageBoxId": 20,
    "itemBit": "0x0001",
    "bossBit": "0x0000",
    "speedXY": "0x0000",
    "gatherX": "0x0000",
    "gatherY": "0x0000",
    "health": "0x0064",
    "damage": "0x0000"
  }
}
```

**Pattern**: Sets health field (0x0064 = 100 energy)

## Memory Address Guide

### What Are Memory Addresses?

Memory addresses tell the game **where to store information** about items in the console's memory.

**Format**: `0xABCD` (hexadecimal)

### Why They Matter

Each item needs a unique address so the game can track:
- Whether you've collected it
- What abilities you have
- How much ammo you carry

### Finding Available Addresses

**For Upgrade Items (wram)**:
1. Look at existing items
2. Find the pattern (e.g., 0x09A4, 0x09A6, 0x09A8)
3. Use the next value (0x09AA)

**For Expansion Items (data)**:
1. Check expansion items
2. Increment by 0x10 (16 in decimal)
3. Example: 0x8F8000, 0x8F8010, 0x8F8020

### Don't Worry Too Much

For most contributions, you can:
1. Copy an existing item
2. Change the name
3. The dev team will assign proper addresses later

## Testing Your Items

### Verify in Game

1. Add your item to the items.json
2. Place it in a room
3. Collect it in-game
4. Verify it appears in your inventory
5. Test that it enables the correct abilities

### Check Requirements

If your item should unlock new areas:

```json
"requires": [
  "Your New Item"
]
```

Test that these requirements work correctly.

## Common Mistakes

### ❌ Mistake 1: Duplicate Names

```json
// WRONG - two items with same name
{"name": "Missile"},
{"name": "Missile"}
```

**Fix**: Use unique names or use expansionItems for multiple copies

### ❌ Mistake 2: Duplicate Addresses

```json
// WRONG - same address
{"name": "Item A", "wram": "0x09A4"},
{"name": "Item B", "wram": "0x09A4"}  // ← Same address!
```

**Fix**: Each item needs a unique address

### ❌ Mistake 3: Wrong Item Type

```json
// WRONG - E-Tank as equipment
{
  "name": "ETank",
  "data": {
    "itemType": "equipment"  // ← Should be "energytank"
  }
}
```

**Fix**: Use correct itemType for each item category

### ❌ Mistake 4: Missing from startingResources

```json
// WRONG - item exists but resource not initialized
"expansionItems": [
  {"name": "Super", "resource": "super", "resourceAmount": 5}
],
"startingResources": [
  // ← Missing "super" resource!
]
```

**Fix**: Add all resource types to startingResources (even if starting at 0)

## Quick Reference

### Item Types

| Type | Use For | Example |
|------|---------|---------|
| `equipment` | Suit/movement upgrades | Morph Ball, Hi-Jump, Varia Suit |
| `beam` | Beam weapons | Charge Beam, Wide Beam, Ice Beam |
| `energytank` | Energy tanks | E-Tank |
| `missile` | Missile expansions | Missile pack |
| `powerbomb` | Power Bomb expansions | Power Bomb pack |
| `xreserve` | Reserve-X items | Reserve X 1-7 |

### Resource Types

| Resource | Description | Starting Max |
|----------|-------------|--------------|
| `regularEnergy` | Health | 99 |
| `reserveXEnergy` | Reserve energy (special) | 0 |
| `reserveXAmmo` | Reserve ammo (special) | 0 |
| `missile` | Missile capacity | 0 |
| `super` | Super missile capacity | 0 |
| `powerBomb` | Power bomb capacity | 0 |

## Next Steps

- Read [Room-Schema.md](Room-Schema.md) to place items in rooms
- Check [Requirements-Schema.md](Requirements-Schema.md) to use items in logic
- See [Item-Placer.md](Item-Placer.md) to understand placement algorithm