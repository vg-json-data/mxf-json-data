# Connections Schema Guide

## What Are Connections?

Connections are the **doorways between rooms**. When you walk through a door in one room, a connection tells the randomizer which door you come out of in the next room.

Think of it like this:
- **Rooms** = Individual levels/areas
- **Nodes** = Specific locations in a room (like doors)
- **Connections** = The "wires" that link doors between rooms

## Schema Location

`schema/mxf-connection.schema.json`

## Basic Structure

```json
{
  "connections": [
    {
      "connectionType": "HorizontalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {
          "area": "MDK",
          "subarea": "Lower MDK",
          "roomid": 1,
          "roomName": "First Room",
          "nodeid": 2,
          "nodeName": "Right Door",
          "position": "right"
        },
        {
          "area": "MDK",
          "subarea": "Lower MDK",
          "roomid": 2,
          "roomName": "Second Room",
          "nodeid": 1,
          "nodeName": "Left Door",
          "position": "left"
        }
      ]
    }
  ]
}
```

## Understanding the Example

Let's break down what this means in plain English:

> "Room 1's right door connects to Room 2's left door. You can go both ways (bidirectional). When you go through Room 1's right door, you come out of Room 2's left door."

## Connection Types

### HorizontalDoor
Regular doors you walk through left/right:

```json
{
  "connectionType": "HorizontalDoor",
  "direction": "Bidirectional",
  "nodes": [...]
}
```

**Use When**: Connecting rooms side-by-side

**Example**: 
- Room A's right door → Room B's left door
- Room B's left door → Room A's right door

### VerticalDoor
Doors you jump up through or fall down through:

```json
{
  "connectionType": "VerticalDoor",
  "direction": "Bidirectional",
  "nodes": [...]
}
```

**Use When**: Connecting rooms above/below each other

**Example**:
- Room A's top door → Room B's bottom door
- Room B's bottom door → Room A's top door

### HorizontalMorphTunnel
Small tunnels only accessible with Morph Ball:

```json
{
  "connectionType": "HorizontalMorphTunnel",
  "direction": "Bidirectional",
  "nodes": [...]
}
```

**Use When**: Small horizontal passages

**Example**: Morph ball tunnels between rooms

### VerticalMorphTunnel
Vertical shafts requiring Morph Ball:

```json
{
  "connectionType": "VerticalMorphTunnel",
  "direction": "Bidirectional",
  "nodes": [...]
}
```

**Use When**: Vertical morph ball passages

### Elevator
Elevator transitions between sectors:

```json
{
  "connectionType": "Elevator",
  "direction": "Bidirectional",
  "nodes": [...]
}
```

**Use When**: Major elevators between game areas

**Example**: MDK → SRX elevator

### TransitionTube
Special transition tubes (often used for cutscenes):

```json
{
  "connectionType": "TransitionTube",
  "direction": "Forward",
  "nodes": [...]
}
```

**Use When**: One-way tubes or special transitions

### StoryMarker
Story progression points:

```json
{
  "connectionType": "StoryMarker",
  "direction": "Forward",
  "nodes": [...]
}
```

**Use When**: Marking story progression (rarely used for actual connections)

## Direction Types

### Bidirectional
You can go both ways:

```json
"direction": "Bidirectional"
```

**Means**: Door A → Door B AND Door B → Door A

**Example**: Normal doors you can walk through in either direction

```
Room 1          Room 2
┌──────┐       ┌──────┐
│      │◄─────►│      │
│      │       │      │
└──────┘       └──────┘
```

### Forward
One-way only:

```json
"direction": "Forward"
```

**Means**: Door A → Door B only (can't go back)

**Example**: Escape sequence, special transitions, drop-downs

```
Room 1          Room 2
┌──────┐       ┌──────┐
│      │──────►│      │
│      │       │      │
└──────┘       └──────┘
```

## Node Fields

Each node in a connection has these fields:

### area
Which sector the room is in:

```json
"area": "MDK"
```

**Options**: `MDK`, `SRX`, `TRO`, `PYR`, `AQA`, `ARC`, `NOC`

### subarea
Which part of the sector:

```json
"subarea": "Lower MDK"
```

**Examples**: `Lower MDK`, `Upper SRX`, `East TRO`

### roomid
The room's unique ID number:

```json
"roomid": 42
```

**Note**: Must match a room's `id` field in the room JSON files

### roomName
The room's name:

```json
"roomName": "Morph Ball Room"
```

**Note**: Must match the room's `name` field

### nodeid
The node's ID within the room:

```json
"nodeid": 1
```

**Note**: Must match a node's `id` field in the room

### nodeName
The node's name:

```json
"nodeName": "Left Door"
```

**Note**: Must match the node's `name` field

### position
Which side of the room this door is on:

```json
"position": "left"
```

**Options**: `left`, `right`, `top`, `bottom`

**Rules**:
- Horizontal connections: use `left` or `right`
- Vertical connections: use `top` or `bottom`

## Complete Examples

### Example 1: Simple Horizontal Connection

Two rooms side by side with blue doors:

```json
{
  "connectionType": "HorizontalDoor",
  "direction": "Bidirectional",
  "description": "Main corridor connection",
  "nodes": [
    {
      "area": "MDK",
      "subarea": "Lower MDK",
      "roomid": 10,
      "roomName": "First Corridor",
      "nodeid": 2,
      "nodeName": "Right Door",
      "position": "right"
    },
    {
      "area": "MDK",
      "subarea": "Lower MDK",
      "roomid": 11,
      "roomName": "Second Corridor",
      "nodeid": 1,
      "nodeName": "Left Door",
      "position": "left"
    }
  ]
}
```

**What This Means**:
- When you exit First Corridor's right door, you enter Second Corridor's left door
- You can go back the same way

### Example 2: Vertical Connection

Room above another room:

```json
{
  "connectionType": "VerticalDoor",
  "direction": "Bidirectional",
  "description": "Vertical shaft connection",
  "nodes": [
    {
      "area": "MDK",
      "subarea": "Lower MDK",
      "roomid": 20,
      "roomName": "Bottom Room",
      "nodeid": 3,
      "nodeName": "Top Door",
      "position": "top"
    },
    {
      "area": "MDK",
      "subarea": "Upper MDK",
      "roomid": 21,
      "roomName": "Top Room",
      "nodeid": 4,
      "nodeName": "Bottom Door",
      "position": "bottom"
    }
  ]
}
```

**What This Means**:
- When you exit Bottom Room's top door, you enter Top Room's bottom door
- You can drop back down

### Example 3: One-Way Drop

Can drop down but can't go back up:

```json
{
  "connectionType": "VerticalDoor",
  "direction": "Forward",
  "description": "One-way drop shaft",
  "nodes": [
    {
      "area": "SRX",
      "subarea": "Upper SRX",
      "roomid": 30,
      "roomName": "High Ledge Room",
      "nodeid": 5,
      "nodeName": "Drop Hole",
      "position": "bottom"
    },
    {
      "area": "SRX",
      "subarea": "Lower SRX",
      "roomid": 31,
      "roomName": "Bottom Pit",
      "nodeid": 6,
      "nodeName": "Landing Zone",
      "position": "top"
    }
  ]
}
```

**What This Means**:
- You can drop from High Ledge Room into Bottom Pit
- You CANNOT go back up through this connection
- Need to find another route back

### Example 4: Morph Ball Tunnel

Small passage requiring Morph Ball:

```json
{
  "connectionType": "HorizontalMorphTunnel",
  "direction": "Bidirectional",
  "description": "Secret morph tunnel",
  "nodes": [
    {
      "area": "TRO",
      "subarea": "East TRO",
      "roomid": 40,
      "roomName": "Hidden Passage Start",
      "nodeid": 7,
      "nodeName": "Tunnel Entrance",
      "position": "right"
    },
    {
      "area": "TRO",
      "subarea": "East TRO",
      "roomid": 41,
      "roomName": "Secret Room",
      "nodeid": 8,
      "nodeName": "Tunnel Exit",
      "position": "left"
    }
  ]
}
```

**What This Means**:
- Small passage between rooms
- Requires Morph Ball to traverse
- Can go both ways

### Example 5: Elevator

Major transition between sectors:

```json
{
  "connectionType": "Elevator",
  "direction": "Bidirectional",
  "description": "MDK to SRX elevator",
  "nodes": [
    {
      "area": "MDK",
      "subarea": "Central MDK",
      "roomid": 50,
      "roomName": "MDK Elevator",
      "nodeid": 9,
      "nodeName": "Elevator Platform",
      "position": "top"
    },
    {
      "area": "SRX",
      "subarea": "Central SRX",
      "roomid": 51,
      "roomName": "SRX Elevator",
      "nodeid": 10,
      "nodeName": "Elevator Platform",
      "position": "bottom"
    }
  ]
}
```

**What This Means**:
- Connects two major areas
- Can travel both directions
- Usually has a loading screen

## Visual Guide

### How Connections Work

```
┌─────────────────┐         ┌─────────────────┐
│   Room 1        │         │   Room 2        │
│                 │         │                 │
│  [1] Left Door  │         │  [2] Item       │
│  [2] Right Door │◄───────►│  [1] Left Door  │
│                 │         │                 │
└─────────────────┘         └─────────────────┘

Connection:
{
  "nodes": [
    {"roomid": 1, "nodeid": 2, "position": "right"},
    {"roomid": 2, "nodeid": 1, "position": "left"}
  ]
}
```

### Position Rules

```
        top
         ↑
    ┌─────────┐
left← │  Room  │ →right
    └─────────┘
         ↓
       bottom
```

**For Horizontal Connections**:
- First node: `right` (exiting to the right)
- Second node: `left` (entering from the left)

**For Vertical Connections**:
- First node: `top` (exiting upward)
- Second node: `bottom` (entering from below)

## Common Patterns

### Pattern 1: Linear Corridor

Three rooms in a row:

```json
{
  "connections": [
    {
      "connectionType": "HorizontalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 1, "nodeid": 2, "position": "right"},
        {"roomid": 2, "nodeid": 1, "position": "left"}
      ]
    },
    {
      "connectionType": "HorizontalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 2, "nodeid": 2, "position": "right"},
        {"roomid": 3, "nodeid": 1, "position": "left"}
      ]
    }
  ]
}
```

**Result**: Room 1 ↔ Room 2 ↔ Room 3

### Pattern 2: Vertical Shaft

Rooms stacked vertically:

```json
{
  "connections": [
    {
      "connectionType": "VerticalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 10, "nodeid": 3, "position": "top"},
        {"roomid": 11, "nodeid": 4, "position": "bottom"}
      ]
    },
    {
      "connectionType": "VerticalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 11, "nodeid": 3, "position": "top"},
        {"roomid": 12, "nodeid": 4, "position": "bottom"}
      ]
    }
  ]
}
```

**Result**: 
```
Room 12 (top)
    ↕
Room 11 (middle)
    ↕
Room 10 (bottom)
```

### Pattern 3: Room with Multiple Exits

One room connecting to multiple others:

```json
{
  "connections": [
    {
      "connectionType": "HorizontalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 20, "nodeid": 1, "position": "left"},
        {"roomid": 21, "nodeid": 2, "position": "right"}
      ]
    },
    {
      "connectionType": "HorizontalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 20, "nodeid": 2, "position": "right"},
        {"roomid": 22, "nodeid": 1, "position": "left"}
      ]
    },
    {
      "connectionType": "VerticalDoor",
      "direction": "Bidirectional",
      "nodes": [
        {"roomid": 20, "nodeid": 3, "position": "top"},
        {"roomid": 23, "nodeid": 4, "position": "bottom"}
      ]
    }
  ]
}
```

**Result**: Room 20 is a hub connecting to rooms 21, 22, and 23

## How to Add a New Connection

### Step 1: Identify the Rooms

Find the two rooms you want to connect:
- Room A: ID 100, has a right door (node 2)
- Room B: ID 101, has a left door (node 1)

### Step 2: Choose Connection Type

Is it horizontal, vertical, an elevator, or a tunnel?
- Horizontal door (walking through)

### Step 3: Choose Direction

Can you go both ways?
- Yes (bidirectional)

### Step 4: Write the JSON

```json
{
  "connectionType": "HorizontalDoor",
  "direction": "Bidirectional",
  "description": "Optional description",
  "nodes": [
    {
      "area": "MDK",
      "subarea": "Lower MDK",
      "roomid": 100,
      "roomName": "Room A Name",
      "nodeid": 2,
      "nodeName": "Right Door",
      "position": "right"
    },
    {
      "area": "MDK",
      "subarea": "Lower MDK",
      "roomid": 101,
      "roomName": "Room B Name",
      "nodeid": 1,
      "nodeName": "Left Door",
      "position": "left"
    }
  ]
}
```

### Step 5: Verify

Check that:
- ✅ Room IDs match room JSON files
- ✅ Node IDs match node definitions in rooms
- ✅ Room names match exactly
- ✅ Node names match exactly
- ✅ Positions are correct (left/right for horizontal, top/bottom for vertical)

## Common Mistakes

### ❌ Mistake 1: Wrong Position

```json
// WRONG - horizontal door using top/bottom
{
  "connectionType": "HorizontalDoor",
  "nodes": [
    {"position": "top"},    // ← Wrong!
    {"position": "bottom"}  // ← Wrong!
  ]
}
```

**Fix**: Use `left`/`right` for horizontal doors

### ❌ Mistake 2: Mismatched Room/Node IDs

```json
// WRONG - room 100 doesn't have node 99
{
  "nodes": [
    {"roomid": 100, "nodeid": 99}  // ← Node doesn't exist!
  ]
}
```

**Fix**: Check the room JSON to find the correct node ID

### ❌ Mistake 3: Backwards Positions

```json
// WRONG - right door connecting to right door
{
  "nodes": [
    {"roomid": 1, "position": "right"},
    {"roomid": 2, "position": "right"}  // ← Should be "left"
  ]
}
```

**Fix**: First room's right connects to second room's left

### ❌ Mistake 4: Typo in Names

```json
// WRONG - name doesn't match room
{
  "roomid": 42,
  "roomName": "Morfh Ball Room"  // ← Typo!
}
```

**Fix**: Copy the exact name from the room JSON

## Testing Your Connection

After adding a connection:

1. **Check Syntax**: Make sure JSON is valid
2. **Verify IDs**: Room and node IDs must exist
3. **Test Navigation**: Load the randomizer and verify you can traverse the connection
4. **Check Both Directions**: If bidirectional, test going both ways

## Optional Fields

### description
Human-readable description:

```json
{
  "description": "Main entrance to Morph Ball area",
  "connectionType": "HorizontalDoor",
  ...
}
```

### note
Additional notes:

```json
{
  "note": "This connection is only accessible after defeating the boss",
  ...
}
```

### devNote
Developer notes:

```json
{
  "devNote": "TODO: Verify this connection's position after room redesign",
  ...
}
```

## Next Steps

- Read [Room-Schema.md](Room-Schema.md) to understand nodes better
- Check [Graph-Builder.md](Graph-Builder.md) to see how connections become edges
- See [Logic-System-Overview.md](Logic-System-Overview.md) for the big picture

## Quick Reference

### Connection Types Cheatsheet

| Type | Use For | Direction |
|------|---------|-----------|
| `HorizontalDoor` | Side-by-side rooms | Usually Bidirectional |
| `VerticalDoor` | Stacked rooms | Usually Bidirectional |
| `HorizontalMorphTunnel` | Horizontal morph passages | Bidirectional |
| `VerticalMorphTunnel` | Vertical morph passages | Bidirectional |
| `Elevator` | Sector transitions | Bidirectional |
| `TransitionTube` | Special transitions | Usually Forward |
| `StoryMarker` | Story progression | Forward |

### Position Rules Cheatsheet

**Horizontal Connections**:
- Room on left: use `right` position
- Room on right: use `left` position

**Vertical Connections**:
- Room below: use `top` position
- Room above: use `bottom` position