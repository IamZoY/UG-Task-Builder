# UG Task Builder

A visual script builder for creating automated bank-standing and skilling workflows. Build scripts by combining tasks and conditions in a drag-and-drop interface — no coding required. Ideal for herblore, crafting, fletching, gem cutting, magic training, and other repetitive activities.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Key Concepts](#2-key-concepts)
3. [Interface Overview](#3-interface-overview)
4. [Available Task Types](#4-available-task-types)
5. [Available Conditions](#5-available-conditions)
6. [Delays, Ticks, and Timing](#6-delays-ticks-and-timing)
7. [Wait Tasks Explained](#7-wait-tasks-explained)
8. [Building a Script](#8-building-a-script)
9. [Example Script Patterns](#9-example-script-patterns)
10. [Presets](#10-presets)
11. [Detailed Script Guides](#11-detailed-script-guides)
12. [Importing and Exporting Scripts](#12-importing-and-exporting-scripts)
13. [Tips and Best Practices](#13-tips-and-best-practices)
14. [Troubleshooting](#14-troubleshooting)
15. [FAQ](#15-faq)
16. [Support](#16-support)

---

## 1. Overview

UG Task Builder lets you create **scripts** that run a sequence of **tasks** in order. Each task performs a single action — such as opening the bank, withdrawing items, using items together, or selecting a menu option. You add **conditions** to control when each task runs, so the script only acts when the right game state is met.

Scripts run from top to bottom. When **Loop** is enabled, the script repeats from the beginning after the last task finishes. This makes it easy to automate bank-standing activities that follow a fixed pattern: bank, gather materials, craft, repeat.

---

## 2. Key Concepts

| Term | Meaning |
|------|---------|
| **Script** | A named collection of tasks. You can have multiple scripts; enabled ones run in order. |
| **Task** | A single action the plugin performs (e.g., Open Bank, Withdraw Item, Use Item on Item). |
| **Condition** | A rule attached to a task. The task only runs when all its conditions are true. |
| **Conditional Block** | A task that contains child tasks inside it. The children only run when the block's condition is true. |
| **Loop** | When enabled, scripts restart from the first task after the last one finishes. |
| **Delay** | Time to wait after a task completes before the next one runs. Can be in ticks or milliseconds. |
| **Grace Period** | Extra time given for an animation to start before deciding the player is idle. Prevents false "not animating" detection. |
| **Preset** | A ready-made script template you can add and then customize. |
| **Import** | Load a script from a JSON file (e.g., a backup or shared script). |
| **Export** | Save a script to a JSON file for backup or sharing. |

---

## 3. Interface Overview

### Main Panel

The side panel shows your scripts and controls:

| Control | What it does |
|---------|--------------|
| **START / STOP** | Starts or stops running all enabled scripts. |
| **+** | Create a new empty script. |
| **Presets** (folder icon) | Choose from ready-made script templates. |
| **Import** (down arrow) | Load a script from a JSON file. |
| **Loop** toggle | When ON, scripts repeat from the start after finishing. |

### Per-Script Controls

Each script in the list has:

| Control | What it does |
|---------|--------------|
| **Toggle** (on/off) | Enable or disable the script. Disabled scripts are skipped during execution. |
| **Edit** | Open the task editor to view and modify the script's tasks. |
| **Export** | Save the script as a JSON file. |
| **Delete** | Remove the script (asks for confirmation). |

### Task Editor

When you click **Edit** on a script, you see its task list:

| Control | What it does |
|---------|--------------|
| **Add Task** (+) | Add a new task and configure its type, fields, and conditions. |
| **Clear All** (X) | Remove all tasks (asks for confirmation). |
| **Move Up / Move Down** | Reorder tasks. |
| **Edit** (pencil) | Open the task editor dialog to change a task's settings. |
| **Delete** (X) | Remove a single task (asks for confirmation). |
| **Back** | Return to the script list. |

### In-Game Overlay

While running, an overlay shows:

- **Time** — How long the script has been running.
- **Status** — What the plugin is currently doing.
- **Task** — The name of the task currently running.
- **Progress** — Which task out of the total (e.g., 3/8).
- **Cycles** — How many full loops have completed.

---

## 4. Available Task Types

### Bank Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Open Bank** | Opens the nearest bank booth or chest. | None required. |
| **Close Bank** | Closes the bank interface. | None required. |
| **Deposit All** | Deposits all items, or all of a specific item if you name one. | Item Name (optional). |
| **Deposit Item** | Deposits a set amount of a specific item. | Item Name, Quantity. |
| **Withdraw Item** | Withdraws a set amount of an item from the bank. | Item Name, Quantity. |

**Quantity for bank tasks:**
- A positive number (e.g., `14`) = that exact amount.
- `-1` = all of that item.

---

### Item Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Use Item on Item** | Uses one item on another (e.g., herb on vial). | Item 1, Item 2. |
| **Use Item on Object** | Uses an item on a game object (e.g., ore on furnace). | Item Name, Object Name, Action (optional). |
| **Drop Item** | Drops an item from inventory. | Item Name. |
| **Equip Item** | Equips an item (tries Wear, Wield, and Equip actions). | Item Name. |
| **Inventory Click** | Clicks an inventory item with a specific action. | Item Name, Action (optional). If no action is specified, the first available action is used. |

**Item name matching:**
- Names are **case-insensitive** and **ignore spaces**.
- **Wildcards** are supported: `rune*` matches "Rune platebody", "Rune scimitar", etc. `*potion*` matches anything containing "potion".
- You can also use **numeric IDs** (e.g., `995` for coins) for Inventory Click tasks, and for NPC/Object conditions.

---

### Interaction Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Click Object** | Interacts with a game object (e.g., a tree, furnace, altar). | Object Name, Action (optional). |
| **Click NPC** | Interacts with an NPC (e.g., a banker, shopkeeper). | NPC Name, Action (optional). |
| **Click Widget** | Clicks a specific interface element. | Widget Info (in `group:child` format). |

Object and NPC names support **wildcards** (e.g., `Bank*` matches "Bank booth" and "Bank chest") and are case-insensitive.

If no action is specified for Click Object or Click NPC, the first available action is used.

---

### Spell Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Cast Spell** | Casts a spell with no target. | Spell Name. |
| **Cast Spell on Item** | Casts a spell on an inventory item. | Spell Name, Item Name. |

**Common spell names:** High Level Alchemy, Plank Make, Superheat Item, Enchant Crossbow Bolt.

---

### Wait and Delay Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Wait Ticks** | Pauses for a number of game ticks. | Wait duration (ticks). |
| **Wait Animation** | Waits while the player is animating, with a max timeout. | Anim timeout (ticks), Grace period. |
| **Wait Idle** | Waits until the player stops animating. | Grace period. |
| **Wait Animation Start** | Waits for an animation to begin (use right after clicking). | Grace period. |
| **Wait Animation Cycle** | Waits for a complete animation (start then stop). | Grace period. |

See [Section 7 — Wait Tasks Explained](#7-wait-tasks-explained) for detailed guidance on when to use each.

---

### Movement Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Walk To** | Walks to coordinates and waits until arrived (within 2 tiles). | X, Y, Z (plane). |

Z is the game floor (0 = ground, 1 = first floor, 2 = second floor, 3 = third floor).

**Getting coordinates:** Use the Developer Tools plugin or check the wiki for location coordinates.

---

### Menu and Dialog Actions

| Task | What it does | Fields to set |
|------|--------------|---------------|
| **Select Menu Option** | Clicks an option in the skill/make menu (Make, Cook, Cut, etc.). | Option Number. |
| **Custom Action** | Runs a custom command string (advanced use). | Custom action string. |

**Option Number for Select Menu Option:**
- `1` = first option
- `2` = second option
- `3` = third option
- And so on.

---

### Conditional Blocks

Conditional blocks are special tasks that contain **child tasks** inside them. The children only execute when the block's condition is true.

| Conditional Block | When children run |
|-------------------|-------------------|
| **Condition: Has Item** | When inventory contains the named item. |
| **Condition: No Item** | When inventory does NOT contain the named item. |
| **Condition: Bank Open** | When the bank interface is open. |
| **Condition: Bank Closed** | When the bank interface is closed. |
| **Condition: Animating** | When the player is currently animating. |
| **Condition: Idle** | When the player is not animating. |
| **Condition: Inv Full** | When the inventory has 28 items. |
| **Condition: Inv Empty** | When the inventory has 0 items. |
| **Condition: Inv Count** | When the inventory has at least X of an item. |
| **Condition: Menu Open** | When the skill/make menu is visible. |
| **Condition: NPC Exists** | When a specific NPC is nearby. |
| **Condition: Object Exists** | When a specific object is nearby. |
| **Condition: In Region** | When the player is in a specific region. |
| **Condition: In Combat** | When the player is in combat. |

Each conditional block can have child tasks added inside it, and can also be inverted using the **NOT** option.

---

### Simple Conditionals (Legacy)

These older task types gate the tasks that follow them. They still work but the **Conditional Block** system above is recommended instead.

| Task | What it checks |
|------|----------------|
| **If Has Item** | Inventory contains the named item. |
| **If Bank Open** | Bank interface is open. |
| **If Animating** | Player is currently animating. |

---

## 5. Available Conditions

Conditions are rules attached to **any** task. A task with conditions only runs when **all** of its conditions are true.

### Condition Types

| Condition | What it checks | Fields needed |
|-----------|----------------|---------------|
| **Has Item** | Inventory contains the item. | Item Name |
| **No Item** | Inventory does not contain the item. | Item Name |
| **Bank Open** | Bank interface is open. | None |
| **Bank Closed** | Bank interface is closed. | None |
| **Animating** | Player is currently animating. | Grace period (optional), Grace variance % (optional) |
| **Idle** | Player is not animating. | Grace period (optional), Grace variance % (optional) |
| **Inv Full** | Inventory has 28 items. | None |
| **Inv Empty** | Inventory has 0 items. | None |
| **Inv Count** | Inventory has at least X of an item. | Item Name, Min Count |
| **Menu Open** | Skill/make menu is visible. | None |
| **NPC Exists** | A specific NPC exists nearby. | NPC Name or ID, Max Distance (optional, 0 = any) |
| **Object Exists** | A specific object exists nearby. | Object Name or ID, Max Distance (optional, 0 = any) |
| **In Region** | Player is in a specific game region. | Region ID |
| **In Combat** | Player is in combat. | None |

### The NOT Option

Each condition can be **inverted** using the NOT checkbox:
- **Has Item** = true when you have the item.
- **NOT Has Item** = true when you do NOT have the item.

This works for every condition type.

### Multiple Conditions

When a task has multiple conditions, **all must be true** for the task to run (AND logic):

```
Task: Withdraw Herbs
Conditions:
  - Bank Open
  - Has Item: "Vial of water"
  - NOT Has Item: "Guam leaf"
```

This task runs only when:
1. Bank is open, AND
2. You have vials, AND
3. You don't have herbs.

### Grace Period on Animation Conditions

For **Animating** and **Idle** conditions, you can set:
- **Grace period** — How long to wait for an animation to start before concluding the player is idle. Default: 2 ticks.
- **Grace variance (%)** — Optional randomization. For example, 20% means the grace period varies by ±20% each time, making timing less predictable. At 0%, grace is exact.

---

## 6. Delays, Ticks, and Timing

### What Is a Tick?

A **tick** is one game cycle, roughly **600 milliseconds** (0.6 seconds). Many timing fields in UG Task Builder can be set in ticks or milliseconds.

### Post-Action Delay

Every task has a **Post-action delay** setting. This is how long the plugin waits after a task finishes before moving to the next one. It helps avoid acting before the game has updated.

| Setting | Meaning |
|---------|---------|
| **Post-action delay** | Base wait time after the task completes. |
| **Ticks** checkbox | If checked, the delay is in game ticks (e.g., 2 = 1200 ms). If unchecked, the value is in milliseconds. |

### Timing Variance (Randomization %)

The **Timing variance (%)** setting adds random variation to delays and timeouts:
- `0%` = exact timing, no randomization.
- `20%` with a base of 2 ticks = actual delay varies between roughly 1.6 and 2.4 ticks.
- Applied per execution, so each time the task runs the delay is slightly different.

Use this to make timing less predictable and more natural.

### Grace Period

The **Anim detect grace** (grace period) controls how long the plugin waits for an animation to start when checking if the player is animating.

After you click an action, there is a brief delay before the animation appears. Without a grace period, the plugin might incorrectly decide you are idle and skip ahead.

- **Default:** 2 ticks.
- **Increase** if the plugin skips actions or thinks you are idle too soon.
- Available on Wait Animation tasks, Wait Idle tasks, Wait Animation Start, Wait Animation Cycle, and on Animating/Idle conditions.

**Grace variance (%)** optionally randomizes the grace period by ±X% each check.

### Recommended Grace Values

| Activity | Suggested Grace Period |
|----------|----------------------|
| Gem Cutting | 2–3 ticks |
| Potion Making | 2 ticks |
| Glass Blowing | 2–3 ticks |
| Fletching | 2 ticks |
| Smithing | 3 ticks |
| Slow or laggy actions | 3–4 ticks |

### When to Adjust Timing

- **Too fast / spam-clicking:** Increase delays, add Wait Animation before menu clicks.
- **Skipping actions:** Increase grace period on Animating/Idle conditions or Wait tasks.
- **Lag or high latency:** Use longer grace periods and delays.

---

## 7. Wait Tasks Explained

This section explains the five wait task types and when to use each.

### Wait Ticks

**What it does:** Pauses for a fixed number of game ticks.

**When to use:** Simple delays between actions where you know exactly how long to wait.

**Fields:** Wait duration (ticks). Supports timing variance (%).

---

### Wait Animation

**What it does:** Waits while the player is animating. Has a configurable maximum timeout to prevent waiting forever.

**When to use:** After starting an activity (like using items), to wait until the activity finishes. The timeout ensures the script continues even if the animation never stops.

**Fields:** Anim timeout (ticks) — maximum time to wait. Grace period. Supports timing variance (%).

**Example:** You use a chisel on a gem. Wait Animation waits while the cutting animation plays, up to the timeout.

---

### Wait Idle

**What it does:** Waits until the player stops animating and is idle.

**When to use:** Before starting a new action that requires the player to be idle. Useful to make sure one activity finishes before the next one begins.

**Fields:** Grace period.

---

### Wait Animation Start

**What it does:** Waits for an animation to begin within the grace period. Use this right after clicking an action to confirm the action actually started.

**When to use:** After clicking an item or object, to verify the animation has begun before continuing.

**Fields:** Grace period.

**Example:** You click "Use Item on Item." Wait Animation Start confirms the crafting animation has started within the grace window.

---

### Wait Animation Cycle

**What it does:** Waits for a complete animation cycle — first waits for the animation to start, then waits for it to stop.

**When to use:** When you need the full action to complete before continuing. Ideal for activities where you want to wait for one full crafting cycle.

**Fields:** Grace period.

**Example:** You use items together. Wait Animation Cycle waits for the animation to begin, then waits until it finishes, before moving to the next task.

---

### Quick Comparison

| Task | Waits for... | Has timeout? | Best used for... |
|------|-------------|-------------|-----------------|
| **Wait Ticks** | Fixed number of ticks | N/A | Simple pauses |
| **Wait Animation** | Animation to finish | Yes (max ticks) | Waiting out a known activity |
| **Wait Idle** | Player to be idle | No | Confirming player is ready |
| **Wait Animation Start** | Animation to begin | Grace period | Confirming an action started |
| **Wait Animation Cycle** | Full start-to-stop cycle | Grace + stop timeout | Waiting for one complete action |

---

## 8. Building a Script

### Step-by-Step

1. **Create a script** — Click **+**, enter a name, confirm.
2. **Open the editor** — Click **Edit** on the script.
3. **Add tasks** — Click **Add Task** (+), choose a type, fill in the fields, and configure conditions.
4. **Set delays** — Use Post-action delay and Timing variance where needed.
5. **Order tasks** — Use Move Up / Move Down so tasks run in the correct sequence.
6. **Save** — Click Save in the task editor dialog.
7. **Go back** — Return to the script list.
8. **Enable the script** — Toggle it ON.
9. **Enable Loop** — If you want the script to repeat.
10. **Start** — Click **START**.

### Example: Making Attack Potions

**Goal:** Combine Guam potion (unf) + Eye of newt at a bank.

**Task 1: Open Bank**
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  - Bank Closed
  - NOT Has Item: "Guam potion (unf)"
```

**Task 2: Deposit All**
```
Type: Deposit All
Delay: 1 tick
Conditions:
  - Bank Open
  - NOT Has Item: "Guam potion (unf)"
```

**Task 3: Withdraw Unf Potions**
```
Type: Withdraw Item
Item Name: "Guam potion (unf)"
Quantity: 14
Delay: 2 ticks
Conditions:
  - Bank Open
  - NOT Has Item: "Guam potion (unf)"
```

**Task 4: Withdraw Secondary**
```
Type: Withdraw Item
Item Name: "Eye of newt"
Quantity: 14
Delay: 2 ticks
Conditions:
  - Bank Open
  - Has Item: "Guam potion (unf)"
  - NOT Has Item: "Eye of newt"
```

**Task 5: Close Bank**
```
Type: Close Bank
Delay: 1 tick
Conditions:
  - Bank Open
  - Has Item: "Guam potion (unf)"
  - Has Item: "Eye of newt"
```

**Task 6: Use Items**
```
Type: Use Item on Item
Item 1: "Guam potion (unf)"
Item 2: "Eye of newt"
Delay: 1 tick
Conditions:
  - Bank Closed
  - Has Item: "Guam potion (unf)"
  - Has Item: "Eye of newt"
  - NOT Menu Open
  - Idle (Grace: 2t)
```

**Task 7: Wait for Animation**
```
Type: Wait Animation
Anim Timeout: 25 ticks
Grace Period: 2 ticks
```

**Task 8: Select Menu**
```
Type: Select Menu Option
Option: 1
Delay: 1 tick
Conditions:
  - Menu Open
```

Enable the script, turn on **Loop**, and click **START**.

---

## 9. Example Script Patterns

### Banking Pattern

```
1. Open Bank      [Bank Closed, NOT Has Item: "material"]
2. Deposit All    [Bank Open]
3. Withdraw A     [Bank Open, NOT Has Item: "A"]
4. Withdraw B     [Bank Open, Has Item: "A", NOT Has Item: "B"]
5. Close Bank     [Bank Open, Has Item: "A", Has Item: "B"]
```

### Making/Crafting Pattern

```
6. Use Item on Item   [Bank Closed, Has A, Has B, NOT Menu Open, Idle]
7. Wait Animation     [timeout ~25 ticks]
8. Select Menu        [Menu Open]
```

### Full Skilling Loop

Combine both patterns above. With **Loop** enabled, the script repeats: bank → craft → bank → craft...

### Walking Pattern

```
1. Walk To Bank   [NOT Has Item: "product"]
2. Open Bank      [Bank Closed]
3. Deposit All    [Bank Open]
4. Withdraw A     [Bank Open, NOT Has Item: "A"]
5. Close Bank     [Bank Open, Has Item: "A"]
6. Walk To Spot   [Has Item: "A"]
7. Click Object   [Has Item: "A"]
8. Wait Animation Cycle
```

### Spell Loop (e.g., High Alchemy)

```
1. Open Bank           [NOT Has Item: "Rune platebody"]
2. Withdraw Item       [Bank Open, NOT Has Item: "Rune platebody"]
3. Close Bank          [Bank Open, Has Item: "Rune platebody"]
4. Cast Spell on Item  [Has Item: "Rune platebody"]
   Spell: "High Level Alchemy"
   Item: "Rune platebody"
   Delay: 3 ticks
```

### Recommended Delays

| Action | Suggested delay |
|--------|-----------------|
| Open Bank | 2 ticks |
| Deposit / Withdraw | 1–2 ticks |
| Close Bank | 1 tick |
| Use Item | 1 tick |
| Select Menu | 1 tick |
| Cast Spell | 3 ticks |
| Wait Animation timeout | 20–40 ticks (varies by activity) |

---

## 10. Presets

Presets are ready-made script templates for common activities. Add a preset, then edit it to match your needs.

### How to Use Presets

1. Click the **Presets** (folder) button.
2. Choose a preset from the menu.
3. The script is added to your list.
4. Click **Edit** to view and adjust tasks, conditions, and timing.
5. Enable the script and start it.

Presets are templates — you can modify them freely after adding.

### Available Presets

| Category | Activities |
|----------|-----------|
| **Herblore** | Attack Potion, Strength Potion, Prayer Potion, Super Attack, Super Strength, Super Restore, Saradomin Brew |
| **Unf Potions** | Guam, Ranarr, Toadflax, Snapdragon |
| **Crafting** | Green / Blue / Black D'hide Bodies |
| **Gem Cutting** | Sapphires, Emeralds, Rubies, Diamonds |
| **Fletching** | Arrow Shafts, Yew Longbows, Magic Longbows |
| **Bow Stringing** | Yew Longbows, Magic Longbows |
| **Magic** | Plank Make (Oak / Mahogany), High Alchemy |
| **Battlestaves** | Air, Water, Earth, Fire |
| **Glass Blowing** | Unpowered Orbs, Lantern Lens |

---

## 11. Detailed Script Guides

For in-depth guides on each preset category — including full task breakdowns, XP rates, profit analysis, and tips — see the individual guides:

| Guide | Description | Link |
|-------|-------------|------|
| **Herblore** | Making finished potions from unf potions | [scripts/Herblore.md](scripts/Herblore.md) |
| **Unfinished Potions** | Making unf potions from herbs + vials | [scripts/UnfPotions.md](scripts/UnfPotions.md) |
| **Crafting (Leather)** | Crafting dragonhide bodies | [scripts/Crafting.md](scripts/Crafting.md) |
| **Gem Cutting** | Cutting uncut gems | [scripts/GemCutting.md](scripts/GemCutting.md) |
| **Fletching** | Cutting logs into bows | [scripts/Fletching.md](scripts/Fletching.md) |
| **Bow Stringing** | Stringing unstrung bows | [scripts/BowStringing.md](scripts/BowStringing.md) |
| **Magic** | Plank Make and High Alchemy | [scripts/Magic.md](scripts/Magic.md) |
| **Battlestaves** | Attaching orbs to staves | [scripts/Battlestaves.md](scripts/Battlestaves.md) |
| **Glass Blowing** | Blowing molten glass | [scripts/GlassBlowing.md](scripts/GlassBlowing.md) |

Each guide includes:
- Full task breakdown with exact settings and conditions
- Flow diagrams
- XP rates and profit analysis
- Tips and tricks
- Requirements (levels, items, setup)

---

## 12. Importing and Exporting Scripts

### Exporting a Script

1. Click **Export** on the script you want to save.
2. Choose where to save the file.
3. The script is saved as a **JSON file**.

**Use export to:**
- Back up working scripts before making changes.
- Share scripts with other users.
- Move scripts between setups.

### Importing a Script

1. Click **Import** (down arrow).
2. Select a compatible JSON script file.
3. The script is added to your list.

### Important Notes

- **Only import from trusted sources.** Imported scripts run actions automatically — review them in the editor before starting.
- **Imported scripts should use supported task and condition types.** If a script was made with a different version, some settings may need adjustment.
- **If you edit JSON files manually,** invalid values may cause the script not to work correctly. Use the in-game editor when possible.
- **Always keep backups.** Export working scripts before modifying them. If something breaks, you can re-import the backup.

### What an Export Contains

A script export is a JSON file containing:
- The script name and settings
- All tasks with their types, fields, conditions, and timing
- Task order and structure

It does **not** contain account-specific data or plugin settings.

---

## 13. Tips and Best Practices

### 1. Always Use Conditions

Without conditions, tasks run every cycle regardless of game state. This leads to spam-clicking and broken behavior.

```
Bad:  Withdraw Herbs (no conditions)
Good: Withdraw Herbs [Bank Open, NOT Has Item: "Herb"]
```

### 2. Order Matters

Tasks run top-to-bottom. Put banking tasks **before** crafting tasks. Put wait tasks **between** actions and menu clicks.

### 3. Wait for Animations

Always add a **Wait Animation** (or Wait Animation Cycle) between clicking and selecting a menu option. Without it, the menu may not be open yet when the Select Menu task runs.

### 4. Use the Idle Condition

Add an **Idle** condition to "Use Item" tasks. This prevents the plugin from interrupting an ongoing animation to start a new action.

### 5. Test in Small Steps

1. Add 2–3 tasks.
2. Test them.
3. Add more tasks.
4. Repeat.

Don't build a 20-task script and test it all at once.

### 6. Export Working Scripts

Once a script works reliably, **Export** it immediately. This is your backup if you break something while editing.

### 7. Use Grace Period for Animation Detection

If your script skips actions because it thinks you're idle right after clicking:
- Increase the **grace period** on the Idle condition (try 3 ticks instead of 2).
- Increase grace on Wait Animation tasks.
- Use 3–4 ticks for slow animations or laggy connections.

### 8. Use Timing Variance

Set **Timing variance (%)** to 10–20% on delays to make timing less predictable and more natural. At 0%, all timing is exact.

### 9. Use Clear Names

Name scripts and tasks descriptively. "Attack Potions" is better than "Script 1". You'll thank yourself when you have 10 scripts.

---

## 14. Troubleshooting

### Script Does Nothing

- Is the script **enabled** (toggle ON)?
- Is **Loop** enabled?
- Are you **logged in**?
- Do conditions match the current game state? (e.g., a "Bank Open" condition won't pass if the bank isn't open.)
- Do you have tasks in the script?

### Tasks Repeat Too Often / Clicking Too Fast

- Add **Wait Animation** before menu clicks.
- Add **Idle** condition to action tasks.
- Increase **Post-action delay**.
- Enable **Timing variance (%)** for more natural pacing.

### Menu Option Not Selected

- Add **Menu Open** condition to the Select Menu Option task.
- Verify the **Option Number** (1 = first, 2 = second, etc.).
- Add **Wait Animation** before the menu task to ensure the menu has time to open.

### Bank Keeps Opening

- Add **NOT Has Item** (for the item you're gathering) to the Open Bank task.
- Ensure withdraw tasks have conditions so they don't run when you already have the items.

### Actions Happen Too Fast

- Increase **Post-action delay**.
- Enable **Timing variance (%)**.
- Add **Wait Animation** or **Wait Idle** where needed.

### Animation / Wait Timing Seems Wrong

- Increase the **grace period** on Animating/Idle conditions and Wait tasks.
- Try 3–4 ticks instead of the default 2.
- If you have lag, use longer grace periods.
- Check that the **Anim timeout** on Wait Animation is long enough for the activity.

### Imported Script Doesn't Work Correctly

- Open the script in the editor and check all task types and conditions.
- Ensure all required fields are filled in.
- The script may have been created with a different version — adjust settings as needed.
- Re-export from a working script and compare.

---

## 15. FAQ

**Can I run multiple scripts at once?**
Yes. All enabled scripts run in order from top to bottom. Disable scripts you don't want to run.

**What does Loop do?**
When Loop is on, after the last task finishes, the script restarts from the first task. Without Loop, the script runs once and stops.

**What does NOT mean on a condition?**
NOT inverts the condition. "NOT Has Item" is true when you do *not* have the item. "NOT Bank Open" is true when the bank is *closed*.

**Should I use Wait Animation or Wait Animation Cycle?**
- **Wait Animation** — Waits while the player is animating, with a max timeout. Good when the script needs to move on even if the animation doesn't stop.
- **Wait Animation Cycle** — Waits for a complete start-to-stop animation cycle. Good when you need one full action to finish before continuing.

**Can I share scripts with other users?**
Yes. Export a script to a JSON file and share it. The other user imports the file. Both need a compatible version of the plugin.

**What happens if I import an old script?**
Older scripts should still work. New fields that didn't exist when the script was created will use default values. If something doesn't work, open the script in the editor and adjust.

**Can I use item IDs instead of names?**
For Inventory Click tasks, you can enter a numeric item ID. For NPC and Object conditions, you can use numeric IDs. For most other tasks, use item names (which support wildcards like `rune*`).

**What is the grace period for?**
After you click an action, there's a short delay before the animation starts. The grace period is how long the plugin waits for that animation to appear before deciding you're idle. Increase it if the plugin skips actions.

**What does Timing variance do?**
It adds small random variation to delays and timeouts. For example, 20% variance on a 2-tick delay means the actual delay is roughly 1.6–2.4 ticks. This makes timing less predictable. At 0%, timing is exact.

---

## 16. Support

For help and support:
- Ask in the **Discord** support channel.
- Share your script JSON file if you need help debugging.
- Include any error messages you see.
- Describe what you expected to happen vs. what actually happened.
