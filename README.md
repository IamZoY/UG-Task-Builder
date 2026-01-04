# UG Task Builder - Complete Documentation

A visual script builder for creating automated bank-standing and skilling bots without writing code.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Interface Overview](#interface-overview)
3. [Script Types Reference](#script-types-reference)
4. [Conditions System](#conditions-system)
5. [Building Your First Bot](#building-your-first-bot)
6. [Preset Scripts](#preset-scripts)
7. [Detailed Script Guides](#detailed-script-guides)
8. [Advanced Usage](#advanced-usage)
9. [Tips & Best Practices](#tips--best-practices)

---

## Getting Started

### What is UG Task Builder?

UG Task Builder allows you to create automated scripts by combining **tasks** in a visual interface. Each task performs a specific action (like opening the bank, withdrawing items, or clicking menus), and you can add **conditions** to control when each task runs.

### Basic Concepts

- **Script**: A collection of tasks that run in sequence
- **Task**: A single action (e.g., "Open Bank", "Withdraw Item")
- **Condition**: A rule that must be true for a task to execute
- **Loop**: Scripts can repeat indefinitely when "Loop" is enabled

---

## Interface Overview

### Main Panel

```
┌─────────────────────────────────┐
│  UG Task Builder          [Status] │
├─────────────────────────────────┤
│  [START]                        │
├─────────────────────────────────┤
│  [+] [📁] [↓]           Loop [○]│
├─────────────────────────────────┤
│  ┌─ Script Name (3) ──────[○]─┐ │
│  │  [Edit] [Export] [Delete]  │ │
│  └────────────────────────────┘ │
│                                 │
│  ┌─ Another Script (5) ───[○]─┐ │
│  │  [Edit] [Export] [Delete]  │ │
│  └────────────────────────────┘ │
└─────────────────────────────────┘
```

### Toolbar Buttons

| Button | Function |
|--------|----------|
| **+** | Add a new empty script |
| **📁** | Open presets menu |
| **↓** | Import script from file |
| **Loop** | Toggle script looping |

### Script Controls

| Button | Function |
|--------|----------|
| **Edit** | Open task editor for this script |
| **Export** | Save script to JSON file |
| **Delete** | Remove script |
| **Toggle** | Enable/disable script |

---

## Script Types Reference

### Bank Operations

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `Open Bank` | Opens the nearest bank booth/chest | None |
| `Close Bank` | Closes the bank interface | None |
| `Deposit All` | Deposits all items (or specific item if named) | Item Name (optional) |
| `Deposit Item` | Deposits X amount of an item | Item Name, Quantity |
| `Withdraw Item` | Withdraws X amount from bank | Item Name, Quantity |

**Quantity Values:**
- Positive number: Withdraw/deposit that exact amount
- `-1`: Withdraw/deposit ALL

---

### Item Operations

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `Use Item on Item` | Combines two items (e.g., herb + vial) | Item 1, Item 2 |
| `Use Item on Object` | Uses item on game object | Item Name, Target Name, Action |
| `Drop Item` | Drops an item from inventory | Item Name |
| `Equip Item` | Equips an item (Wear/Wield/Equip) | Item Name |

---

### Interaction Operations

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `Click Object` | Interacts with a game object | Target Name, Action |
| `Click NPC` | Interacts with an NPC | Target Name, Action |
| `Click Widget` | Clicks a specific interface element | Widget Info |
| `Select Menu Option` | Clicks skill menu option (Make/Cook/etc.) | Option Number |

**Menu Option Values:**
- `1` = First option (widget 15)
- `2` = Second option (widget 16)
- etc.

---

### Spell Operations

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `Cast Spell` | Casts a spell (no target) | Spell Name |
| `Cast Spell on Item` | Casts spell on inventory item | Spell Name, Item Name |

**Common Spell Names:**
- `High Level Alchemy`
- `Plank Make`
- `Superheat Item`
- `Enchant Crossbow Bolt`

---

### Wait/Delay Operations

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `Wait Ticks` | Pauses for X game ticks (1 tick = 600ms) | Quantity (ticks) |
| `Wait Animation` | Waits while player is animating | Max ticks (optional) |
| `Wait Idle` | Waits until player stops animating | None |
| `Walk To` | Walks to coordinates and waits until arrived | X, Y, Z coordinates |

---

### Conditional Checks (Simple)

These return true/false and can be used with conditions:

| Type | Description |
|------|-------------|
| `If Has Item` | True if inventory contains item |
| `If Bank Open` | True if bank interface is open |
| `If Animating` | True if player is currently animating |

---

## Conditions System

### What Are Conditions?

Conditions control **when** a task executes. A task with conditions will only run if **ALL** conditions are true.

### Available Condition Types

| Condition | Description | Needs Item? | Needs Qty? |
|-----------|-------------|-------------|------------|
| `Has Item` | Inventory contains item | ✅ | ❌ |
| `No Item` | Inventory does NOT contain item | ✅ | ❌ |
| `Bank Open` | Bank interface is open | ❌ | ❌ |
| `Bank Closed` | Bank interface is closed | ❌ | ❌ |
| `Animating` | Player is animating | ❌ | ❌ |
| `Idle` | Player is NOT animating | ❌ | ❌ |
| `Inv Full` | Inventory has 28 items | ❌ | ❌ |
| `Inv Empty` | Inventory has 0 items | ❌ | ❌ |
| `Inv Count` | Has X or more of item | ✅ | ✅ |
| `Menu Open` | Skill menu is visible | ❌ | ❌ |

### The NOT Modifier

Each condition can be **inverted** using the NOT checkbox:
- `Has Item` → True if you HAVE the item
- `NOT Has Item` → True if you DON'T have the item

### Multiple Conditions

When a task has multiple conditions, **ALL must be true**:

```
Task: Withdraw Herbs
Conditions:
  - Bank Open ✅
  - Has Item: "Vial of water" ✅  
  - NOT Has Item: "Guam leaf" ✅
```

This task only runs when:
1. Bank is open, AND
2. You have vials, AND
3. You don't have herbs

---

## Building Your First Bot

### Example: Potion Making Bot

Let's create a bot that makes Attack Potions (Guam potion (unf) + Eye of newt).

#### Step 1: Create New Script

1. Click **+** to create a new script
2. Name it "Attack Potions"
3. Click **Edit** to open the task editor

#### Step 2: Add Bank Tasks

**Task 1: Open Bank**
```
Type: Open Bank
Conditions:
  - Bank Closed
  - NOT Has Item: "Guam potion (unf)"
```

**Task 2: Deposit All**
```
Type: Deposit All
Conditions:
  - NOT Has Item: "Guam potion (unf)"
```

**Task 3: Withdraw Unf Potions**
```
Type: Withdraw Item
Item Name: "Guam potion (unf)"
Quantity: 14
Conditions:
  - Bank Open
  - NOT Has Item: "Guam potion (unf)"
```

**Task 4: Withdraw Secondary**
```
Type: Withdraw Item
Item Name: "Eye of newt"
Quantity: 14
Conditions:
  - Bank Open
  - Has Item: "Guam potion (unf)"
  - NOT Has Item: "Eye of newt"
```

**Task 5: Close Bank**
```
Type: Close Bank
Conditions:
  - Bank Open
  - Has Item: "Guam potion (unf)"
  - Has Item: "Eye of newt"
```

#### Step 3: Add Making Tasks

**Task 6: Use Items**
```
Type: Use Item on Item
Item 1: "Guam potion (unf)"
Item 2: "Eye of newt"
Conditions:
  - Bank Closed
  - Has Item: "Guam potion (unf)"
  - Has Item: "Eye of newt"
  - NOT Menu Open
  - Idle
```

**Task 7: Wait for Animation**
```
Type: Wait Animation
Quantity: 25 (max ticks to wait)
```

**Task 8: Select Menu**
```
Type: Select Menu Option
Option: "1" (first option)
Conditions:
  - Menu Open
```

#### Step 4: Enable and Start

1. Go back to the main panel
2. Enable the script (toggle ON)
3. Enable **Loop**
4. Click **START**

---

## Preset Scripts

The **Presets** menu contains ready-made scripts for common activities:

### Herblore
- Attack Potion, Strength Potion, Prayer Potion
- Super Attack, Super Strength, Super Restore
- Saradomin Brew

### Unf Potions
- Guam, Ranarr, Toadflax, Snapdragon

### Crafting
- Green/Blue/Black D'hide Bodies

### Gem Cutting
- Sapphires, Emeralds, Rubies, Diamonds

### Fletching
- Arrow Shafts, Yew Longbows, Magic Longbows

### Bow Stringing
- Yew Longbows, Magic Longbows

### Magic
- Plank Make (Oak/Mahogany), High Alch

### Battlestaves
- Air, Water, Earth, Fire

### Glass Blowing
- Unpowered Orbs, Lantern Lens

---

## Detailed Script Guides

For in-depth documentation on each preset category including task breakdowns, XP rates, profit analysis, and tips, see the individual guides:

| Guide | Description | Link |
|-------|-------------|------|
| **Herblore** | Making finished potions from unf potions | [scripts/Herblore.md](scripts/Herblore.md) |
| **Unfinished Potions** | Making unf potions from herbs + vials | [scripts/UnfPotions.md](scripts/UnfPotions.md) |
| **Crafting (Leather)** | Crafting dragonhide bodies | [scripts/Crafting.md](scripts/Crafting.md) |
| **Gem Cutting** | Cutting uncut gems | [scripts/GemCutting.md](scripts/GemCutting.md) |
| **Fletching** | Cutting logs into bows | [scripts/Fletching.md](scripts/Fletching.md) |
| **Bow Stringing** | Stringing unstrung bows | [scripts/BowStringing.md](scripts/BowStringing.md) |
| **Magic** | Plank Make & High Alchemy | [scripts/Magic.md](scripts/Magic.md) |
| **Battlestaves** | Attaching orbs to staves | [scripts/Battlestaves.md](scripts/Battlestaves.md) |
| **Glass Blowing** | Blowing molten glass | [scripts/GlassBlowing.md](scripts/GlassBlowing.md) |

### What's in Each Guide?

Each detailed guide includes:

- **Full Task Breakdown** - Every task with exact settings and conditions
- **Flow Diagrams** - Visual representation of script logic
- **XP Rates** - Estimated experience per hour
- **Profit Analysis** - Cost breakdown and profit margins
- **Tips & Tricks** - Expert advice for each activity
- **Requirements** - Levels, items, and setup needed

### Example: Reading a Task Breakdown

```
Task 3: Withdraw Herbs
├─ Type: Withdraw Item
├─ Item: "Guam leaf"
├─ Quantity: 14
├─ Delay: 2 ticks
└─ Conditions:
    ├─ Bank Open
    └─ NOT Has Item: "Guam leaf"
```

This format shows:
- The task type and configuration
- Required item names and quantities
- Delay settings
- All conditions that must be true

---

## Advanced Usage

### Walk To Task

The `Walk To` task allows your bot to walk to specific coordinates:

```
Type: Walk To
X: 3164
Y: 3487
Z: 0 (ground floor)
```

The bot will:
1. Start walking to the destination
2. Wait until it arrives (within 2 tiles)
3. Continue to the next task

**Use Cases:**
- Walking to a bank from a crafting station
- Moving between resource spots
- Navigating complex areas

**Getting Coordinates:**
- Use the Developer Tools plugin
- Check the wiki for location coordinates

### Delay Settings

Every task has delay settings:

| Setting | Description |
|---------|-------------|
| **Delay** | Base delay after task completes |
| **Ticks** | If checked, delay is in game ticks (600ms each) |
| **Randomize %** | Adds random variance (e.g., 15% = ±15%) |

Example: Delay 2, Ticks ✅, Randomize 20%
- Base: 1200ms (2 ticks)
- Range: 960ms - 1440ms

### Combining Multiple Scripts

You can have multiple scripts enabled at once. They execute in order from top to bottom.

**Example Setup:**
1. Script A: Potion Making (enabled)
2. Script B: Emergency Banking (disabled - enable manually)
3. Script C: High Alch Profits (enabled)

---

## Tips & Best Practices

### 1. Always Use Conditions

❌ **Bad:**
```
Task: Withdraw Herbs (no conditions)
```

✅ **Good:**
```
Task: Withdraw Herbs
Conditions:
  - Bank Open
  - NOT Has Item: "Herb"
```

### 2. Order Matters

Tasks execute top-to-bottom. Put banking tasks BEFORE crafting tasks.

### 3. Wait for Animations

Always add `Wait Animation` before menu selections to prevent spam-clicking.

### 4. Use Idle Condition

Add `Idle` condition to "Use Item" tasks to prevent interrupting ongoing actions.

### 5. Test Incrementally

1. Add 2-3 tasks
2. Test manually
3. Add more tasks
4. Repeat

### 6. Export Working Scripts

Once a script works, **Export** it as a backup!

### 7. Delay Recommendations

| Action | Recommended Delay |
|--------|-------------------|
| Bank Open | 2 ticks |
| Deposit/Withdraw | 1-2 ticks |
| Close Bank | 1 tick |
| Use Item | 1 tick |
| Menu Select | 1 tick |
| Wait Animation | 20-40 ticks (depends on activity) |

---

## Troubleshooting

### Bot Does Nothing

- Check if script is **enabled** (toggle ON)
- Check if **Loop** is enabled
- Verify conditions are correct
- Make sure you're logged in

### Task Spam / Clicking Too Fast

- Add `Wait Animation` before menu clicks
- Add `Idle` condition to action tasks
- Increase delays

### Bot Opens Bank Repeatedly

- Add `NOT Has Item` condition to bank opening task
- Make sure withdraw tasks have proper conditions

### Menu Not Clicking

- Verify `Menu Open` condition is set
- Check the option number is correct (1, 2, 3...)
- Add wait animation before menu task

---

## Quick Reference Card

### Common Task Patterns

**Banking Pattern:**
```
1. Open Bank     [Bank Closed, No Items]
2. Deposit All   [No Items]
3. Withdraw A    [Bank Open, No A]
4. Withdraw B    [Bank Open, Has A, No B]
5. Close Bank    [Bank Open, Has A, Has B]
```

**Making Pattern:**
```
6. Use Items     [Bank Closed, Has A, Has B, Menu Closed, Idle]
7. Wait Anim     [no conditions]
8. Select Menu   [Menu Open]
```

**Walking Pattern:**
```
1. Walk To Bank  [No Items]
2. [Banking tasks...]
3. Walk To Spot  [Has Items]
4. [Action tasks...]
```

---

## Support

For help and support:
- Ask in the Discord support channel
- Share your script JSON for debugging
- Include error messages if any

Happy botting! 🤖
