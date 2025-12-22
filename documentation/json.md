# JSON Schema Guide for Beginners

## What Is JSON?

JSON (JavaScript Object Notation) is a **simple text format** for storing data. It's easy to read and write, even if you've never programmed before.

Think of JSON like filling out a form, but in a structured text format.

## Basic JSON Syntax

### Objects
Objects use **curly braces** `{}` and contain **key-value pairs**:

```json
{
  "name": "Morph Ball",
  "type": "equipment"
}
```

**Think of it like**:
```
Form:
  Name: Morph Ball
  Type: equipment
```

### Arrays
Arrays use **square brackets** `[]` and contain **lists of items**:

```json
["Morph Ball", "Bombs", "Speed Booster"]
```

**Think of it like**:
```
List:
  - Morph Ball
  - Bombs
  - Speed Booster
```

### Values
Values can be:
- **Strings** (text): `"Hello"`
- **Numbers**: `42` or `3.14`
- **Booleans**: `true` or `false`
- **null**: `null` (means "nothing")
- **Objects**: `{...}`
- **Arrays**: `[...]`

## JSON Rules

### 1. Keys Must Be Strings

✅ **Correct**:
```json
{
  "name": "Morph Ball"
}
```

❌ **Wrong** (missing quotes):
```json
{
  name: "Morph Ball"
}
```

### 2. Use Double Quotes

✅ **Correct**:
```json
{
  "name": "Morph Ball"
}
```

❌ **Wrong** (single quotes):
```json
{
  'name': 'Morph Ball'
}
```

### 3. Commas Between Items

✅ **Correct**:
```json
{
  "name": "Morph Ball",
  "type": "equipment"
}
```

❌ **Wrong** (missing comma):
```json
{
  "name": "Morph Ball"
  "type": "equipment"
}
```

### 4. NO Comma After Last Item

✅ **Correct**:
```json
{
  "name": "Morph Ball",
  "type": "equipment"
}
```

❌ **Wrong** (comma after last item):
```json
{
  "name": "Morph Ball",
  "type": "equipment",
}
```

### 5. Case Sensitive

```json
{
  "name": "Morph Ball"  // ← "name" is different from "Name"
}
```

`"name"` and `"Name"` are **NOT the same thing**!

## Common Patterns in MXF

### Pattern 1: Simple Object

```json
{
  "id": 1,
  "name": "Morph Ball"
}
```

**Reading It**:
- ID is 1
- Name is "Morph Ball"

### Pattern 2: Object with Array

```json
{
  "name": "Morph Ball",
  "requires": ["Power Suit", "Upgrade Station"]
}
```

**Reading It**:
- Name is "Morph Ball"
- Requires a list of 2 items: "Power Suit" and "Upgrade Station"

### Pattern 3: Array of Objects

```json
[
  {
    "id": 1,
    "name": "Morph Ball"
  },
  {
    "id": 2,
    "name": "Bombs"
  }
]
```

**Reading It**:
- This is a list
- First item: ID 1, name "Morph Ball"
- Second item: ID 2, name "Bombs"

### Pattern 4: Nested Objects

```json
{
  "name": "Enemy",
  "attacks": {
    "contact": 30,
    "projectile": 20
  }
}
```

**Reading It**:
- Name is "Enemy"
- Attacks is an object containing:
  - contact: 30
  - projectile: 20

### Pattern 5: Complex Requirements

```json
{
  "requires": [
    "Morph Ball",
    {
      "or": ["Bombs", "Power Bomb"]
    }
  ]
}
```

**Reading It**:
- Requires:
  1. Morph Ball (item)
  2. EITHER Bombs OR Power Bomb (choice)

## Understanding MXF Schemas

### What Is a Schema?

A **schema** is like a **template** or **rulebook** that defines:
- What fields are required
- What types of values are allowed
- How data should be structured

Think of it like a form that tells you:
- ✅ "This field is required"
- ✅ "This field must be a number"
- ✅ "This field is optional"

### Schema Reference

At the top of every JSON file:

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  ...
}
```

**What This Does**: Tells your editor which rulebook to follow, enabling auto-complete and error checking.

## Common JSON Mistakes

### Mistake 1: Trailing Comma

❌ **Wrong**:
```json
{
  "name": "Morph Ball",
  "type": "equipment",
}
```

✅ **Correct**:
```json
{
  "name": "Morph Ball",
  "type": "equipment"
}
```

### Mistake 2: Missing Quotes

❌ **Wrong**:
```json
{
  name: "Morph Ball"
}
```

✅ **Correct**:
```json
{
  "name": "Morph Ball"
}
```

### Mistake 3: Missing Comma

❌ **Wrong**:
```json
{
  "name": "Morph Ball"
  "type": "equipment"
}
```

✅ **Correct**:
```json
{
  "name": "Morph Ball",
  "type": "equipment"
}
```

### Mistake 4: Single Quotes

❌ **Wrong**:
```json
{
  'name': 'Morph Ball'
}
```

✅ **Correct**:
```json
{
  "name": "Morph Ball"
}
```

### Mistake 5: Comments (Not Allowed!)

❌ **Wrong**:
```json
{
  "name": "Morph Ball",  // This is the morph ball item
  "type": "equipment"
}
```

JSON **does not allow comments**. If you need notes, use a `"note"` field:

✅ **Correct**:
```json
{
  "name": "Morph Ball",
  "type": "equipment",
  "note": "This is the morph ball item"
}
```

## Reading MXF Files

### Example: Item File

```json
{
  "$schema": "../schema/mxf-items.schema.json",
  "startingRoom": "Revival Room",
  "startingItems": ["Ship"],
  "upgradeItems": [
    {
      "name": "Morph Ball",
      "wram": "0x09A4",
      "data": {
        "itemType": "equipment"
      }
    }
  ]
}
```

**Breaking It Down**:
1. Uses the items schema
2. Starting room is "Revival Room"
3. Starting items is a list with one item: "Ship"
4. Upgrade items is a list with one object:
   - Name: "Morph Ball"
   - Memory address: "0x09A4"
   - Data is an object:
     - Item type: "equipment"

### Example: Room File

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 1,
  "name": "First Room",
  "area": "MDK",
  "nodes": [
    {
      "id": 1,
      "name": "Left Door",
      "nodeType": "door"
    },
    {
      "id": 2,
      "name": "Item",
      "nodeType": "item"
    }
  ]
}
```

**Breaking It Down**:
1. Uses the room schema
2. ID is 1
3. Name is "First Room"
4. Area is "MDK"
5. Nodes is a list of 2 objects:
   - Node 1: ID 1, name "Left Door", type "door"
   - Node 2: ID 2, name "Item", type "item"

## Editing JSON Files

### Using a Text Editor

**Recommended Editors**:
- Visual Studio Code (free)
- Sublime Text
- Notepad++ (Windows)

### Tips for Editing

1. **Use Syntax Highlighting**: Makes it easier to see structure
2. **Use Auto-Complete**: Editors can suggest field names
3. **Validate Often**: Check for errors as you go
4. **Use Formatting**: Auto-format to keep it readable

### Visual Studio Code Tips

1. **Install JSON Schema Extension**:
   - Provides auto-complete
   - Shows errors in real-time

2. **Format Document**:
   - Right-click → "Format Document"
   - Or: Shift+Alt+F (Windows), Shift+Option+F (Mac)

3. **Collapse Sections**:
   - Click the arrows next to braces to collapse sections
   - Makes large files easier to navigate

## Hexadecimal Numbers

Many MXF files use **hexadecimal** (hex) numbers:

```json
"wram": "0x09A4"
```

**Understanding Hex**:
- Starts with `0x`
- Uses digits 0-9 and letters A-F
- `0x09A4` is a memory address

**You usually don't need to understand hex** - just copy the format!

## Required vs Optional Fields

### Required Fields

**Must** be present:

```json
{
  "id": 1,        // Required
  "name": "Item"  // Required
}
```

Missing required fields = **error**!

### Optional Fields

**Can** be omitted:

```json
{
  "id": 1,
  "name": "Item",
  "note": "Optional description"  // Optional
}
```

### How to Know?

Check the schema documentation:
- ✅ **Required**: Must include
- ⭕ **Optional**: Can skip

## Working with Arrays

### Adding Items

**Before**:
```json
"items": [
  "Morph Ball"
]
```

**After** (add "Bombs"):
```json
"items": [
  "Morph Ball",
  "Bombs"
]
```

**Remember**: Add comma after existing item!

### Removing Items

**Before**:
```json
"items": [
  "Morph Ball",
  "Bombs"
]
```

**After** (remove "Bombs"):
```json
"items": [
  "Morph Ball"
]
```

**Remember**: Remove comma if it's the last item!

## Working with Objects

### Adding Fields

**Before**:
```json
{
  "id": 1,
  "name": "Item"
}
```

**After** (add "type"):
```json
{
  "id": 1,
  "name": "Item",
  "type": "equipment"
}
```

### Removing Fields

**Before**:
```json
{
  "id": 1,
  "name": "Item",
  "type": "equipment"
}
```

**After** (remove "type"):
```json
{
  "id": 1,
  "name": "Item"
}
```

## Validating Your JSON

### Online Validators

Try these websites:
1. **JSONLint**: https://jsonlint.com/
2. **JSON Formatter**: https://jsonformatter.org/

**Steps**:
1. Copy your JSON
2. Paste into validator
3. Click "Validate"
4. Fix any errors shown

### Common Errors

**"Unexpected token }"**:
- Extra comma somewhere
- Missing comma somewhere

**"Unexpected string"**:
- Missing comma between items
- Missing quotes around key

**"Expected '}' but found ','**:
- Extra comma at end of object

## Practical Example: Adding a Room

Let's add a simple room step-by-step:

### Step 1: Start with Template

```json
{
  "$schema": "../schema/mxf-room.schema.json"
}
```

### Step 2: Add Required Fields

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 100,
  "name": "My New Room",
  "area": "MDK"
}
```

### Step 3: Add Room Environments

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 100,
  "name": "My New Room",
  "area": "MDK",
  "roomEnvironments": [
    {
      "heated": false
    }
  ]
}
```

### Step 4: Add Nodes

```json
{
  "$schema": "../schema/mxf-room.schema.json",
  "id": 100,
  "name": "My New Room",
  "area": "MDK",
  "roomEnvironments": [
    {
      "heated": false
    }
  ],
  "nodes": [
    {
      "id": 1,
      "name": "Left Door",
      "nodeType": "door"
    }
  ]
}
```

### Step 5: Continue Building

Add more nodes, links, strats, etc. following the same pattern!

## Tips for Success

### 1. Start Small
- Don't try to write a huge file at once
- Add one section at a time
- Validate after each addition

### 2. Copy Existing Examples
- Look at existing room/item files
- Copy structure
- Modify values

### 3. Use Comments (in separate file)
- Keep notes in a separate .txt file
- JSON doesn't allow comments
- Reference notes while editing

### 4. Test Frequently
- Validate JSON after changes
- Load in randomizer to test
- Fix errors immediately

### 5. Ask for Help
- Post in Discord/forum
- Share your JSON
- Community can help debug

## Quick Reference Card

```
JSON Basics:
  Object: { "key": "value" }
  Array: ["item1", "item2"]
  String: "text"
  Number: 42
  Boolean: true or false
  null: null

Rules:
  ✓ Double quotes only
  ✓ Commas between items
  ✗ NO comma after last item
  ✗ NO comments allowed
  ✓ Keys must be strings
  
Common Patterns:
  List of objects: [{...}, {...}]
  Nested object: {"key": {...}}
  OR condition: {"or": [...]}
  AND condition: ["item1", "item2"]
```

## Next Steps

Now that you understand JSON basics:

1. **Read Schema Guides**:
   - [Room-Schema.md](Room-Schema.md)
   - [Items-Schema.md](Items-Schema.md)
   - [Requirements-Schema.md](Requirements-Schema.md)

2. **Study Examples**:
   - Look at existing JSON files
   - Understand the patterns

3. **Start Small**:
   - Edit existing files first
   - Then create new simple files
   - Graduate to complex files

4. **Get Help**:
   - Community is here to help
   - Don't be afraid to ask questions!