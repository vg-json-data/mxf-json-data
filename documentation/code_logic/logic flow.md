# Logic Flow Guide

## Overview

This document explains how data flows through the randomizer from user input to final seed output.

## Entry Point

`randomizer.js` → `window.ProcessLogic(userSettings, randoVersion, gameVersion)`

## Complete Flow Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                         USER INPUT                                │
│  {                                                                │
│    seed: "my-custom-seed",                                        │
│    difficulty: "normal",                                          │
│    startLocation: "random",                                       │
│    etankCount: 14,                                                │
│    missileCount: 47,                                              │
│    ...tech settings...                                            │
│  }                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                    STEP 1: VALIDATE DATA                          │
│  window.MXFDataManager.isDataLoaded()                             │
│  ✓ Rooms loaded                                                   │
│  ✓ Connections loaded                                             │
│  ✓ Items/Tech/Helpers loaded                                     │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                    STEP 2: GENERATE SEED                          │
│  seed = generateSeed(userSettings)                                │
│  - If manual seed: use FNV-1a hash of seed string                │
│  - If auto: hash(settings + timestamp)                            │
│  - Create RNG(seed) for deterministic randomization              │
│                                                                    │
│  Output: seed = 8472615392847561                                  │
│          seedHash = "ALPHA BETA GAMMA DELTA"                      │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                  STEP 3: BUILD WORLD GRAPH                        │
│  graphBuilder = new GraphBuilder(userSettings)                    │
│  graph = graphBuilder.build()                                     │
│                                                                    │
│  Process:                                                         │
│  1. Load all room JSON files                                     │
│  2. Create nodes for each room location                          │
│  3. Create edges for each strat (intra-room)                     │
│  4. Create connections between rooms (inter-room)                │
│  5. Process door locks                                            │
│                                                                    │
│  Output: {                                                        │
│    rooms: { 1: {...}, 2: {...}, ... },                          │
│    nodes: { "1:1": {...}, "1:2": {...}, ... },                  │
│    areas: Set("MDK", "SRX", ...)                                 │
│  }                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                 STEP 4: INITIALIZE SYSTEMS                        │
│  evaluator = new RequirementEvaluator(graph, userSettings)       │
│  navigator = new GraphNavigator(graph, evaluator)                │
│                                                                    │
│  Purpose:                                                         │
│  - Evaluator: Checks if requirements are satisfied               │
│  - Navigator: Finds reachable nodes using BFS                    │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│              STEP 5: PREPARE ITEMS & LOCATIONS                    │
│  {itemPool, reserveXItems, locationPool, locationsByArea} =      │
│    getItemsAndLocations(graph, userSettings)                      │
│                                                                    │
│  Process:                                                         │
│  1. Get upgrade items from items.json                            │
│  2. Add expansion items (ETanks, Missiles, PowerBombs)           │
│  3. Collect all item locations from graph                        │
│  4. Categorize locations:                                         │
│     - auxPowerCell: Aux Power Cell locations                     │
│     - etecoon: Etecoon locations                                 │
│     - reserveX: Reserve-X locations                              │
│     - standard: Everything else                                   │
│  5. Group locations by area                                       │
│                                                                    │
│  Output:                                                          │
│  itemPool = ['Morph Ball', 'Bombs', ..., 'ETank', ...]          │
│  reserveXItems = ['Reserve X1', 'Reserve X2', ...]               │
│  locationPool = {                                                 │
│    auxPowerCell: [...],                                           │
│    etecoon: [...],                                                │
│    reserveX: [...],                                               │
│    standard: [...]                                                │
│  }                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│             STEP 6: DETERMINE START LOCATION                      │
│  startInfo = determineStartLocation(userSettings, graph)          │
│                                                                    │
│  Process:                                                         │
│  1. If settings specify locations, choose random from selected   │
│  2. Otherwise use default start location                          │
│  3. Find room and extract start nodes                            │
│                                                                    │
│  Output: {                                                        │
│    roomId: 1,                                                     │
│    roomName: "Revival Room",                                      │
│    area: "MDK",                                                   │
│    fullName: "MDK - Revival Room",                               │
│    startNodes: ["1:1", "1:2"]                                     │
│  }                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│               STEP 7: CREATE INITIAL STATE                        │
│  initialState = createInitialState(userSettings)                  │
│                                                                    │
│  Process:                                                         │
│  1. Create GameState with settings                               │
│  2. Add starting items from items.json                            │
│  3. Set starting HP/ammo                                          │
│                                                                    │
│  Output: GameState {                                              │
│    items: Set('Ship', ...startingItems),                         │
│    hpCur: 99,                                                     │
│    hpMax: 99,                                                     │
│    ammo: { missiles: 0, supers: 0, powerbombs: 0 }              │
│  }                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                   STEP 8: PLACE ITEMS                             │
│  placer = new ItemPlacer(graph, navigator, evaluator, rng,       │
│                           userSettings)                           │
│  placements = placer.placeItemsAssumedFill(                       │
│    startInfo.startNodes,                                          │
│    initialState,                                                  │
│    itemPool,                                                      │
│    reserveXItems,                                                 │
│    locationPool,                                                  │
│    locationsByArea                                                │
│  )                                                                │
│                                                                    │
│  Process (Assumed Fill):                                          │
│  1. Place special categories:                                     │
│     a. Aux Power Cells (shuffle among themselves)                │
│     b. Etecoon (shuffle among themselves)                        │
│     c. Reserve-X (one per area, no duplicates)                   │
│  2. Filter used locations from standard pool                     │
│  3. Split items: major vs filler                                  │
│  4. Place major items (progression first!)                       │
│     - Find reachable locations                                    │
│     - Choose random reachable location                           │
│     - Place item                                                  │
│     - Add item to state                                          │
│     - Repeat                                                      │
│  5. Place filler items (same process)                            │
│                                                                    │
│  Output: [                                                        │
│    {                                                              │
│      item: "Morph Ball",                                          │
│      locationNodeId: "42:2",                                      │
│      locationName: "Item",                                        │
│      roomId: 42,                                                  │
│      roomName: "Morph Ball Room",                                │
│      roomArea: "MDK",                                             │
│      isProgression: true,                                         │
│      category: "major"                                            │
│    },                                                             │
│    ...                                                            │
│  ]                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│              STEP 9: ANALYZE PROGRESSION                          │
│  analyzer = new ProgressionAnalyzer(graph, navigator,            │
│                                     evaluator, userSettings)      │
│  spheres = analyzer.analyzeWithPlacements(                        │
│    startInfo.startNodes,                                          │
│    initialState,                                                  │
│    placements                                                     │
│  )                                                                │
│                                                                    │
│  Process:                                                         │
│  1. Start with initial state                                      │
│  2. Find reachable nodes                                          │
│  3. Collect items at reachable nodes                             │
│  4. Add items to state                                            │
│  5. Repeat until no new nodes found                              │
│                                                                    │
│  Output: [                                                        │
│    {                                                              │
│      index: 1,                                                    │
│      nodes: ["1:1", "1:2", "2:1", ...],                         │
│      items: [                                                     │
│        {                                                          │
│          nodeId: "2:3",                                           │
│          item: "Morph Ball",                                      │
│          roomName: "Morph Ball Room",                            │
│          locationName: "Item",                                    │
│          isProgression: true                                      │
│        },                                                         │
│        ...                                                        │
│      ]                                                            │
│    },                                                             │
│    ...                                                            │
│  ]                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│               STEP 10: BUILD PLAYTHROUGH                          │
│  playthrough = analyzer.buildPlaythrough(spheres, placements)     │
│                                                                    │
│  Process:                                                         │
│  1. Convert sphere data into human-readable format               │
│  2. Add descriptions for each sphere                             │
│  3. Format item and location names                               │
│                                                                    │
│  Output: [                                                        │
│    {                                                              │
│      sphere: 1,                                                   │
│      description: "Starting area - Initial locations accessible", │
│      nodesAccessible: 15,                                         │
│      itemsFound: 3,                                               │
│      items: [                                                     │
│        {                                                          │
│          item: "Morph Ball",                                      │
│          location: "[Morph Ball Room] Item",                     │
│          isProgression: true                                      │
│        },                                                         │
│        ...                                                        │
│      ]                                                            │
│    },                                                             │
│    ...                                                            │
│  ]                                                                │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                      OUTPUT                                       │
│  seedSolution = {                                                 │
│    seed: 8472615392847561,                                        │
│    seedHash: "ALPHA BETA GAMMA DELTA",                           │
│    seedHashBase64: "QUxQSEEgQkVUQSBHQU1NQSBERUxUQQ",             │
│    timestamp: "2025-01-15T12:34:56.789Z",                        │
│    randoVersion: "1.0.0",                                         │
│    gameVersion: "1.0.0",                                          │
│    settings: {...},                                               │
│    startLocation: {...},                                          │
│    startNodes: [...],                                             │
│    itemPlacements: [...],                                         │
│    playthrough: [...],                                            │
│    graphSummary: {                                                │
│      totalRooms: 150,                                             │
│      totalNodes: 1000,                                            │
│      itemNodes: 120,                                              │
│      doorNodes: 400,                                              │
│      areas: ["MDK", "SRX", "TRO", "PYR", "AQA", "ARC", "NOC"]   │
│    }                                                              │
│  }                                                                │
└──────────────────────────────────────────────────────────────────┘
```

## Detailed Step Examples

### Step 3 Example: Graph Building

```javascript
// Input: Room JSON
{
  "id": 42,
  "name": "Morph Ball Room",
  "nodes": [
    {"id": 1, "name": "Left Door", "nodeType": "door"},
    {"id": 2, "name": "Item", "nodeType": "item"}
  ],
  "strats": [
    {
      "id": 1,
      "link": [1, 2],
      "requires": ["Morph Ball"]
    }
  ]
}

// Output: Graph Structure
graph.rooms[42] = {
  id: 42,
  name: "Morph Ball Room",
  nodes: {
    "42:1": {...},
    "42:2": {...}
  }
};

graph.nodes["42:1"] = {
  id: "42:1",
  type: "door",
  links: [
    {
      type: "strat",
      from: "42:1",
      to: "42:2",
      requirements: ["Morph Ball"]
    }
  ]
};
```

### Step 4 Example: Requirement Evaluation

```javascript
// Input: Requirement
const req = [
  "Morph Ball",
  {
    "or": ["Bombs", "Spring Ball"]
  }
];

// State
const state = {
  items: Set("Morph Ball", "Bombs")
};

// Evaluation
evaluator.evaluate(req, state);
// Step 1: Check "Morph Ball" → ✓ in state.items
// Step 2: Check OR → ✓ "Bombs" in state.items
// Result: {satisfied: true, cost: {}}
```

### Step 5 Example: Item & Location Preparation

```javascript
// Input: Graph with item nodes
graph.nodes = {
  "1:1": {type: "door", ...},
  "1:2": {type: "item", name: "Energy Tank", item: "ETank"},
  "2:1": {type: "door", ...},
  "2:2": {type: "item", name: "Morph Ball", item: "Morph Ball"},
  "3:1": {type: "door", ...},
  "3:2": {type: "item", name: "Aux Power Cell", item: "Aux Power Cell 1"},
  ...
};

// Output: Location Pool
locationPool = {
  auxPowerCell: [
    {nodeId: "3:2", nodeName: "Aux Power Cell", roomId: 3, ...}
  ],
  etecoon: [],
  reserveX: [],
  standard: [
    {nodeId: "1:2", nodeName: "Energy Tank", roomId: 1, ...},
    {nodeId: "2:2", nodeName: "Morph Ball", roomId: 2, ...},
    ...
  ]
};

// Item Pool
itemPool = [
  "Morph Ball", "Bombs", "Hi-Jump", ...,
  "ETank", "ETank", "Missile", ...
];
```

### Step 8 Example: Item Placement

```javascript
// Iteration 1
state = {items: Set("Ship")};
reachable = ["1:1", "1:2", "2:1"];
reachableLocations = [
  {nodeId: "1:2", ...},  // Energy Tank location
];

// Place first major item
item = "Morph Ball";
location = reachableLocations[0];  // "1:2"
placements.push({
  item: "Morph Ball",
  locationNodeId: "1:2",
  ...
});
state.addItem("Morph Ball");

// Iteration 2
state = {items: Set("Ship", "Morph Ball")};
reachable = ["1:1", "1:2", "2:1", "2:2", "3:1"];  // More locations!
reachableLocations = [
  {nodeId: "2:2", ...},  // Original Morph Ball location
  {nodeId: "3:1", ...},  // Another location
];

// Place second major item
item = "Bombs";
location = reachableLocations[1];  // "3:1"
placements.push({
  item: "Bombs",
  locationNodeId: "3:1",
  ...
});
state.addItem("Bombs");
```

### Step 9 Example: Progression Analysis

```javascript
// Sphere 1
state = {items: Set("Ship")};
reachable = findReachableNodes(["ship"], state);
// Result: ["1:1", "1:2", "2:1"]

items = [
  {item: "Morph Ball", location: "1:2"}  // Placed at 1:2
];

spheres[0] = {
  index: 1,
  nodes: ["1:1", "1:2", "2:1"],
  items: items
};

state.addItem("Morph Ball");

// Sphere 2
reachable = findReachableNodes(["ship"], state);
// Result: ["1:1", "1:2", "2:1", "2:2", "3:1", "3:2"]

items = [
  {item: "Bombs", location: "3:1"}  // Placed at 3:1
];

spheres[1] = {
  index: 2,
  nodes: ["2:2", "3:1", "3:2"],  // Only NEW nodes
  items: items
};

state.addItem("Bombs");
```

## Error Handling

### Data Not Loaded

```javascript
if (!window.MXFDataManager || !window.MXFDataManager.isDataLoaded()) {
  throw new Error("MXF data not loaded");
}
```

### Invalid Settings

```javascript
try {
  const seed = generateSeed(userSettingsJSON);
} catch (error) {
  console.error("Failed to generate seed:", error);
  throw error;
}
```

### Placement Failures

```javascript
if (items.length > 0) {
  console.warn(`⚠ ${items.length} items could not be placed`);
  // Continue anyway - seed may still be valid
}
```

### Graph Errors

```javascript
if (!graph.rooms || Object.keys(graph.rooms).length === 0) {
  throw new Error("Graph building failed - no rooms loaded");
}
```

## Console Logging

Throughout the process, detailed logging helps with debugging:

```javascript
console.log("\n" + "=".repeat(80));
console.log("METROID X-FUSION RANDOMIZER - LOGIC PROCESSING");
console.log("=".repeat(80));

console.log("\n[STEP 1] Validating data...");
console.log("\n[STEP 2] Generating seed...");
console.log(`  Seed: ${seed}`);
console.log(`  Hash: ${seedHash}`);

console.log("\n[STEP 3] Building world graph...");
// ... GraphBuilder logs its own progress

console.log("\n[STEP 8] Placing items...");
// ... ItemPlacer logs placement progress
console.log(`  Placed ${placements.length} items`);

console.log("\n[STEP 9] Analyzing progression...");
// ... ProgressionAnalyzer logs sphere generation
console.log(`  Generated ${spheres.length} spheres`);

console.log("\n" + "=".repeat(80));
console.log("LOGIC PROCESSING COMPLETE");
console.log("=".repeat(80));
```

## Performance Metrics

Typical execution times on a modern browser:

- **Step 1 (Validate)**: < 1ms
- **Step 2 (Seed)**: < 1ms
- **Step 3 (Graph)**: 50-100ms
- **Step 4 (Systems)**: < 1ms
- **Step 5 (Items/Locs)**: 10-20ms
- **Step 6 (Start)**: < 1ms
- **Step 7 (State)**: < 1ms
- **Step 8 (Placement)**: 500-2000ms ⭐ (slowest)
- **Step 9 (Analysis)**: 100-500ms
- **Step 10 (Playthrough)**: 10-20ms

**Total**: ~1-3 seconds

## Optimization Opportunities

### 1. Cache Reachability Results

```javascript
// Instead of recalculating every iteration
const reachabilityCache = new Map();
```

### 2. Parallel Processing (Future)

```javascript
// Process multiple seeds in parallel
await Promise.all(seeds.map(seed => processLogic(seed, settings)));
```

### 3. Incremental Graph Updates

```javascript
// Only recalculate changed portions of graph
graph.updateNode(nodeId, newData);
```

## Debugging Tips

### 1. Enable Verbose Logging

```javascript
window.MXF_DEBUG = true;

// In code:
if (window.MXF_DEBUG) {
  console.log('[DEBUG] Detailed info here');
}
```

### 2. Inspect Intermediate Results

```javascript
// After graph building
console.log('Graph:', JSON.stringify(graph, null, 2));

// After placement
console.log('Placements:', placements);

// After spheres
console.log('Spheres:', spheres);
```

### 3. Verify Specific Placement

```javascript
const placement = placements.find(p => p.item === "Morph Ball");
console.log('Morph Ball placed at:', placement.locationName);

// Check which sphere it's in
const sphere = spheres.find(s => 
  s.items.some(i => i.item === "Morph Ball")
);
console.log('In sphere:', sphere.index);
```

### 4. Test Reachability Manually

```javascript
const state = new GameState(settings);
state.addItem("Morph Ball");
state.addItem("Bombs");

const reachable = navigator.findReachableNodes(["1:1"], state);
console.log('Reachable nodes:', reachable);
```

## Next Steps

- Read [Graph-Builder.md](Graph-Builder.md) for graph construction details
- Check [Item-Placer.md](Item-Placer.md) for placement algorithm
- See [Progression-Analyzer.md](Progression-Analyzer.md) for sphere generation
- Review [Requirement-Evaluator.md](Requirement-Evaluator.md) for logic evaluation