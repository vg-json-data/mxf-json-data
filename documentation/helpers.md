# Helpers Schema Guide

## What Are Helpers?

Helpers are **reusable requirement sets** that you can reference by name instead of writing the same requirements over and over.

Think of helpers as **shortcuts** or **macros** for common requirement patterns.

## Schema Location

`schema/mxf-helpers.schema.json`

## Why Use Helpers?

### Without Helpers (Repetitive)

```json
// Room 1
{
  "requires": [
    "Morph Ball",
    {"or": ["Bombs", "Power Bomb"]}
  ]
}

// Room 2
{
  "requires": [
    "Morph Ball",
    {"or": ["Bombs", "Power Bomb"]}
  ]
}

// Room 3
{
  "requires": [
    "Morph Ball",
    {"or": ["Bombs", "Power Bomb"]}
  ]
}
```

### With Helpers (Clean)

Define once:
```json
{
  "name": "canUseMorphBombs",
  "requires": [
    "Morph Ball",
    {"or": ["Bombs", "Power Bomb"]}
  ]
}
```

Use everywhere:
```json
// Room 1
{"requires": ["canUseMorphBombs"]}

// Room 2
{"requires": ["canUseMorphBombs"]}

// Room 3
{"requires": ["canUseMorphBombs"]}
```

**Benefits**:
- ✅ Less typing
- ✅ Easier to update (change in one place)
- ✅ More readable
- ✅ Consistent across rooms

## Basic Structure

```json
{
  "helperCategories": [
    {
      "name": "Basic Actions",
      "description": "Common combinations of items needed for basic actions",
      "helpers": [
        {
          "name": "canUseMorphBombs",
          "requires": [
            "Morph Ball",
            {"or": ["Bombs", "Power Bomb"]}
          ],
          "note": "Can bomb things while in morph ball"
        }
      ]
    }
  ]
}
```

## Helper Categories

Helpers are grouped into categories for organization:

### Common Categories

**Basic Actions**:
- Using morph bombs
- Opening doors
- Breaking blocks

**Movement**:
- Getting up high
- Getting through water
- Flying/jumping combinations

**Combat**:
- Damaging enemies
- Killing specific bosses
- Surviving hazards

**Utility**:
- Saving/refilling
- Map station access

## Helper Object Structure

```json
{
  "name": "helperName",
  "requires": [...],
  "note": "Description of what this helper represents"
}
```

### name
The helper's name (used in requirements):

```json
"name": "canUseMorphBombs"
```

**Naming Convention**: Usually starts with "can" like tech

**Important**: This is what you reference in room requirements:
```json
"requires": ["canUseMorphBombs"]
```

### requires
The actual requirements this helper represents:

```json
"requires": [
  "Morph Ball",
  {"or": ["Bombs", "Power Bomb"]}
]
```

**Can be ANY valid requirement structure** - see Requirements-Schema.md

### note
Description explaining what the helper is for:

```json
"note": "Ability to bomb things while in morph ball form"
```

or multiple lines:

```json
"note": [
  "Ability to bomb things while in morph ball.",
  "Requires either regular Bombs or Power Bombs."
]
```

## Complete Examples

### Example 1: Simple Item Combination

```json
{
  "name": "canUseMorphBombs",
  "requires": [
    "Morph Ball",
    {
      "or": ["Bombs", "Power Bomb"]
    }
  ],
  "note": "Can bomb things in morph ball form"
}
```

**What This Represents**:
- Must have Morph Ball
- Must have EITHER Bombs OR Power Bombs
- Common pattern for breaking bomb blocks

**Usage**:
```json
"requires": ["canUseMorphBombs"]
```

### Example 2: Movement Helper

```json
{
  "name": "canFly",
  "requires": [
    {
      "or": [
        "Space Jump",
        ["Hi-Jump", "canWallJump"],
        ["canIBJ", "Morph Ball", "Bombs"]
      ]
    }
  ],
  "note": "Can reach high places by any means"
}
```

**What This Represents**:
- Option 1: Space Jump
- Option 2: Hi-Jump + Wall Jump tech
- Option 3: Infinite Bomb Jump

**Usage**:
```json
"requires": ["canFly"]
```

### Example 3: Combat Helper

```json
{
  "name": "canKillEnemies",
  "requires": [
    {
      "or": [
        "Charge Beam",
        "Missiles",
        "Super Missiles"
      ]
    }
  ],
  "note": "Can kill basic enemies with some weapon"
}
```

**What This Represents**:
- Must have at least one offensive weapon
- For rooms where any weapon works

**Usage**:
```json
"requires": ["canKillEnemies"]
```

### Example 4: Complex Helper

```json
{
  "name": "canNavigateHeatedRooms",
  "requires": [
    {
      "or": [
        "Varia Suit",
        "Gravity",
        {
          "resourceCapacity": [
            {
              "type": "Energy",
              "count": 299
            }
          ]
        }
      ]
    }
  ],
  "note": [
    "Can survive heated rooms.",
    "Either have heat protection or enough energy to tank damage."
  ]
}
```

**What This Represents**:
- Option 1: Varia Suit (heat protection)
- Option 2: Gravity Suit (better heat protection)
- Option 3: Lots of energy (tank the damage)

**Usage**:
```json
"requires": [
  "canNavigateHeatedRooms",
  {"heatFrames": 600}
]
```

## Complete Helper Category Example

```json
{
  "name": "Basic Actions",
  "description": "Common item combinations for basic actions",
  "helpers": [
    {
      "name": "canUseMorphBombs",
      "requires": [
        "Morph Ball",
        {"or": ["Bombs", "Power Bomb"]}
      ],
      "note": "Can bomb things in morph ball"
    },
    {
      "name": "canOpenRedDoors",
      "requires": [
        {
          "or": [
            {"ammo": {"type": "Missile", "count": 1}},
            {"ammo": {"type": "Super", "count": 1}}
          ]
        }
      ],
      "note": "Can open red doors with missiles or supers"
    },
    {
      "name": "canBreakBlocks",
      "requires": [
        {
          "or": [
            ["Morph Ball", "Bombs"],
            ["Morph Ball", "Power Bomb"],
            "Speed Booster",
            "Screw Attack"
          ]
        }
      ],
      "note": "Can break destructible blocks by any means"
    }
  ]
}
```

## How Helpers Are Used

### In Room Requirements

Just reference the helper name:

```json
{
  "link": [1, 2],
  "name": "Bomb Through Blocks",
  "requires": [
    "canUseMorphBombs"
  ]
}
```

### Combined with Other Requirements

```json
{
  "link": [1, 2],
  "name": "Bomb Through Heated Room",
  "requires": [
    "canUseMorphBombs",
    {"heatFrames": 300}
  ]
}
```

### In OR Conditions

```json
{
  "link": [1, 2],
  "name": "Multiple Paths",
  "requires": [
    {
      "or": [
        "canFly",           // Helper
        "canUseMorphBombs", // Another helper
        "Space Jump"        // Direct item
      ]
    }
  ]
}
```

### Nested with Complex Logic

```json
{
  "link": [1, 2],
  "name": "Complex Path",
  "requires": [
    "canNavigateHeatedRooms",
    {
      "or": [
        ["canFly", {"heatFrames": 200}],
        ["canUseMorphBombs", {"heatFrames": 400}]
      ]
    }
  ]
}
```

## Common Helper Patterns

### Pattern 1: Item Alternatives

When multiple items can do the same thing:

```json
{
  "name": "canOpenPowerBombDoors",
  "requires": [
    "Power Bomb",
    {
      "ammo": {
        "type": "PowerBomb",
        "count": 1
      }
    }
  ]
}
```

### Pattern 2: Item + Tech Combinations

When tech makes items more useful:

```json
{
  "name": "canShortChargeRun",
  "requires": [
    "Speed Booster",
    "canShortCharge",
    {
      "canShineCharge": {
        "usedTiles": 20,
        "openEnd": 1
      }
    }
  ]
}
```

### Pattern 3: Survivability Checks

For hazard navigation:

```json
{
  "name": "canSurviveLava",
  "requires": [
    {
      "or": [
        "Gravity",
        {
          "and": [
            "Varia Suit",
            {
              "resourceCapacity": [
                {"type": "Energy", "count": 199}
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### Pattern 4: Complex Movement

Multiple ways to reach a location:

```json
{
  "name": "canReachHighLedge",
  "requires": [
    {
      "or": [
        "Hi-Jump",
        "Space Jump",
        ["canWallJump", "canPreciseWallJump"],
        ["canIBJ", "Morph Ball", "Bombs"],
        ["canSpringBallJump", "Spring Ball"]
      ]
    }
  ]
}
```

## How to Add a New Helper

### Step 1: Identify the Pattern

Notice you're writing the same requirements multiple times:

```json
// Used in 10 different rooms:
"requires": [
  "Morph Ball",
  "Bombs",
  "canHBJ"
]
```

### Step 2: Choose a Name

Pick a descriptive name:
- `canHorizontalBombJump`
- `canHBJAcrossGaps`

Let's use: `canHBJAcrossGaps`

### Step 3: Choose a Category

Where does it fit?
- Basic Actions?
- Movement?
- Morph Ball Techniques?

Let's say: "Movement"

### Step 4: Write the Helper

```json
{
  "name": "canHBJAcrossGaps",
  "requires": [
    "Morph Ball",
    "Bombs",
    "canHBJ"
  ],
  "note": "Can use horizontal bomb jumps to cross gaps"
}
```

### Step 5: Add to Category

```json
{
  "name": "Movement",
  "description": "Movement techniques and combinations",
  "helpers": [
    // ... existing helpers ...
    {
      "name": "canHBJAcrossGaps",
      "requires": [
        "Morph Ball",
        "Bombs",
        "canHBJ"
      ],
      "note": "Can use horizontal bomb jumps to cross gaps"
    }
  ]
}
```

### Step 6: Use in Rooms

Replace the repeated requirements:

**Before**:
```json
"requires": [
  "Morph Ball",
  "Bombs",
  "canHBJ"
]
```

**After**:
```json
"requires": ["canHBJAcrossGaps"]
```

## When to Create a Helper

### ✅ Create a Helper When:

1. **Pattern repeats 3+ times**
   - Same requirements in multiple rooms

2. **Complex but common**
   - OR conditions with many options

3. **Logical grouping**
   - Items that always go together

4. **Readability improvement**
   - Makes requirements clearer

### ❌ Don't Create a Helper When:

1. **Used only once**
   - Just write requirements directly

2. **Too specific**
   - Only applies to one unique situation

3. **Too vague**
   - Doesn't have clear meaning

4. **Already exists**
   - Check existing helpers first

## Helper Evaluation

### How It Works

When the evaluator sees a helper name:

```javascript
// In room
"requires": ["canUseMorphBombs"]

// Evaluator looks up helper
helper = getHelper("canUseMorphBombs");

// Evaluates helper's requirements
evaluate(helper.requires, state);
// Which is:
evaluate([
  "Morph Ball",
  {"or": ["Bombs", "Power Bomb"]}
], state);
```

### Recursive Evaluation

Helpers can reference other helpers:

```json
{
  "name": "canAccessEarlyItems",
  "requires": [
    "canUseMorphBombs",    // References another helper
    "canOpenRedDoors",     // References another helper
    "canNavigateHeatedRooms" // References another helper
  ]
}
```

## Common Mistakes

### ❌ Mistake 1: Circular References

```json
// WRONG
{
  "name": "helperA",
  "requires": ["helperB"]
},
{
  "name": "helperB",
  "requires": ["helperA"]  // ← Circular!
}
```

**Fix**: Make sure helpers don't reference each other in a loop

### ❌ Mistake 2: Too Specific

```json
// WRONG - only useful in one room
{
  "name": "canGetItemInRoom42Node3",
  "requires": [
    "Morph Ball",
    "Bombs",
    {"heatFrames": 123}
  ]
}
```

**Fix**: Make helpers general enough to be reusable

### ❌ Mistake 3: Unclear Name

```json
// WRONG - what does this mean?
{
  "name": "canDoThing",
  "requires": [...]
}
```

**Fix**: Use descriptive names that explain what the helper does

### ❌ Mistake 4: Duplicate Functionality

```json
// WRONG - already have canUseMorphBombs
{
  "name": "canBombStuff",
  "requires": [
    "Morph Ball",
    {"or": ["Bombs", "Power Bomb"]}
  ]
}
```

**Fix**: Check existing helpers before creating new ones

## Testing Your Helper

### Step 1: Add to helpers.json

```json
{
  "name": "canMyNewHelper",
  "requires": [
    "Item A",
    {"or": ["Item B", "Item C"]}
  ],
  "note": "Description"
}
```

### Step 2: Use in a Room

```json
{
  "link": [1, 2],
  "name": "Test Strat",
  "requires": ["canMyNewHelper"]
}
```

### Step 3: Test Without Requirements

State: No items
- Path should be unreachable

### Step 4: Test With Requirements

State: Has Item A and Item B
- Path should be reachable

State: Has Item A and Item C
- Path should be reachable

State: Has only Item A
- Path should be unreachable (missing B or C)

## Best Practices

### 1. Clear Names

Use names that explain the purpose:
- ✅ `canNavigateUnderwater`
- ❌ `waterHelper`

### 2. Good Documentation

Explain what the helper is for:

```json
{
  "name": "canOpenAllDoors",
  "note": [
    "Can open any color door.",
    "Requires missiles for red doors,",
    "supers for green doors,",
    "and power bombs for yellow doors."
  ]
}
```

### 3. Logical Grouping

Group helpers by theme:
- Movement helpers in "Movement" category
- Combat helpers in "Combat" category

### 4. Consistent Patterns

Follow existing naming patterns:
- `canDoX` for abilities
- `hasX` for possession
- `needsX` for requirements

## Quick Reference

### Common Helper Patterns

| Pattern | Example | Use |
|---------|---------|-----|
| Item alternatives | `{"or": ["A", "B"]}` | Multiple items work |
| Item + tech | `["Item", "canTech"]` | Tech enhances item |
| Survivability | `{"or": ["Protection", "HighHP"]}` | Hazard navigation |
| Complex movement | `{"or": [multiple paths]}` | Multiple ways to reach |

### Helper Naming Guide

| Prefix | Meaning | Example |
|--------|---------|---------|
| `can` | Ability to do something | `canUseMorphBombs` |
| `has` | Possession of something | `hasWeapons` |
| `needs` | Requirement for something | `needsHeatProtection` |

## Next Steps

- Read [Requirements-Schema.md](Requirements-Schema.md) to understand requirement syntax
- Check [Tech-Schema.md](Tech-Schema.md) to combine helpers with tech
- See [Room-Schema.md](Room-Schema.md) to use helpers in rooms
- Review [Logic-System-Overview.md](Logic-System-Overview.md) for how helpers fit into logic