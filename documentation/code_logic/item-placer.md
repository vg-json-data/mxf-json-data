# Item Placer Guide

## Overview

The Item Placer implements the **Assumed Fill** algorithm to place items in locations throughout the game world while ensuring the seed remains beatable.

## Source File

`logic/placer.js`

## Assumed Fill Algorithm

### Core Concept

Instead of checking "can we reach this location with current items?" (forward fill), Assumed Fill works backwards:

1. **Assume you have everything**
2. **Place items one by one, removing them from your inventory**
3. **Check that each placement is reachable with remaining items**

### Why Assumed Fill?

✅ **Guarantees Beatable Seeds**: If you can reach everywhere with all items, removing items only makes placements "easier" to reach

✅ **Faster**: No need to retry failed placements

✅ **Better Distribution**: More natural item spread across the world

✅ **No Backtracking**: Each placement is final

## Algorithm Flow

```
┌─────────────────────────────────────┐
│   1. SPECIAL CATEGORIES             │
│   - Aux Power Cells                 │
│   - Etecoon                          │
│   - Reserve-X Items                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   2. FILTER USED LOCATIONS          │
│   Remove locations used by special  │
│   categories from standard pool     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   3. SPLIT ITEMS                    │
│   - Major Items (progression)       │
│   - Filler Items (expansions)       │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   4. PLACE MAJOR ITEMS              │
│   (Critical fix - majors first!)    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   5. PLACE FILLER ITEMS             │
│   Fill remaining locations          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   OUTPUT: Complete Placements       │
└─────────────────────────────────────┘
```

## Step-by-Step Breakdown

### Step 1: Special Categories

Some items have restricted placement rules:

#### Aux Power Cells

```javascript
// Aux Power Cells stay in their original locations but shuffled
const auxLocations = locationPool.auxPowerCell;
const auxItems = auxLocations.map(loc => loc.originalItem);

shuffle(auxLocations);
shuffle(auxItems);

for (let i = 0; i < auxLocations.length; i++) {
  place(auxItems[i], auxLocations[i]);
}
```

**Why?** Aux Power Cells are special progression markers that should stay in their designated spots.

#### Etecoon

```javascript
// Etecoon locations also stay internal but shuffle
const etecoonLocations = locationPool.etecoon;
const etecoonItems = etecoonLocations.map(loc => loc.originalItem);

shuffle(etecoonLocations);
shuffle(etecoonItems);

for (let i = 0; i < etecoonLocations.length; i++) {
  place(etecoonItems[i], etecoonLocations[i]);
}
```

**Why?** Etecoon items have special story significance.

#### Reserve-X Items

```javascript
// One Reserve-X per sector (no duplicate sectors)
const usedSectors = new Set();

for (const reserveXItem of reserveXItems) {
  const availableSectors = sectors.filter(s => !usedSectors.has(s));
  const sector = random.choice(availableSectors);
  
  const locationsInSector = locationsBySector[sector]
    .filter(loc => !isSpecial(loc));
  
  const location = random.choice(locationsInSector);
  place(reserveXItem, location);
  usedSectors.add(sector);
}
```

**Why?** Reserve-X items should be spread across the map, one per sector.

### Step 2: Filter Used Locations

```javascript
const usedNodes = new Set(
  placements.map(p => p.locationNodeId)
);

availableLocations = locationPool.standard.filter(
  loc => !usedNodes.has(loc.nodeId)
);
```

Remove any locations already used by special categories.

### Step 3: Split Items

Items are categorized into two groups:

```javascript
function isMajorItem(item) {
  // Progressive items are always major
  if (item.startsWith('Progressive ')) {
    return true;
  }
  
  // Pure expansions are filler
  const fillerExact = ['ETank', 'Missile', 'PowerBomb'];
  if (fillerExact.includes(item)) {
    return false;
  }
  
  // Everything else is major (discrete upgrades)
  return true;
}
```

**Major Items** (Progression):
- `Morph Ball`
- `Bombs`
- `Speed Booster`
- `Hi-Jump`
- `Missiles` (the first one)
- `Super Missiles` (the first one)
- `Progressive Speed Booster`
- etc.

**Filler Items** (Non-Progression):
- `ETank`
- `Missile` (expansion)
- `PowerBomb` (expansion)

### Step 4: Place Major Items (CRITICAL)

**This is the most important fix**: Major items MUST be placed before filler items.

```javascript
function placeItemSetAssumedFill(label, items, locations, startNodes, state, placements) {
  let iterations = 0;
  const maxIterations = items.length * 5;
  
  while (items.length > 0 && locations.length > 0 && iterations < maxIterations) {
    iterations++;
    
    // Find reachable locations with CURRENT state
    const reachable = navigator.findReachableNodes(startNodes, state);
    const reachableLocations = locations.filter(loc =>
      reachable.includes(loc.nodeId)
    );
    
    // Choose a random reachable location
    const location = reachableLocations.length > 0
      ? reachableLocations[rng.int(0, reachableLocations.length - 1)]
      : locations[0];  // Fallback to any location
    
    // Remove location from pool
    const locIdx = locations.findIndex(l => l.nodeId === location.nodeId);
    if (locIdx >= 0) locations.splice(locIdx, 1);
    
    // Place next item
    const item = items.shift();
    placements.push(createPlacement(item, location, label));
    
    // Add item to state for next iteration
    state.addItem(item);
  }
}
```

**Why Major Items First?**
- Major items unlock new areas
- Placing fillers first could "waste" early locations
- This ensures progression items appear early enough

### Step 5: Place Filler Items

Same process as major items:

```javascript
placeItemSetAssumedFill(
  'filler',
  fillerItems,
  availableLocations,  // Recomputed after major placement
  startNodes,
  state,
  placements
);
```

## State Management

### Initial State

```javascript
const initialState = new GameState(settings);

// Add starting items
for (const itemName of startingItems) {
  initialState.addItem(itemName);
}
```

### State Updates During Placement

```javascript
// After placing an item
state.addItem(item);

// State now reflects we have this item
// Future reachability checks will include it
```

### State Cloning for Simulation

```javascript
// Don't modify original state during checks
const testState = state.clone();
testState.addItem('Morph Ball');

const reachable = navigator.findReachableNodes(startNodes, testState);
```

## Reachability Checks

```javascript
const reachable = navigator.findReachableNodes(startNodes, state);
const reachableLocations = locations.filter(loc =>
  reachable.includes(loc.nodeId)
);
```

**Process**:
1. Start at initial nodes
2. Use BFS to find all reachable nodes
3. Filter locations to only reachable ones
4. Choose randomly from reachable locations

## Placement Object

```javascript
{
  item: "Morph Ball",
  locationNodeId: "42:2",
  locationName: "Item",
  note: "Hidden behind blocks",
  roomId: 42,
  roomName: "Morph Ball Room",
  roomArea: "MDK",
  originalItem: "Morph Ball",
  isProgression: true,
  category: "major"
}
```

## Safety Mechanisms

### 1. Iteration Limit

```javascript
const maxIterations = items.length * 5;

if (iterations >= maxIterations) {
  console.warn('Hit iteration limit - some items may not be placed');
  break;
}
```

Prevents infinite loops.

### 2. Fallback Placement

```javascript
const location = reachableLocations.length > 0
  ? reachableLocations[rng.int(0, reachableLocations.length - 1)]
  : locations[0];  // Fallback to first available location
```

If no locations are reachable (shouldn't happen), place anyway.

### 3. Warning for Unplaced Items

```javascript
if (items.length > 0) {
  console.warn(`⚠ ${items.length} ${label} items could not be placed`);
}
```

## Randomization

### Deterministic RNG

```javascript
const seed = generateSeed(settings);
const rng = new RNG(seed);

// Shuffle items
rng.shuffle(majorItems);
rng.shuffle(fillerItems);

// Choose location
const idx = rng.int(0, locations.length - 1);
const location = locations[idx];
```

Same seed = same placements.

### Shuffling

```javascript
rng.shuffle(items);
```

Uses Fisher-Yates shuffle with seeded RNG.

## Example Walkthrough

Let's trace placing 3 items with Assumed Fill:

### Initial Setup

```javascript
items = ['Morph Ball', 'Bombs', 'Missiles'];
locations = ['Loc1', 'Loc2', 'Loc3', 'Loc4', 'Loc5'];
state = { items: ['Ship'] };
```

### Iteration 1

```javascript
// Check reachability
reachable = findReachableNodes(['ship'], state);
// Result: ['Loc1', 'Loc2'] (only starting area)

// Choose random reachable location
location = random.choice(['Loc1', 'Loc2']);
// Let's say: 'Loc1'

// Place item
place('Morph Ball', 'Loc1');

// Update state
state.addItem('Morph Ball');

// Update pools
items = ['Bombs', 'Missiles'];
locations = ['Loc2', 'Loc3', 'Loc4', 'Loc5'];
```

### Iteration 2

```javascript
// Check reachability (now with Morph Ball)
reachable = findReachableNodes(['ship'], state);
// Result: ['Loc1', 'Loc2', 'Loc3', 'Loc4'] (Morph Ball opened more areas)

// Choose random reachable location
location = random.choice(['Loc2', 'Loc3', 'Loc4']);
// Let's say: 'Loc3'

// Place item
place('Bombs', 'Loc3');

// Update state
state.addItem('Bombs');

// Update pools
items = ['Missiles'];
locations = ['Loc2', 'Loc4', 'Loc5'];
```

### Iteration 3

```javascript
// Check reachability (now with Morph Ball + Bombs)
reachable = findReachableNodes(['ship'], state);
// Result: ['Loc1', 'Loc2', 'Loc3', 'Loc4', 'Loc5'] (all locations)

// Choose random reachable location
location = random.choice(['Loc2', 'Loc4', 'Loc5']);
// Let's say: 'Loc5'

// Place item
place('Missiles', 'Loc5');

// Update state
state.addItem('Missiles');

// Update pools
items = [];
locations = ['Loc2', 'Loc4'];
```

### Final Placements

```javascript
[
  { item: 'Morph Ball', location: 'Loc1', category: 'major' },
  { item: 'Bombs', location: 'Loc3', category: 'major' },
  { item: 'Missiles', location: 'Loc5', category: 'major' }
]
```

## Debugging

### Check Placements

```javascript
console.log(`Placed ${placements.length} items`);

placements.forEach(p => {
  console.log(`${p.item} → [${p.roomName}] ${p.locationName}`);
});
```

### Verify Reachability

```javascript
const spheres = analyzer.analyzeWithPlacements(startNodes, initialState, placements);

spheres.forEach((sphere, idx) => {
  console.log(`Sphere ${idx}: ${sphere.items.length} items`);
  sphere.items.forEach(item => {
    console.log(`  - ${item.item}`);
  });
});
```

### Check Unplaced Items

```javascript
if (items.length > 0) {
  console.warn('Unplaced items:', items);
}
```

## Common Issues

### 1. Major Items Placed Too Late

**Problem**: Progression items appear in late-game spheres

**Solution**: Ensure major items are placed before filler items (already implemented)

### 2. Locations Not Reachable

**Problem**: Some locations never become reachable

**Solution**:
- Check graph connections
- Verify requirement logic
- Ensure starting items are correct

### 3. Special Items in Wrong Category

**Problem**: Items placed in wrong category (e.g., Aux Power in standard pool)

**Solution**:
```javascript
// Ensure special items are filtered correctly
if (loc.nodeName.startsWith('Aux Power Cell')) {
  locationPool.auxPowerCell.push(loc);
} else {
  locationPool.standard.push(loc);
}
```

## Best Practices

### 1. Always Place Major Items First

```javascript
// ✅ Correct order
placeItemSetAssumedFill('major', majorItems, ...);
placeItemSetAssumedFill('filler', fillerItems, ...);

// ❌ Wrong order
placeItemSetAssumedFill('filler', fillerItems, ...);
placeItemSetAssumedFill('major', majorItems, ...);
```

### 2. Verify Settings

```javascript
const settings = {
  etankCount: 14,
  missileCount: 47,
  powerBombCount: 25,
  restrictSpecialItemSwaps: true  // Important!
};
```

### 3. Test Edge Cases

- Minimal item count
- Maximum item count
- Different starting locations
- Different difficulty settings

### 4. Log Progress

```javascript
console.log('[ItemPlacer] Starting assumed fill');
console.log(`  Major items: ${majorItems.length}`);
console.log(`  Filler items: ${fillerItems.length}`);
```

## Performance

- **Time Complexity**: O(N * (V + E)) where N = items, V = nodes, E = edges
- **Space Complexity**: O(N + L) where N = items, L = locations
- **Typical Runtime**: 1-5 seconds for ~100 items

## Next Steps

- Read [Progression-Analyzer.md](Progression-Analyzer.md) for sphere analysis
- Check [Graph-Navigator.md](Graph-Navigator.md) for reachability logic
- See [Game-State.md](Game-State.md) for state management