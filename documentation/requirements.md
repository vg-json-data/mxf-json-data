# Requirements Schema Guide

## Overview

The requirements schema defines the logical conditions needed to traverse edges, unlock doors, and perform actions in the game. Requirements are the heart of the randomizer's logic system.

## Schema Location

`schema/mxf-requirements.schema.json`

## Core Concept

Requirements are expressed as **JSON arrays** where elements are implicitly **ANDed** together. More complex logic uses **objects** with operators like `or`, `not`, `and`.

## Basic Structure

```json
{
  "requires": [
    "item_or_tech_name",
    { "operator": [...] },
    { "cost_type": value }
  ]
}
```

## Requirement Types

### 1. String Requirements (Items/Tech)

The simplest form - just check if the player has something:

```json
"requires": [
  "Morph Ball",
  "Bombs",
  "Speed Booster"
]
```

This means: `Morph Ball AND Bombs AND Speed Booster`

**Examples**:
```json
// Single item
"requires": ["Missiles"]

// Multiple items (all required)
"requires": ["Morph Ball", "Bombs", "Power Bomb"]

// Tech requirement
"requires": ["canWallJump"]

// Mix of items and tech
"requires": ["Morph Ball", "canMockball"]
```

### 2. Logical Operators

#### AND Operator (Default)

Elements in an array are implicitly ANDed:

```json
"requires": [
  "Morph Ball",
  "Bombs"
]
```

Explicit AND (rarely needed):

```json
"requires": [
  {
    "and": ["Morph Ball", "Bombs"]
  }
]
```

#### OR Operator

Choose one option from several:

```json
"requires": [
  {
    "or": [
      "Bombs",
      "Power Bomb",
      "Spring Ball"
    ]
  }
]
```

This means: `Bombs OR Power Bomb OR Spring Ball`

**Complex OR Example**:
```json
"requires": [
  "Morph Ball",
  {
    "or": [
      ["Bombs", "Hi-Jump"],
      ["Power Bomb"],
      ["Spring Ball", "Space Jump"]
    ]
  }
]
```

This means: `Morph Ball AND (Bombs + Hi-Jump) OR (Power Bomb) OR (Spring Ball + Space Jump)`

#### NOT Operator

Negate a requirement:

```json
"requires": [
  {
    "not": "Gravity Suit"
  }
]
```

This means: `Must NOT have Gravity Suit`

**Practical Use Case** (gravityless underwater movement):
```json
"requires": [
  {
    "not": "Gravity Suit"
  },
  "canWaterMovement"
]
```

### 3. Resource Requirements

#### Ammo Costs

Require spending ammunition:

```json
{
  "ammo": {
    "type": "Missile",
    "count": 5
  }
}
```

**Types**: `Missile`, `Super`, `PowerBomb`

**Examples**:
```json
// Missile door
"requires": [
  {
    "ammo": {
      "type": "Missile",
      "count": 1
    }
  }
]

// Multiple missiles
"requires": [
  {
    "ammo": {
      "type": "Missile",
      "count": 10
    }
  }
]

// Super missile door
"requires": [
  {
    "ammo": {
      "type": "Super",
      "count": 1
    }
  }
]

// Power Bomb door
"requires": [
  {
    "ammo": {
      "type": "PowerBomb",
      "count": 1
    }
  }
]
```

#### Resource Availability

Check if player has enough resources:

```json
{
  "resourceAvailable": [
    {
      "type": "Energy",
      "count": 100
    },
    {
      "type": "Missile",
      "count": 10
    }
  ]
}
```

**Types**: `Energy`, `RegularEnergy`, `ReserveEnergy`, `Missile`, `Super`, `PowerBomb`

#### Resource Capacity

Check maximum capacity:

```json
{
  "resourceCapacity": [
    {
      "type": "RegularEnergy",
      "count": 199
    }
  ]
}
```

Means: "Must have at least 199 energy capacity (2 E-Tanks)"

### 4. Environmental Damage

#### Heat Frames

Surviving heated rooms:

```json
{
  "heatFrames": 300
}
```

**Calculation**:
- Base damage: `frames * heatDpf * difficultyMultiplier`
- With Varia: `frames * heatDpf * 0.5 * difficultyMultiplier`
- With Varia + Gravity: `frames * heatDpf * 0.25 * difficultyMultiplier`

**Examples**:
```json
// Short heat exposure
"requires": [
  {
    "heatFrames": 60
  }
]

// Long heat exposure - likely needs Varia
"requires": [
  {
    "heatFrames": 600
  }
]

// Alternative: quick route or heat protection
"requires": [
  {
    "or": [
      {"heatFrames": 180},
      ["Varia Suit", {"heatFrames": 600}]
    ]
  }
]
```

#### Acid Frames

Surviving acid:

```json
{
  "acidFrames": 120
}
```

**Calculation**:
- Base damage: `frames * acidDpf * difficultyMultiplier`
- With Gravity: `frames * acidDpf * 0.25 * difficultyMultiplier`

#### Lava Frames

Similar to acid frames:

```json
{
  "lavaFrames": 60
}
```

#### Other Damage Types

```json
// Spike hits
{
  "spikeHits": 3
}

// Thorn hits (weaker spikes)
{
  "thornHits": 5
}

// Hibashi (fire pillar) hits
{
  "hibashiHits": 2
}

// Enemy damage
{
  "enemyDamage": {
    "enemy": "Sidehopper",
    "type": "contact",
    "hits": 2
  }
}
```

### 5. Movement Requirements

#### Shine Charging

```json
{
  "canShineCharge": {
    "usedTiles": 28,
    "openEnd": 1
  }
}
```

**Fields**:
- `usedTiles`: Runway length in tiles
- `openEnd`: Number of open ends (0-2)
- `gentleUpTiles`: Tiles with gentle upward slope
- `gentleDownTiles`: Tiles with gentle downward slope
- `steepUpTiles`: Tiles with steep upward slope
- `steepDownTiles`: Tiles with steep downward slope
- `startingDownTiles`: Downward tiles at start (prevents stutter)

**Example**:
```json
"requires": [
  "Speed Booster",
  {
    "canShineCharge": {
      "usedTiles": 32,
      "openEnd": 1,
      "gentleUpTiles": 4
    }
  }
]
```

#### Getting Blue Speed

```json
{
  "getBlueSpeed": {
    "usedTiles": 35,
    "openEnd": 1
  }
}
```

Similar fields to `canShineCharge`.

#### Shinesparking

```json
{
  "shinespark": {
    "frames": 45
  }
}
```

**Energy Cost**: `max(30, frames)` energy

**Example**:
```json
"requires": [
  "Speed Booster",
  {
    "canShineCharge": {
      "usedTiles": 28,
      "openEnd": 1
    }
  },
  {
    "shinespark": {
      "frames": 60
    }
  }
]
```

### 6. Combat Requirements

#### Enemy Kills

```json
{
  "enemyKill": {
    "enemies": [
      ["Sidehopper"],
      ["Sidehopper", "Sidehopper"]
    ],
    "explicitWeapons": ["Missile", "Super"],
    "excludedWeapons": ["Power Bomb"]
  }
}
```

**Fields**:
- `enemies`: Array of enemy groups (each group can be hit by one AoE attack)
- `explicitWeapons`: Only these weapons can be used (optional)
- `excludedWeapons`: These weapons cannot be used (optional)
- `farmableAmmo`: Ammo types that can be farmed (don't count cost)

**Examples**:
```json
// Kill 3 sidehoppers
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

// Kill with specific weapon
"requires": [
  {
    "enemyKill": {
      "enemies": [["Mother Brain"]],
      "explicitWeapons": ["Charge Beam"]
    }
  }
]

// Kill but can't use Power Bomb
"requires": [
  {
    "enemyKill": {
      "enemies": [["Kraid"]],
      "excludedWeapons": ["Power Bomb"]
    }
  }
]
```

### 7. World State Requirements

#### Obstacles

```json
// Obstacle must be cleared
{
  "obstaclesCleared": ["obstacleA", "obstacleB"]
}

// Obstacle must NOT be cleared
{
  "obstaclesNotCleared": ["obstacleC"]
}
```

#### Door Locks

```json
{
  "doorUnlockedAtNode": 5
}
```

Means: "Door at node 5 must already be unlocked"

#### Notables

```json
{
  "notable": "Zebetite Skip"
}
```

Room-specific techniques that are tracked.

### 8. Utility Requirements

#### Refills

```json
// Full refill
{
  "refill": ["Energy", "Missile", "Super", "PowerBomb"]
}

// Partial refill
{
  "partialRefill": {
    "type": "Energy",
    "limit": 150
  }
}
```

#### Room Reset

```json
{
  "resetRoom": {
    "nodes": [1, 2]
  }
}
```

Means: "Must enter from node 1 or 2"

## Complex Examples

### Example 1: Multiple Paths Through a Room

```json
"requires": [
  "Morph Ball",
  {
    "or": [
      // Easy path: Bombs + Hi-Jump
      [
        "Bombs",
        "Hi-Jump"
      ],
      // Medium path: Spring Ball + some damage
      [
        "Spring Ball",
        {
          "enemyDamage": {
            "enemy": "Sidehopper",
            "type": "contact",
            "hits": 2
          }
        }
      ],
      // Hard path: No items, lots of damage
      [
        {
          "enemyDamage": {
            "enemy": "Sidehopper",
            "type": "contact",
            "hits": 5
          }
        },
        {
          "heatFrames": 300
        }
      ]
    ]
  }
]
```

### Example 2: Heated Boss Room

```json
"requires": [
  "Morph Ball",
  "Bombs",
  {
    "or": [
      // Quick kill with good gear
      [
        "Charge Beam",
        "Varia Suit",
        {
          "enemyKill": {
            "enemies": [["Ridley"]],
            "explicitWeapons": ["Charge Beam", "Missile", "Super"]
          }
        },
        {
          "heatFrames": 600
        }
      ],
      // Slower kill, more heat
      [
        "Varia Suit",
        {
          "enemyKill": {
            "enemies": [["Ridley"]]
          }
        },
        {
          "heatFrames": 1200
        }
      ],
      // Tank the damage
      [
        {
          "resourceCapacity": [
            {
              "type": "Energy",
              "count": 399
            }
          ]
        },
        {
          "enemyKill": {
            "enemies": [["Ridley"]]
          }
        },
        {
          "heatFrames": 1800
        }
      ]
    ]
  }
]
```

### Example 3: Shinespark Chain

```json
"requires": [
  "Speed Booster",
  {
    "canShineCharge": {
      "usedTiles": 32,
      "openEnd": 1
    }
  },
  {
    "shinespark": {
      "frames": 45
    }
  },
  {
    "heatFrames": 180
  },
  {
    "or": [
      "Hi-Jump",
      {
        "canWallJump": true
      }
    ]
  }
]
```

### Example 4: Resource Gating

```json
"requires": [
  {
    "or": [
      // Missile path
      [
        "Missiles",
        {
          "ammo": {
            "type": "Missile",
            "count": 20
          }
        }
      ],
      // Super path (more efficient)
      [
        "Super Missiles",
        {
          "ammo": {
            "type": "Super",
            "count": 5
          }
        }
      ],
      // Power Bomb path
      [
        "Morph Ball",
        "Power Bomb",
        {
          "ammo": {
            "type": "PowerBomb",
            "count": 3
          }
        }
      ]
    ]
  }
]
```

## Best Practices

### 1. Use OR for Alternative Paths

```json
"requires": [
  {
    "or": [
      "Easy path",
      "Medium path",
      "Hard path"
    ]
  }
]
```

### 2. Keep Costs Realistic

- Heat damage should account for Varia/Gravity
- Ammo costs should be reasonable
- Don't require absurd energy amounts

### 3. Document Complex Requirements

Use `note` fields to explain:

```json
{
  "name": "Complex Shinespark",
  "requires": [...],
  "note": "Requires a diagonal shinespark from the lower platform, then quickly morphing to avoid the ceiling"
}
```

### 4. Test Your Requirements

Verify that:
- Requirements are satisfiable
- Costs are affordable
- Alternative paths work correctly

### 5. Avoid Circular Dependencies

Bad:
```json
// Item A requires Item B
"requires": ["ItemB"]

// Item B requires Item A
"requires": ["ItemA"]
```

## Common Patterns

### Pattern: Heated Room Traversal

```json
"requires": [
  {
    "or": [
      ["Varia Suit", {"heatFrames": 500}],
      [{"heatFrames": 200}],  // Quick route
      [{"resourceCapacity": [{"type": "Energy", "count": 299}]}, {"heatFrames": 600}]
    ]
  }
]
```

### Pattern: Boss with Farming

```json
"requires": [
  {
    "enemyKill": {
      "enemies": [["Boss"]],
      "farmableAmmo": ["Missile", "Super"]
    }
  }
]
```

### Pattern: Morph Ball Maze

```json
"requires": [
  "Morph Ball",
  {
    "or": [
      "Bombs",
      "Spring Ball",
      {"enemyDamage": {"enemy": "Sidehopper", "type": "contact", "hits": 3}}
    ]
  }
]
```

## Debugging Requirements

### Check if a requirement is satisfied:

```javascript
const result = evaluator.evaluate(requirements, state);
console.log('Satisfied:', result.satisfied);
console.log('Cost:', result.cost);
```

### Common Issues:

1. **Requirement never satisfied**: Check item names match exactly
2. **Too expensive**: Reduce heat frames or ammo costs
3. **Wrong operator**: Make sure `and`/`or` are used correctly
4. **Missing tech**: Ensure tech is defined in tech.json

## Next Steps

- See [Room-Schema.md](Room-Schema.md) for how requirements are used in strats
- Check [Tech-Schema.md](Tech-Schema.md) for defining new techniques
- Read [Helpers-Schema.md](Helpers-Schema.md) for reusable requirement sets