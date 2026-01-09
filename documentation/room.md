# Room Schema Guide

## Overview

The room schema defines the structure of individual rooms in the game world. Each room contains nodes (locations) and strats (ways to traverse between nodes).

## Schema Location

`schema/mxf-room.schema.json`

## Room Structure

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 42,
  "name": "Room Name",
  "area": "MDK",
  "subarea": "Lower MDK",
  "subsubarea": "Pink Section",
  "roomEnvironments": [...],
  "mapTileMask": [...],
  "nodes": [...],
  "obstacles": [...],
  "enemies": [...],
  "links": [...],
  "strats": [...],
  "notables": [...]
}
```

## Required Fields

### Basic Information

```json
{
  "id": 42,              // Unique room ID
  "name": "Energy Tank Room",
  "area": "MDK",         // Sector: MDK, SRX, TRO, PYR, AQA, ARC, NOC
  "subarea": "Lower",    // Subsection within sector
  "subsubarea": "Pink"   // Optional fine-grained location
}
```

### Room Environments

Describes environmental properties:

```json
"roomEnvironments": [
  {
    "heated": false,
    "entranceNodes": [1, 2]  // Optional: only for these entrance nodes
  }
]
```

**Examples**:

```json
// Always heated
"roomEnvironments": [
  {
    "heated": true
  }
]

// Heated only when entering from certain nodes
"roomEnvironments": [
  {
    "heated": false,
    "entranceNodes": [1]
  },
  {
    "heated": true,
    "entranceNodes": [2, 3]
  }
]
```

### Map Tile Mask

2D array showing accessible tiles:

```json
"mapTileMask": [
  [1, 1, 1],
  [1, 0, 1],
  [1, 1, 1]
]
```

- `1`: Accessible tile
- `0`: Inaccessible tile

## Nodes

Nodes are locations within a room. Types include doors, items, junctions, and utility stations.

### Node Structure

```json
{
  "id": 1,
  "name": "Left Door",
  "nodeType": "door",
  "nodeSubType": "blue",
  "doorOrientation": "left",
  "doorEnvironments": [...]
}
```

### Node Types

#### 1. Door Nodes

Connections to other rooms:

```json
{
  "id": 1,
  "name": "Left Door",
  "nodeType": "door",
  "nodeSubType": "blue",
  "doorOrientation": "left",
  "doorEnvironments": [
    {
      "physics": "air"
    }
  ]
}
```

**Door SubTypes**:
- `blue`: Free to open
- `red`: Requires missiles
- `green`: Requires super missiles
- `yellow`: Requires power bombs
- `gray`: Requires special unlock
- `eye`: Boss eye door
- `elevator`: Elevator connection
- `passage`: Open passage

**Door Orientations**: `left`, `right`, `up`, `down`

**Physics Types**: `air`, `water`, `lava`, `acid`

**Complete Door Example**:

```json
{
  "id": 1,
  "name": "Left Door",
  "nodeType": "door",
  "nodeSubType": "red",
  "doorOrientation": "left",
  "doorEnvironments": [
    {
      "physics": "air"
    }
  ],
  "locks": [
    {
      "name": "Red Door Lock",
      "lockType": "coloredDoor",
      "unlockStrats": [
        {
          "name": "Base",
          "requires": [
            {
              "ammo": {
                "type": "Missile",
                "count": 1
              }
            }
          ]
        }
      ]
    }
  ]
}
```

#### 2. Item Nodes

Locations where items can be placed:

```json
{
  "id": 2,
  "name": "Energy Tank",
  "nodeType": "item",
  "nodeSubType": "visible",
  "nodeItem": "ETank"
}
```

**Item SubTypes**:
- `visible`: Clearly visible item
- `hidden`: Hidden item (behind blocks, etc.)
- `chozo`: Chozo statue item

**Example with Chozo**:

```json
{
  "id": 3,
  "name": "Morph Ball",
  "nodeType": "item",
  "nodeSubType": "chozo",
  "nodeItem": "Morph Ball",
  "note": "Requires defeating the Chozo statue"
}
```

#### 3. Junction Nodes

Intermediate points for complex traversal logic:

```json
{
  "id": 4,
  "name": "Middle Platform",
  "nodeType": "junction",
  "nodeSubType": "junction"
}
```

**Use Cases**:
- Breaking up complex room traversal
- Representing different heights/platforms
- Handling conditional room states

**Example**:

```json
{
  "id": 5,
  "name": "Top of Tall Room",
  "nodeType": "junction",
  "nodeSubType": "junction",
  "note": "The high platform area"
}
```

#### 4. Utility Nodes

Save stations, refill stations, map stations:

```json
{
  "id": 6,
  "name": "Save Station",
  "nodeType": "utility",
  "nodeSubType": "save",
  "utility": ["save"]
}
```

**Utility Types**:
- `save`: Save station
- `energy`: Energy refill
- `missile`: Missile refill
- `powerbomb`: Power Bomb refill
- `reserve`: Reserve tank refill
- `map`: Map station

**Example with Multiple Utilities**:

```json
{
  "id": 7,
  "name": "Refill Station",
  "nodeType": "utility",
  "nodeSubType": "energy",
  "utility": ["energy", "missile", "powerbomb"]
}
```

## Locks

Locks prevent access until unlocked:

```json
"locks": [
  {
    "name": "Boss Door",
    "lockType": "bossFight",
    "lock": ["f_DefeatedKraid"],
    "unlockStrats": [
      {
        "name": "Defeat Kraid",
        "requires": [
          {
            "enemyKill": {
              "enemies": [["Kraid"]]
            }
          }
        ]
      }
    ],
    "yields": ["f_DefeatedKraid"]
  }
]
```

**Lock Types**:
- `bossFight`: Locks until boss defeated
- `coloredDoor`: Locks until door opened with weapon
- `cutscene`: Locks until cutscene triggers
- `escapeFunnel`: One-way lock during escape
- `gameFlag`: Locks until flag set
- `killEnemies`: Locks until enemies cleared
- `permanent`: Never unlocks

**Lock Example (Colored Door)**:

```json
{
  "name": "Red Door",
  "lockType": "coloredDoor",
  "unlockStrats": [
    {
      "name": "Open with Missiles",
      "requires": [
        {
          "ammo": {
            "type": "Missile",
            "count": 1
          }
        }
      ]
    }
  ]
}
```

**Lock Example (Boss Fight)**:

```json
{
  "name": "Ridley Gate",
  "lockType": "bossFight",
  "lock": ["f_DefeatedRidley"],
  "unlockStrats": [
    {
      "name": "Defeat Ridley",
      "requires": [
        {
          "enemyKill": {
            "enemies": [["Ridley"]],
            "explicitWeapons": ["Charge Beam", "Missile", "Super"]
          }
        },
        {
          "heatFrames": 900
        }
      ]
    }
  ],
  "yields": ["f_DefeatedRidley"]
}
```

## Links

Links define which nodes are connected:

```json
"links": [
  {
    "from": 1,
    "to": [
      {"id": 2},
      {"id": 3}
    ]
  },
  {
    "from": 2,
    "to": [
      {"id": 1}
    ]
  }
]
```

**Example**:

```json
"links": [
  {
    "from": 1,
    "to": [
      {"id": 2, "note": "Main path"},
      {"id": 3, "note": "Secret passage"}
    ]
  }
]
```

## Strats

Strats (strategies) define HOW to traverse between nodes:

```json
{
  "id": 1,
  "link": [1, 2],
  "name": "Base",
  "requires": []
}
```

### Basic Strat

No requirements - anyone can do this:

```json
{
  "id": 1,
  "link": [1, 2],
  "name": "Walk Across",
  "requires": []
}
```

### Strat with Requirements

```json
{
  "id": 2,
  "link": [1, 3],
  "name": "Bomb Jump",
  "requires": [
    "Morph Ball",
    "Bombs"
  ]
}
```

### Multiple Strats for Same Link

```json
{
  "id": 3,
  "link": [1, 4],
  "name": "Hi-Jump Path",
  "requires": [
    "Hi-Jump"
  ]
},
{
  "id": 4,
  "link": [1, 4],
  "name": "Space Jump Path",
  "requires": [
    "Space Jump"
  ]
},
{
  "id": 5,
  "link": [1, 4],
  "name": "Wall Jump Path",
  "requires": [
    "canWallJump",
    {
      "enemyDamage": {
        "enemy": "Sidehopper",
        "type": "contact",
        "hits": 2
      }
    }
  ]
}
```

### Strat with Entrance Condition

For strats that require entering the room in a specific way:

```json
{
  "id": 6,
  "link": [1, 2],
  "name": "Speed Boost Through",
  "entranceCondition": {
    "comeInRunning": {
      "speedBooster": true,
      "minTiles": 3
    }
  },
  "requires": [
    "Speed Booster"
  ]
}
```

**Entrance Condition Types**:
- `comeInNormally`: Normal entry
- `comeInRunning`: Running with speed
- `comeInJumping`: Jumping through door
- `comeInShinecharging`: Charging shine spark
- `comeInShinecharged`: Already shinecharged
- `comeInWithSpark`: Shinesparking through door
- `comeInSpinning`: Spin jumping through
- And many more...

### Strat with Exit Condition

```json
{
  "id": 7,
  "link": [1, 2],
  "name": "Leave Shinecharged",
  "requires": [
    "Speed Booster",
    {
      "canShineCharge": {
        "usedTiles": 28,
        "openEnd": 1
      }
    }
  ],
  "exitCondition": {
    "leaveShinecharged": {}
  }
}
```

**Exit Condition Types**:
- `leaveNormally`: Normal exit
- `leaveWithRunway`: Exit with runway data
- `leaveShinecharged`: Exit shinecharged
- `leaveWithSpark`: Exit shinesparking
- And many more...

### Strat with Effects

Strats can have side effects:

```json
{
  "id": 8,
  "link": [1, 2],
  "name": "Break Blocks",
  "requires": [
    "Morph Ball",
    "Bombs"
  ],
  "clearsObstacles": ["A"],
  "note": "Bombing the crumble blocks"
}
```

**Effect Types**:
- `clearsObstacles`: Removes obstacles
- `resetsObstacles`: Resets obstacles to original state
- `unlocksDoors`: Unlocks doors
- `collectsItems`: Collects items
- `setsFlags`: Sets game flags

### Strat with Door Unlocking

```json
{
  "id": 9,
  "link": [1, 2],
  "name": "Open Red Door",
  "requires": [
    {
      "ammo": {
        "type": "Missile",
        "count": 1
      }
    }
  ],
  "unlocksDoors": [
    {
      "nodeId": 2,
      "types": ["missiles"],
      "requires": []
    }
  ]
}
```

## Obstacles

Destructible or clearable obstacles:

```json
"obstacles": [
  {
    "id": "A",
    "name": "Crumble Blocks",
    "obstacleType": "inanimate"
  },
  {
    "id": "B",
    "name": "Speed Booster Blocks",
    "obstacleType": "inanimate"
  }
]
```

**Obstacle Types**:
- `inanimate`: Blocks, barriers
- `enemies`: Enemy group that must be cleared
- `abstract`: Abstract concept (like a timer)

**Usage in Strats**:

```json
{
  "link": [1, 2],
  "name": "Path Blocked by A",
  "requires": [
    {
      "obstaclesCleared": ["A"]
    }
  ]
},
{
  "link": [1, 3],
  "name": "Clear Obstacle A",
  "requires": [
    "Morph Ball",
    "Bombs"
  ],
  "clearsObstacles": ["A"]
}
```

## Enemies

Enemy groups in the room:

```json
"enemies": [
  {
    "id": "e1",
    "groupName": "Morph Ball Room Sidehoppers",
    "enemyName": "Sidehopper",
    "quantity": 3,
    "homeNodes": [1, 2]
  },
  {
    "id": "e2",
    "groupName": "Corridor Zebes",
    "enemyName": "Zeb",
    "quantity": 5,
    "betweenNodes": [2, 3]
  }
]
```

**Fields**:
- `homeNodes`: Enemies at specific nodes
- `betweenNodes`: Enemies encountered traveling between nodes
- `spawn`: Requirements for enemies to spawn
- `stopSpawn`: Requirements for enemies to stop spawning

## Notables

Named techniques specific to the room:

```json
"notables": [
  {
    "id": 1,
    "name": "Reverse Shinespark",
    "note": [
      "Charge shine spark going left,",
      "then shinespark going right up the shaft"
    ]
  }
]
```

**Usage in Strats**:

```json
{
  "link": [1, 5],
  "name": "Reverse Shinespark to Top",
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
        "frames": 60
      }
    },
    {
      "notable": "Reverse Shinespark"
    }
  ]
}
```

## Complete Room Example

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 100,
  "name": "High Jump Room",
  "area": "MDK",
  "subarea": "Lower MDK",
  "roomEnvironments": [
    {
      "heated": false
    }
  ],
  "mapTileMask": [
    [1, 1, 1],
    [1, 1, 1]
  ],
  "nodes": [
    {
      "id": 1,
      "name": "Left Door",
      "nodeType": "door",
      "nodeSubType": "blue",
      "doorOrientation": "left",
      "doorEnvironments": [
        {
          "physics": "air"
        }
      ]
    },
    {
      "id": 2,
      "name": "Item",
      "nodeType": "item",
      "nodeSubType": "chozo",
      "nodeItem": "Hi-Jump"
    },
    {
      "id": 3,
      "name": "Right Door",
      "nodeType": "door",
      "nodeSubType": "blue",
      "doorOrientation": "right",
      "doorEnvironments": [
        {
          "physics": "air"
        }
      ]
    }
  ],
  "links": [
    {
      "from": 1,
      "to": [
        {"id": 2},
        {"id": 3}
      ]
    },
    {
      "from": 2,
      "to": [
        {"id": 1},
        {"id": 3}
      ]
    },
    {
      "from": 3,
      "to": [
        {"id": 1},
        {"id": 2}
      ]
    }
  ],
  "strats": [
    {
      "id": 1,
      "link": [1, 2],
      "name": "Get Item with Morph Ball",
      "requires": [
        "Morph Ball",
        {
          "or": [
            "Bombs",
            "Spring Ball"
          ]
        }
      ]
    },
    {
      "id": 2,
      "link": [1, 3],
      "name": "Walk Through",
      "requires": []
    },
    {
      "id": 3,
      "link": [2, 3],
      "name": "Leave After Getting Item",
      "requires": []
    },
    {
      "id": 4,
      "link": [3, 1],
      "name": "Walk Back",
      "requires": []
    }
  ]
}
```

## Best Practices

### 1. Clear Node Names

Use descriptive names:
- ✅ "Left Door", "Top Right Door"
- ❌ "Node 1", "Door A"

### 2. Provide Notes

```json
{
  "name": "Hidden Passage",
  "note": "Behind the speedbooster blocks in the ceiling"
}
```

### 3. Multiple Strats for Difficulty

```json
// Easy strat
{
  "link": [1, 2],
  "name": "With Hi-Jump",
  "requires": ["Hi-Jump"]
},
// Hard strat
{
  "link": [1, 2],
  "name": "Wall Jump",
  "requires": ["canWallJump"],
  "note": "Precise wall jumps required"
}
```

### 4. Break Up Complex Rooms

Use junction nodes for clarity:

```json
"nodes": [
  {"id": 1, "name": "Bottom Left Door", ...},
  {"id": 2, "name": "Middle Platform", "nodeType": "junction", ...},
  {"id": 3, "name": "Top Right Door", ...},
  {"id": 4, "name": "Item", ...}
]
```

### 5. Document Notables

```json
{
  "name": "Tricky Shinespark Sequence",
  "note": [
    "Charge in the long hallway,",
    "then shinespark diagonally through three screens",
    "while avoiding the Space Pirates"
  ]
}
```

## Next Steps

- Read [Requirements-Schema.md](Requirements-Schema.md) for requirement syntax
- Check [Connections-Schema.md](Connections-Schema.md) for inter-room links
- See [Graph-Builder.md](Graph-Builder.md) for how rooms become graphs