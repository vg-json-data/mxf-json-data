# Tech Schema Guide

## What Is Tech?

"Tech" is short for **technique** - player skills and tricks that can be performed in the game. Examples:
- **Wall Jump**: Jumping off walls to gain height
- **Mockball**: Running while in morph ball form
- **Short Charge**: Getting a shinespark with less runway than normal

Tech is **optional** - players can turn techniques on/off based on their skill level.

## Schema Location

`schema/mxf-tech.schema.json`

## Why Tech Matters

Tech affects **what's logically required** to complete paths:

**Without Wall Jump**:
```json
"requires": ["Hi-Jump"]  // Need Hi-Jump to reach high ledge
```

**With Wall Jump**:
```json
"requires": [
  {
    "or": [
      "Hi-Jump",           // Option 1: Use item
      "canWallJump"        // Option 2: Use tech
    ]
  }
]
```

Players who enable wall jump can reach more locations without needing certain items.

## Basic Structure

```json
{
  "techCategories": [
    {
      "name": "Movement",
      "description": "Basic movement techniques",
      "techs": [
        {
          "id": 1,
          "name": "canWallJump",
          "techRequires": [],
          "otherRequires": [],
          "note": "Jump off walls to gain height"
        }
      ]
    }
  ]
}
```

## Tech Categories

Techs are grouped into categories for organization:

### Common Categories

**Movement**:
- Wall jumping
- Spin jumping
- Horizontal bomb jumping

**Shinespark**:
- Short charging
- Mid-air spark techniques
- Spark conservation

**Morph Ball**:
- Mockball
- Spring ball techniques
- Bomb jumping

**Combat**:
- Quick kills
- Specific enemy strats

**Glitches** (Optional):
- X-ray climbing
- Crystal flash
- Advanced glitches

## Tech Object Structure

```json
{
  "id": 1,
  "name": "canWallJump",
  "techRequires": [],
  "otherRequires": [],
  "note": "Jump off walls repeatedly to gain height",
  "extensionTechs": []
}
```

### id
Unique identifier:

```json
"id": 1
```

**Note**: Must be unique across ALL tech

### name
The tech's name (used in requirements):

```json
"name": "canWallJump"
```

**Naming Convention**: Usually starts with "can" for abilities

**Important**: This exact name is what you use in room requirements:
```json
"requires": ["canWallJump"]
```

### techRequires
Other tech needed to perform this tech:

```json
"techRequires": [
  "canWallJump",
  "canMockball"
]
```

**Example**: Advanced mockball might require basic mockball

**Empty if no prerequisites**:
```json
"techRequires": []
```

### otherRequires
Items or other requirements needed:

```json
"otherRequires": [
  "Hi-Jump",
  "Speed Booster"
]
```

**Example**: Short charge requires Speed Booster

**Empty if no items needed**:
```json
"otherRequires": []
```

### note
Description of what the tech is:

```json
"note": "Jump off walls repeatedly to gain height. Requires timing and positioning."
```

or multiple paragraphs:

```json
"note": [
  "Jump off walls repeatedly to gain height.",
  "Press jump just as Samus touches the wall.",
  "Alternate between left and right walls."
]
```

### extensionTechs
More advanced versions of this tech:

```json
"extensionTechs": [
  {
    "id": 2,
    "name": "canPreciseWallJump",
    "techRequires": ["canWallJump"],
    "otherRequires": [],
    "note": "Very precise wall jumps in tight spaces"
  }
]
```

## Complete Examples

### Example 1: Basic Tech (No Prerequisites)

```json
{
  "id": 1,
  "name": "canWallJump",
  "techRequires": [],
  "otherRequires": [],
  "note": "Jump off walls to gain height"
}
```

**What This Means**:
- Anyone can do it (no prerequisites)
- No items required
- Just player skill

### Example 2: Tech with Item Requirement

```json
{
  "id": 10,
  "name": "canShortCharge",
  "techRequires": [],
  "otherRequires": [
    "Speed Booster"
  ],
  "note": [
    "Charge a shinespark with less runway than normal.",
    "Requires precise timing and stuttering."
  ]
}
```

**What This Means**:
- Requires Speed Booster item
- No other tech prerequisites
- Player technique makes Speed Booster more useful

### Example 3: Tech with Tech Prerequisites

```json
{
  "id": 20,
  "name": "canMockballWithSpringBall",
  "techRequires": [
    "canMockball"
  ],
  "otherRequires": [
    "Spring Ball"
  ],
  "note": "Perform mockball while holding spring ball for a bounce"
}
```

**What This Means**:
- Must know basic mockball
- Must have Spring Ball item
- Combines tech + item for advanced movement

### Example 4: Tech with Extension

```json
{
  "id": 30,
  "name": "canHBJ",
  "techRequires": [],
  "otherRequires": [
    "Morph Ball",
    "Bombs"
  ],
  "note": "Horizontal Bomb Jump - use bombs to move horizontally",
  "extensionTechs": [
    {
      "id": 31,
      "name": "canPreciseHBJ",
      "techRequires": ["canHBJ"],
      "otherRequires": [
        "Morph Ball",
        "Bombs"
      ],
      "note": "Very precise HBJ in tight spaces or with exact positioning"
    }
  ]
}
```

**What This Means**:
- Basic HBJ is one tech
- Precise HBJ is a harder version
- Both require same items, but precise version requires knowing basic HBJ first

## Complete Tech Category Example

```json
{
  "name": "Movement",
  "description": "Basic movement techniques that don't require special items",
  "techs": [
    {
      "id": 1,
      "name": "canWallJump",
      "techRequires": [],
      "otherRequires": [],
      "note": "Jump off walls to gain height"
    },
    {
      "id": 2,
      "name": "canPreciseWallJump",
      "techRequires": ["canWallJump"],
      "otherRequires": [],
      "note": "Wall jump in very tight spaces or with pixel-perfect precision"
    },
    {
      "id": 3,
      "name": "canMockball",
      "techRequires": [],
      "otherRequires": [
        "Morph Ball",
        "Speed Booster"
      ],
      "note": "Roll at high speed while appearing to run"
    }
  ]
}
```

## How Tech Is Used in Requirements

### In Room Strats

```json
{
  "link": [1, 2],
  "name": "Wall Jump Up",
  "requires": [
    "canWallJump"
  ]
}
```

**Means**: This path requires wall jump tech to be enabled

### Combined with Items

```json
{
  "link": [1, 2],
  "name": "Multiple Options",
  "requires": [
    {
      "or": [
        "Hi-Jump",              // Option 1: Use item
        "canWallJump",          // Option 2: Use tech
        "Space Jump"            // Option 3: Different item
      ]
    }
  ]
}
```

**Means**: Player can use EITHER Hi-Jump, wall jump, OR Space Jump

### Tech + Items Together

```json
{
  "link": [1, 2],
  "name": "Short Charge Through",
  "requires": [
    "Speed Booster",
    "canShortCharge"
  ]
}
```

**Means**: Must have Speed Booster AND short charge tech enabled

## Common Tech Patterns

### Pattern 1: Basic Skill

```json
{
  "name": "canSpinJump",
  "techRequires": [],
  "otherRequires": [],
  "note": "Use spin jumps effectively"
}
```

**Use**: Fundamental player skills

### Pattern 2: Item Enhancement

```json
{
  "name": "canShortCharge",
  "techRequires": [],
  "otherRequires": ["Speed Booster"],
  "note": "Charge shinespark with less runway"
}
```

**Use**: Makes items more powerful/flexible

### Pattern 3: Technique Chain

```json
{
  "name": "canAdvancedShinespark",
  "techRequires": [
    "canShinespark",
    "canShortCharge"
  ],
  "otherRequires": ["Speed Booster"],
  "note": "Chain multiple shinesparks together"
}
```

**Use**: Advanced tech building on basics

### Pattern 4: Precise Variant

```json
{
  "name": "canWallJump",
  "note": "Basic wall jumping",
  "extensionTechs": [
    {
      "name": "canPreciseWallJump",
      "techRequires": ["canWallJump"],
      "note": "Wall jump in tight 1-tile gaps"
    }
  ]
}
```

**Use**: Harder version of basic tech

## How to Add New Tech

### Step 1: Decide What It Is

**Name**: canIBJ (Infinite Bomb Jump)
**Description**: Use bombs to gain infinite height
**Requirements**: Morph Ball, Bombs
**Prerequisites**: None (it's a basic tech)

### Step 2: Choose a Category

Where does it fit?
- Movement techniques? → "Movement" category
- Bomb techniques? → "Morph Ball" category
- Shinespark techniques? → "Shinespark" category

Let's say: "Morph Ball" category

### Step 3: Find Next ID

Look at existing tech IDs in that category:
- ID 30: canHBJ
- ID 31: canPreciseHBJ
- Next: **ID 32**

### Step 4: Write the JSON

```json
{
  "id": 32,
  "name": "canIBJ",
  "techRequires": [],
  "otherRequires": [
    "Morph Ball",
    "Bombs"
  ],
  "note": [
    "Infinite Bomb Jump - use bombs to gain infinite height.",
    "Lay bombs while moving upward to maintain altitude.",
    "Very precise timing required."
  ]
}
```

### Step 5: Add to Category

```json
{
  "name": "Morph Ball",
  "description": "Morph ball techniques",
  "techs": [
    // ... existing techs ...
    {
      "id": 32,
      "name": "canIBJ",
      // ... rest of tech ...
    }
  ]
}
```

### Step 6: Use in Rooms

Now you can use it in room requirements:

```json
{
  "link": [1, 2],
  "name": "IBJ to Top",
  "requires": [
    "Morph Ball",
    "Bombs",
    "canIBJ"
  ]
}
```

## Difficulty Tiers

Tech can be organized by difficulty:

### Beginner
- Basic spin jumps
- Simple bomb jumps
- Basic enemy avoidance

### Intermediate  
- Wall jumping
- Mockball
- Short charge

### Advanced
- Precise wall jumps
- Advanced shinespark chains
- Complex bomb jump sequences

### Expert
- Frame-perfect techniques
- TAS-level precision
- Glitch exploitation

**In JSON**:

```json
{
  "name": "Movement - Beginner",
  "description": "Easy techniques anyone can learn",
  "techs": [...]
},
{
  "name": "Movement - Advanced",
  "description": "Difficult techniques requiring practice",
  "techs": [...]
}
```

## Settings Integration

### In User Settings

Players can toggle tech in the UI:

```javascript
userSettings = {
  canWallJump: true,
  canMockball: true,
  canShortCharge: false,  // Not enabled
  ...
}
```

### In Logic

The evaluator checks if tech is enabled:

```javascript
if (requirement === "canWallJump") {
  return settings.canWallJump === true;
}
```

If tech is disabled, paths requiring it become unreachable.

## Common Mistakes

### ❌ Mistake 1: Forgetting otherRequires

```json
// WRONG - short charge needs Speed Booster
{
  "name": "canShortCharge",
  "techRequires": [],
  "otherRequires": []  // ← Missing Speed Booster!
}
```

**Fix**: Add required items to otherRequires

### ❌ Mistake 2: Circular Dependencies

```json
// WRONG - A requires B, B requires A
{
  "name": "techA",
  "techRequires": ["techB"]
},
{
  "name": "techB",
  "techRequires": ["techA"]  // ← Circular!
}
```

**Fix**: Make sure tech dependencies are one-way

### ❌ Mistake 3: Duplicate IDs

```json
// WRONG - same ID
{"id": 10, "name": "techA"},
{"id": 10, "name": "techB"}  // ← Duplicate!
```

**Fix**: Each tech must have unique ID

### ❌ Mistake 4: Typos in techRequires

```json
// WRONG - typo
{
  "name": "advancedTech",
  "techRequires": ["canWalJump"]  // ← Typo! (missing 'l')
}
```

**Fix**: Match names exactly

## Testing Your Tech

### Step 1: Add to tech.json

```json
{
  "id": 50,
  "name": "canMyNewTech",
  "techRequires": [],
  "otherRequires": ["Morph Ball"],
  "note": "Description"
}
```

### Step 2: Use in a Room

```json
{
  "link": [1, 2],
  "name": "Test Path",
  "requires": ["canMyNewTech"]
}
```

### Step 3: Test with Tech Disabled

Settings: `canMyNewTech: false`
- Path should be unreachable

### Step 4: Test with Tech Enabled

Settings: `canMyNewTech: true`
- Path should be reachable (if you have required items)

## Documenting Difficulty

Use notes to communicate difficulty:

```json
{
  "name": "canEasyTech",
  "note": "Simple technique that most players can do"
}
```

```json
{
  "name": "canHardTech",
  "note": [
    "Very difficult technique.",
    "Requires frame-perfect timing.",
    "Recommended only for expert players."
  ]
}
```

## Extension Tech Usage

Extensions create a hierarchy:

```
canWallJump (basic)
  └─ canPreciseWallJump (extension)
      └─ canFramePerfectWallJump (extension of extension)
```

**In JSON**:

```json
{
  "name": "canWallJump",
  "extensionTechs": [
    {
      "name": "canPreciseWallJump",
      "techRequires": ["canWallJump"],
      "extensionTechs": [
        {
          "name": "canFramePerfectWallJump",
          "techRequires": ["canPreciseWallJump"]
        }
      ]
    }
  ]
}
```

## Quick Reference

### Tech Naming Conventions

| Pattern | Example | Use |
|---------|---------|-----|
| `can[Action]` | `canWallJump` | Abilities |
| `canPrecise[Action]` | `canPreciseMockball` | Precise versions |
| `canAdvanced[Action]` | `canAdvancedShinespark` | Advanced versions |
| `can[Specific]` | `canGreenGateGlitch` | Specific tricks |

### Common Tech by Category

**Movement**:
- `canWallJump`
- `canSpinJump`
- `canCeilingClip`

**Shinespark**:
- `canShortCharge`
- `canMidAirSpark`
- `canSparkChain`

**Morph Ball**:
- `canMockball`
- `canHBJ`
- `canIBJ`

**Combat**:
- `canQuickKill`
- `canDodgePrecisely`

## Next Steps

- Read [Requirements-Schema.md](Requirements-Schema.md) to use tech in logic
- Check [Helpers-Schema.md](Helpers-Schema.md) for reusable tech combinations
- See [Room-Schema.md](Room-Schema.md) to apply tech in rooms
- Review [Logic-System-Overview.md](Logic-System-Overview.md) for how tech affects logic