# Human-Readable Strat Language (HRSL) Specification v2

## Overview

HRSL is a simple, human-friendly language for describing Super Metroid strats. Node references use descriptive names instead of numeric IDs, making it easier to write and understand.

## Basic Structure

```
STRAT "Strat Name"
FROM: Bottom Left Door
TO: Top Right Door
ID: 42

[SECTIONS...]

NOTE: "Optional note"
```

## Node Naming

Nodes are referenced by their descriptive names from the room JSON:
- **Doors**: Use the full name (e.g., "Bottom Left Door", "Top Right Door")
- **Items**: Use the node name (e.g., "Missile Tank", "Item")
- **Junctions**: Use the full junction name (e.g., "Junction Below Bomb Blocks")

The converter will match these names to node IDs automatically.

## Core Sections

### Basic Info

```
STRAT "Cross Room with Heated Bomb Jump"
FROM: Bottom Left Door
TO: Bottom Right Corner Junction
ID: 42
```

### Requirements Section

```
REQUIRES:
  - Morph Ball
  - Bombs
  - tech: canBombJump
  - ammo: Missile x5
  - heatFrames: 180
  - enemyDamage: Kihunter "contact" x2
```

#### Requirement Types Reference

**Simple Items/Flags:**
```
REQUIRES:
  - Morph Ball
  - Speed Booster
  - f_DefeatedKraid
```

**Tech/Helper/Notable:**
```
REQUIRES:
  - tech: canWallJump
  - helper: h_canNavigateHeatRooms
  - notable: CrossRoomSpeedBooster
```

**Ammo:**
```
REQUIRES:
  - ammo: Missile x5
  - ammo: Super x3
  - ammo: PowerBomb x2
```

**Environmental Damage:**
```
REQUIRES:
  - heatFrames: 180
  - coldFrames: 120
  - lavaFrames: 60
  - acidFrames: 90
  - electricityFrames: 45
  - hibashiHits: 3
  - spikeHits: 2
  - thornHits: 1
```

**Enemy Damage:**
```
REQUIRES:
  - enemyDamage: Kihunter "contact" x2
  - enemyDamage: Sidehopper "jump" x1
```

**Enemy Kill:**
```
REQUIRES:
  - enemyKill: [[Multiviola], [Dessgeega, Dessgeega]]
    weapons: [Missile, Super]
    excluded: [PowerBomb]
```

**Shine Charge:**
```
REQUIRES:
  - canShineCharge: 25 tiles, openEnd: 1
  - canShineCharge: 18.5 tiles, openEnd: 0, gentleUp: 3, steepDown: 2
```

**Get Blue Speed:**
```
REQUIRES:
  - getBlueSpeed: 23 tiles, openEnd: 2
```

**Speed Ball:**
```
REQUIRES:
  - speedBall: 15 tiles, openEnd: 1
```

**Shinespark:**
```
REQUIRES:
  - shinespark: 30 frames
  - shinespark: 45 frames, excess: 10
```

**Resource Capacity:**
```
REQUIRES:
  - resourceCapacity: Missile 50
  - resourceCapacity: Energy 500
```

**Resource Available:**
```
REQUIRES:
  - resourceAvailable: Missile 10
  - resourceAvailable: Energy 100
```

**Resource At Most:**
```
REQUIRES:
  - resourceAtMost: Energy 50
```

**Auto Reserve Trigger:**
```
REQUIRES:
  - autoReserveTrigger: minReserve: 10, maxReserve: 95
```

**Reset Room:**
```
REQUIRES:
  - resetRoom: [Bottom Left Door, Top Right Door]
```

**Obstacles:**
```
REQUIRES:
  - obstaclesCleared: [ObstacleA, ObstacleB]
  - obstaclesNotCleared: [ObstacleC]
```

**Door Unlocked:**
```
REQUIRES:
  - doorUnlockedAt: Top Right Door
```

**Item State:**
```
REQUIRES:
  - itemNotCollectedAt: Missile Tank
  - itemCollectedAt: Energy Tank
```

**Disable Equipment:**
```
REQUIRES:
  - disableEquipment: Gravity Suit
```

**Flash Suit:**
```
REQUIRES:
  - gainFlashSuit: {}
  - useFlashSuit: {}
  - noFlashSuit: {}
```

**Refill:**
```
REQUIRES:
  - refill: [Missile, Super, Energy]
  - partialRefill: Energy, limit: 100
```

**Logical Operators:**
```
REQUIRES:
  - or:
      - Morph Ball
      - tech: canIBJ
  - and:
      - Speed Booster
      - tech: canShineCharge
  - not: Gravity Suit
```

### Entrance Conditions

```
ENTRANCE:
  comeInRunning:
    speedBooster: true
    minTiles: 15
    maxTiles: 25

ENTRANCE:
  comeInShinecharging:
    length: 12
    openEnd: 1
    gentleUpTiles: 2

ENTRANCE:
  comeInWithSpark:
    position: top

ENTRANCE:
  comeInWithGMode:
    mode: direct
    morphed: true
    mobility: mobile

ENTRANCE:
  comeInWithMockball:
    speedBooster: any
    adjacentMinTiles: 12

ENTRANCE:
  comeInWithSpringBallBounce:
    speedBooster: false
    movementType: controlled
    adjacentMinTiles: 10

ENTRANCE:
  comeInWithTemporaryBlue:
    direction: left

ENTRANCE:
  comeInSpinning:
    speedBooster: false
    unusableTiles: 3
    minExtraRunSpeed: $2.0
    maxExtraRunSpeed: $4.5

ENTRANCE:
  comeInWithStoredFallSpeed:
    fallSpeedInTiles: 1

ENTRANCE:
  comeInWithGrappleSwing:
    blocks:
      - position: [5, 3]
        environment: air
      - position: [8, 2]
        environment: water
        obstructions: [[6, 2], [7, 3]]

ENTRANCE:
  comeInWithPlatformBelow:
    minHeight: 2
    maxHeight: 5
    maxLeftPosition: -1
    minRightPosition: 2

ENTRANCE:
  comeInWithSidePlatform:
    platforms:
      - minHeight: 3
        maxHeight: 5
        minTiles: 8
        speedBooster: false
        obstructions: [[1, 0]]
```

### Exit Conditions

```
EXIT:
  leaveWithRunway:
    length: 12
    openEnd: 1
    gentleDownTiles: 3
    heated: true

EXIT:
  leaveShinecharged: {}

EXIT:
  leaveWithSpark:
    position: bottom

EXIT:
  leaveWithMockball:
    remoteRunway:
      length: 10
      openEnd: 1
    landingRunway:
      length: 5
      openEnd: 0
    blue: yes

EXIT:
  leaveSpinning:
    remoteRunway:
      length: 8
      openEnd: 0
    minExtraRunSpeed: $1.5
    blue: any

EXIT:
  leaveWithSpringBallBounce:
    remoteRunway:
      length: 12
      openEnd: 1
    landingRunway:
      length: 6
      openEnd: 0
    movementType: controlled
    blue: no

EXIT:
  leaveSpaceJumping:
    remoteRunway:
      length: 10
      openEnd: 1
    blue: any

EXIT:
  leaveWithTemporaryBlue:
    direction: right

EXIT:
  leaveWithGModeSetup:
    knockback: true

EXIT:
  leaveWithGMode:
    morphed: false

EXIT:
  leaveWithDoorFrameBelow:
    height: 5

EXIT:
  leaveWithPlatformBelow:
    height: 3
    leftPosition: -1.5
    rightPosition: 2.5

EXIT:
  leaveWithSidePlatform:
    height: 3
    runway:
      length: 12
      openEnd: 0
    obstruction: [1, 0]

EXIT:
  leaveWithGrappleSwing:
    blocks:
      - position: [5, 3]

EXIT:
  leaveWithGrappleJump:
    position: left
```

### Door Unlocks

Reference doors by name:

```
UNLOCKS:
  - door: Top Right Door
    types: [missiles, super, powerbomb]
    requires:
      - Morph Ball
  - door: Bottom Left Door
    types: [gray]
    requires: []
```

If no door is specified, it defaults to the destination node:

```
UNLOCKS:
  - types: [missiles, super]
    requires:
      - Speed Booster
```

### Obstacles

```
CLEARS: [ObstacleA, ObstacleB]
RESETS: [ObstacleC]
```

### Item Collection

Reference items by name:

```
COLLECTS: [Missile Tank, Energy Tank]
```

### Flags

```
FLAGS: [f_DefeatedBoss, f_UsedElevator]
```

### Special Properties

```
BYPASSES_DOOR: true
ENDS_WITH_CHARGE: true
STARTS_WITH_CHARGE: true
WALL_JUMP_AVOID: true
FLASH_SUIT_CHECKED: true
```

### G-Mode

```
GMODE_REGAIN_MOBILITY: {}
```

### Farm Drops

```
FARM_DROPS:
  - enemy: Kihunter
    count: 5
  - enemy: Sidehopper
    count: 3
```

### Failures

```
FAILURES:
  - name: "Fall Down"
    leadsTo: Junction Below Crumble Blocks
    cost:
      - enemyDamage: Sidehopper "contact" x1
  - name: "Softlock"
    softlock: true
```

### Notes

```
NOTE: "This is a challenging strat."
DETAIL: "Jump at the apex of the platform."
DEV: "Check frame-perfect window in RAM."
```

## Complete Example

```
STRAT "Cross Room with Heated Bomb Jump"
FROM: Bottom Left Door
TO: Bottom Right Corner Junction
ID: 42

REQUIRES:
  - Morph Ball
  - Bombs
  - tech: canBombJump
  - heatFrames: 180
  - enemyDamage: Kihunter "contact" x2
  - or:
      - Varia Suit
      - resourceCapacity: Energy 200

ENTRANCE:
  comeInRunning:
    speedBooster: false
    minTiles: 3

EXIT:
  leaveWithRunway:
    length: 8
    openEnd: 1

UNLOCKS:
  - door: Junction Below Bomb Blocks
    types: [missiles, super]

CLEARS: [All Bomb Blocks]
COLLECTS: [Missile Tank]
FLAGS: [f_CrossedHeatRoom]

FARM_DROPS:
  - enemy: Kihunter
    count: 3

FAILURES:
  - name: "Missed Jump"
    leadsTo: Bottom Left Door
    cost:
      - heatFrames: 60

NOTE: "Requires precise bomb timing in heat."
DETAIL: "Place first bomb immediately after morphing."
DEV: "Heat damage calculated at 1 energy per 4 frames."
```

## Real Example from Mickey Mouse Room

```
STRAT "Leave With Runway (Full Runway)"
FROM: Bottom Left Door
TO: Bottom Left Door
ID: 1

REQUIRES:
  - heatFrames: 250
  - or:
      - obstaclesCleared: [Runway Dessgeegas]
      - and:
          - heatFrames: 100
          - enemyKill: [[Dessgeega, Dessgeega, Dessgeega]]
            weapons: [ScrewAttack]
      - and:
          - heatFrames: 350
          - enemyKill: [[Dessgeega, Dessgeega, Dessgeega]]
            weapons: [Plasma, Super, Missile]

EXIT:
  leaveWithRunway:
    length: 45
    openEnd: 1

FLASH_SUIT_CHECKED: true

NOTE: "This runway requires killing the Dessgeegas closest to the door."
DEV: "The Multiviolas move out of the way if coming from the left."
```

## Tips for Writing Strats

1. **Use exact node names** from the room JSON
2. **Reference obstacles by ID** (e.g., "ObstacleA", "Runway Dessgeegas")
3. **Reference enemies by name** from enemies.json (e.g., "Multiviola", "Dessgeega")
4. **Use descriptive flag names** with `f_` prefix
5. **Keep requirements readable** - use indentation for nested OR/AND
6. **Add notes liberally** - they help other testers understand the strat

## Conversion Process

The converter will:
1. Parse the HRSL file
2. Look up node names in the room JSON to get IDs
3. Look up adjacent rooms for door connections
4. Validate item/tech/enemy references against database files
5. Generate proper JSON with numeric IDs
6. Report any unmatched references