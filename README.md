# UG Task Builder

A visual, no-code script builder for automating Old School RuneScape activities inside RuneLite. Build scripts by combining **tasks** and **conditions** in a point-and-click interface — no programming required. Originally built for bank-standing and skilling (herblore, crafting, fletching, gem cutting, magic training), it now also covers combat, gathering, navigation, variables, and **reactive PvM prayer** (including a Great Olm prayer preset).

---

## Table of Contents

1. [What's New](#1-whats-new)
2. [Overview](#2-overview)
3. [How Scripts Run (read this)](#3-how-scripts-run-read-this)
4. [Key Concepts](#4-key-concepts)
5. [Interface Overview](#5-interface-overview)
6. [How to Create a Script (walkthrough)](#6-how-to-create-a-script-walkthrough)
7. [Available Task Types](#7-available-task-types)
8. [Conditional Blocks](#8-conditional-blocks)
9. [Inline Conditions](#9-inline-conditions)
10. [Variables & Repetition](#10-variables--repetition)
11. [Reactive PvM Prayer](#11-reactive-pvm-prayer)
12. [Delays, Ticks, Grace, and the Idle Watchdog](#12-delays-ticks-grace-and-the-idle-watchdog)
13. [Wait Tasks Explained](#13-wait-tasks-explained)
14. [Example Script Patterns](#14-example-script-patterns)
15. [Presets](#15-presets)
16. [Detailed Script Guides](#16-detailed-script-guides)
17. [Importing and Exporting Scripts](#17-importing-and-exporting-scripts)
18. [Tips and Best Practices](#18-tips-and-best-practices)
19. [Troubleshooting](#19-troubleshooting)
20. [FAQ](#20-faq)
21. [Support](#21-support)

---

## 1. What's New

UG Task Builder has grown well beyond bank-standing. Highlights since the original release:

### Variables
- New tasks: **Set Var**, **Inc Var**, **Dec Var**, and the conditional block / inline condition **Condition: Var Compare**.
- Variables are simple named integer counters you can set, increment, decrement, and compare. They let you count cycles, gate work, and stop after a set number of iterations.
- Repetition is done by turning the **Loop** toggle ON and using variables + **Condition: Var Compare** + **Stop Script** (see [Section 10](#10-variables--repetition)).

### Reactive PvM Prayer
- New trigger tasks: **Prayer Trigger: Chat**, **Prayer Trigger: Projectile**, **Prayer Trigger: Animation**.
- These flip your overhead prayer automatically in reaction to game events — a chat message, an incoming projectile, or an NPC attack animation. They react to events instantly, independently of your task list, so prayers swap with no perceptible delay.
- **Olm Basics** preset: open **Presets ▾ → Boss → Olm Basics** to drop in three editable chat triggers that swap overheads for the Great Olm's basic spheres (melee / magic / range).

### Combat, Consumables & Special Attack
- **Eat Food**, **Drink Potion**, **Activate / Deactivate Prayer**, **Deactivate All Prayers**, **Toggle Quick Prayer**, **Special Attack**, **Attack NPC**.

### Banking, Gathering & Navigation
- **Deposit All Except**, **Deposit Equipment**, **Withdraw Noted**, **Pickup Ground Item**, **Walk To Object**, **Walk To NPC**, **Teleport**.

### Numeric Conditional Blocks
- **Condition: HP / Prayer / Run Energy / Spec Energy / Skill Level / Bank Contains / Item Equipped / Ground Item / At Location / Var Compare** — all support a comparator (`< <= > >= ==`) and a threshold.

### Per-Task Interaction Mode & Idle Watchdog
- Each task can pick a **click type** (Legit / Bypass / Hybrid).
- An optional **idle-retry watchdog** re-clicks a task if no animation starts within a window, bounded by max retries and a per-task tick timeout.

### New Boss Preset
- **Olm Basics** — a new **Presets ▾ → Boss** category that adds three editable reactive chat prayer triggers for the Great Olm's basic spheres.

---

## 2. Overview

UG Task Builder lets you create **scripts** made of **tasks**. Each task performs a single action — open the bank, withdraw an item, use items together, attack an NPC, drink a potion, swap a prayer, and so on. You attach **conditions** to a task so it only acts when the right game state is met.

When the **Loop** toggle is ON, the script keeps repeating its task list. This makes it easy to automate any repetitive pattern: bank → gather → make → bank, or fight → eat → loot → repeat.

A separate **reactive prayer** layer can run alongside your tasks, swapping overheads in response to game events. It activates when you press START (for any Prayer Trigger tasks in your enabled scripts) and is cleared when you press STOP (see [Section 11](#11-reactive-pvm-prayer)).

---

## 3. How Scripts Run (read this)

Understanding this makes everything else click. UG Task Builder does **not** run tasks like lines of a program with a cursor that marches straight down the list. Instead it works through your list by **checking conditions**:

1. **It looks at your tasks from the top down.**
2. For each task, it checks that task's conditions. **If all the conditions are true, the task runs.** If they aren't, the task is skipped for now and **checked again a moment later** — it effectively *waits* until its conditions become true.
3. **One real game action happens at a time.** A task that clicks, withdraws, or walks does its thing, and then the plugin re-reads the game before deciding what to do next. Lightweight tasks (variables, prayer triggers) don't make it wait.
4. A task that has already done its action won't repeat — until the list comes around again.
5. **With Loop ON, the whole list becomes eligible again once it finishes.** This is what makes a script repeat: every cycle, all tasks can fire again, starting from the top.

Things worth remembering:
- **Order is a priority hint, not a strict sequence.** The first task whose conditions are true is the one that runs. You use conditions to enforce the real order (e.g. "withdraw herb" only runs once the bank is open and you don't already have the herb).
- **A task with no conditions runs constantly.** That is usually a mistake — gate every task with conditions.
- **Repetition and counting are done with [variables](#10-variables--repetition)** plus the Loop toggle, not by jumping around the list.

> Note: Reactive prayer triggers ([Section 11](#11-reactive-pvm-prayer)) work differently. They react to game events instantly and don't wait their turn in the list, so an overhead swap is never delayed by your other tasks.

---

## 4. Key Concepts

| Term | Meaning |
|------|---------|
| **Script** | A named collection of tasks. You can have multiple scripts; enabled ones run together. |
| **Task** | A single action or check the plugin performs. |
| **Condition** | An inline rule attached to a task. The task only runs when **all** its conditions are true. |
| **Conditional Block** | A task that contains child tasks. The children only run when the block's condition is true. |
| **Loop** | When ON, the task list repeats once it finishes. |
| **Variable** | A named integer counter shared across the script. Set, increment, decrement, and compare it. |
| **Reactive Prayer Trigger** | A rule that swaps your overhead prayer in reaction to a game event (chat / projectile / animation). |
| **Delay** | Post-action wait after a task completes, before the next action. Ticks or milliseconds. |
| **Grace Period** | Extra time given for an animation to start before deciding the player is idle. |
| **Watchdog** | Optional idle-retry safety net that re-clicks a task if no animation starts in time. |
| **Click Type** | Per-task interaction mode: Legit, Bypass, or Hybrid. |
| **Preset** | A ready-made script template you can add and then customize. |

---

## 5. Interface Overview

### Main Panel

| Control | What it does |
|---------|--------------|
| **START / STOP** | One button that starts or stops all enabled scripts. It shows a green **START** when stopped and a red **STOP** while running. Starting clears all variables. |
| **+** | Create a new empty script. |
| **Presets ▾** | Choose from ready-made script templates (now includes a **Boss** category). |
| **Import** | Load a script from a JSON file. |
| **Loop** toggle | On its own row. When ON, the task list repeats once it finishes; when OFF, the list runs once then idles until you press STOP. |

### Per-Script Controls

| Control | What it does |
|---------|--------------|
| **Toggle** | Enable or disable the script. Disabled scripts are skipped. |
| **Edit** | Open the task editor. |
| **Export** | Save the script as a JSON file. |
| **Delete** | Remove the script (asks for confirmation). |

### Task Editor

| Control | What it does |
|---------|--------------|
| **Add Task (+)** | Add a task and configure its type, fields, and conditions. |
| **Clear All (X)** | Remove all tasks (asks for confirmation). |
| **Move Up / Move Down** | Reorder tasks (higher = checked first). |
| **Edit (pencil)** | Open the task dialog to change a task's settings. |
| **Delete (X)** | Remove a single task. |
| **Back** | Return to the script list. |

### In-Game Overlay

While running, the overlay shows elapsed **Time**, current **Status**, the active **Task**, **Progress** (task X of total), and **Cycles** completed.

---

## 6. How to Create a Script (walkthrough)

This walkthrough builds a small, correct script from scratch and explains *why* each choice matters.

### Step 1 — Create the script
Click **+**, type a descriptive name (e.g. "Cut Sapphires"), confirm. Click **Edit** to open the task editor.

### Step 2 — Plan the flow as a set of guarded states
Because the plugin runs the first task whose conditions pass, think in terms of *"do X **when** Y."* Each task is a state guarded by conditions. For a cut-gems loop:

```
WHEN bank closed AND inventory has uncut gems AND idle   → use chisel on gem
WHEN make menu is open                                   → select option 1
WHEN bank closed AND no uncut gems                       → open bank
WHEN bank open AND has cut gems                          → deposit all
WHEN bank open AND no uncut gems                         → withdraw uncut gems
```

### Step 3 — Add the tasks
Click **Add Task**, pick the **Type**, fill the fields, then add **Conditions** at the bottom of the dialog. Each condition has a **NOT** checkbox to invert it. Example for the "use chisel" task:

```
Type: Use Item on Item
Item 1: "Chisel"
Item 2: "Uncut sapphire"
Post-action delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Uncut sapphire"
  └─ Idle (grace 2 ticks)
```

### Step 4 — Set delays, grace, and (optionally) the watchdog
- **Post-action delay** paces the script after each action. Tick the **Ticks** box to use game ticks.
- **Timing variance (%)** randomizes the delay so it isn't perfectly uniform.
- For animation-driven work, set the **grace period** (default 2 ticks). See [Section 12](#12-delays-ticks-grace-and-the-idle-watchdog).
- Optionally enable the **idle watchdog** so the task re-clicks if nothing happens.

### Step 5 — Order, save, enable
Use **Move Up / Move Down** to put higher-priority guards first (it's a tie-breaker when multiple tasks are eligible). **Save** each task, then **Back**. Toggle the script **ON**.

### Step 6 — Loop and run
For a repeating activity, turn the **Loop** toggle **ON** (the task list repeats once it finishes). Click **START**.

### Step 7 — Verify and iterate
Watch the overlay's **Status / Task / Cycles**. Build incrementally — add 2–3 tasks, test, then add more. **Export** the script once it works as a backup.

> Tip: If a task fires every cycle and spam-clicks, it's missing a condition. If a task never fires, its conditions can never all be true at once — re-check them.

---

## 7. Available Task Types

### Bank Actions

| Task | What it does | Fields |
|------|--------------|--------|
| **Open Bank** | Opens the nearest bank booth/chest. | None |
| **Close Bank** | Closes the bank interface. | None |
| **Deposit All** | Deposits all items, or all of one named item. | Item Name (optional) |
| **Deposit Item** | Deposits a set amount of an item. | Item Name, Quantity |
| **Withdraw Item** | Withdraws a set amount of an item. | Item Name, Quantity |
| **Deposit All Except** | Deposits everything except a keep-list. | Keep list (comma-separated names) |
| **Deposit Equipment** | Deposits all worn equipment. | None |
| **Withdraw Noted** | Withdraws an item as noted. | Item Name/ID, Quantity |

**Quantity for bank tasks:** a positive number = that exact amount; `-1` = all of that item.

### Item Actions

| Task | What it does | Fields |
|------|--------------|--------|
| **Use Item on Item** | Uses one item on another. | Item 1, Item 2 |
| **Use Item on Object** | Uses an item on a game object. | Item Name, Object Name, Action (optional) |
| **Drop Item** | Drops an item. | Item Name |
| **Equip Item** | Equips an item (tries Wear/Wield/Equip). | Item Name |
| **Inventory Click** | Clicks an inventory item with an action. | Item Name, Action (optional) |

**Item name matching:** case-insensitive, ignores spaces, supports **wildcards** (`rune*`, `*potion*`). Numeric **IDs** work for Inventory Click and for NPC/Object conditions.

### Interaction Actions

| Task | What it does | Fields |
|------|--------------|--------|
| **Click Object** | Interacts with a game object. | Object Name, Action (optional) |
| **Click NPC** | Interacts with an NPC. | NPC Name, Action (optional) |
| **Click Widget** | Clicks an interface element. | Widget Info (`group:child`) |
| **Attack NPC** | Attacks an NPC; skips if already attacking a match. | NPC Name/ID |

Object/NPC names support **wildcards** and are case-insensitive. If no action is given, the first available action is used.

### Combat, Consumables & Prayer

| Task | What it does | Fields |
|------|--------------|--------|
| **Eat Food** | Eats the first present food from a list. | Food names (comma-separated) |
| **Drink Potion** | Drinks the first present potion (lax dose match) from a list. | Potion names (comma-separated) |
| **Activate Prayer** | Activates a prayer by name. | Prayer Name |
| **Deactivate Prayer** | Deactivates a prayer by name. | Prayer Name |
| **Deactivate All Prayers** | Turns off all active prayers. | None |
| **Toggle Quick Prayer** | Enables/disables quick prayers. | None |
| **Special Attack** | Uses the spec orb if spec ≥ minimum %. | Min Spec % (default 50) |

### Spell & Teleport Actions

| Task | What it does | Fields |
|------|--------------|--------|
| **Cast Spell** | Casts a spell with no target. | Spell Name |
| **Cast Spell on Item** | Casts a spell on an inventory item. | Spell Name, Item Name |
| **Teleport** | Teleports via spell or item. | Spell/Item name |

Common spells: High Level Alchemy, Plank Make, Superheat Item, Enchant Crossbow Bolt.

### Gathering & Movement

| Task | What it does | Fields |
|------|--------------|--------|
| **Pickup Ground Item** | Takes a ground item by name/ID. | Item names (comma-separated) |
| **Walk To** | Walks to X,Y,Z and waits until arrived (within 2 tiles). | X, Y, Z (plane) |
| **Walk To Object** | Walks to an object, then interacts. | Object Name, Action (optional) |
| **Walk To NPC** | Walks to an NPC, then interacts. | NPC Name, Action (optional) |

Z is the game floor (0 = ground, 1/2/3 = upper floors).

### Wait Actions

| Task | What it does | Fields |
|------|--------------|--------|
| **Wait Ticks** | Pauses a fixed number of ticks. | Wait duration (ticks) |
| **Wait Animation** | Waits while animating, up to a timeout. | Anim timeout (ticks), Grace |
| **Wait Idle** | Waits until the player stops animating. | Grace |
| **Wait Animation Start** | Waits for an animation to begin. | Grace |
| **Wait Animation Cycle** | Waits for a full start-then-stop cycle. | Grace |

See [Section 13](#13-wait-tasks-explained).

### Menu, Variable & Control

| Task | What it does | Fields |
|------|--------------|--------|
| **Select Menu Option** | Clicks an option in the make/skill menu. | Menu Option ("Make", or an option number) |
| **Custom Action** | Runs a custom command string (advanced). | Custom action string |
| **Set Var** | Sets a variable to a value. | Var Name, Value |
| **Inc Var** | Increases a variable (default step 1). | Var Name, Amount |
| **Dec Var** | Decreases a variable (default step 1). | Var Name, Amount |
| **Stop Script** | Stops execution. | None |

See [Section 10](#10-variables--repetition) for variables.

### Reactive Prayer Triggers

| Task | What it does | Fields |
|------|--------------|--------|
| **Prayer Trigger: Chat** | Holds an overhead when a chat phrase appears. | Trigger Phrase, Prayer, Priority |
| **Prayer Trigger: Projectile** | Pre-prays an overhead from an incoming projectile id. | Projectile ID, Prayer, Lead (ticks), Priority |
| **Prayer Trigger: Animation** | Prays an overhead after an NPC attack animation. | Anim ID, Prayer, Hit-delay (ticks), Priority |

See [Section 11](#11-reactive-pvm-prayer).

### Simple Conditionals (Legacy)

The older **If Has Item**, **If Bank Open**, and **If Animating** tasks still work but the **Conditional Block** system is recommended instead.

---

## 8. Conditional Blocks

Conditional blocks are tasks that contain **child tasks**. The children only execute when the block's condition is true. Each block can be inverted with **NOT**.

| Conditional Block | Children run when... | Fields |
|-------------------|----------------------|--------|
| **Has Item** | Inventory contains the item. | Item Name |
| **No Item** | Inventory does NOT contain the item. | Item Name |
| **Bank Open / Bank Closed** | Bank is open / closed. | None |
| **Animating / Idle** | Player is / isn't animating. | Grace |
| **Inv Full / Inv Empty** | Inventory has 28 / 0 items. | None |
| **Inv Count** | Inventory has at least X of an item. | Item Name, Min Count |
| **Menu Open** | The make/skill menu is visible. | None |
| **NPC Exists** | A named/ID NPC is nearby. | Name/ID, Max Distance |
| **Object Exists** | A named/ID object is nearby. | Name/ID, Max Distance |
| **Ground Item** | A ground item exists nearby. | Name/ID, Max Distance |
| **In Region** | Player is in a region. | Region ID |
| **In Combat** | Player is in combat. | None |
| **HP** | HP compares to a threshold. | Op, Threshold, %/absolute |
| **Prayer** | Prayer points compare to a threshold. | Op, Threshold, %/absolute |
| **Run Energy** | Run energy compares to a threshold. | Op, Threshold |
| **Spec Energy** | Special-attack % compares to a threshold. | Op, Threshold |
| **Skill Level** | A skill (boosted/real) compares to a threshold. | Skill, Boosted/Real, Op, Threshold |
| **Bank Contains** | Bank has X+ of an item. | Item Name, Op, Threshold |
| **Item Equipped** | An item is worn. | Item Name |
| **At Location** | Player is within range of a tile. | X, Y, Z, Radius |
| **Var Compare** | A variable compares to a value. | Var Name, Op, Threshold |

---

## 9. Inline Conditions

Inline conditions are rules attached to **any** task (not just blocks). A task only runs when **all** of its conditions are true (AND logic). Most block types above have an inline equivalent, plus item/bank/animation checks.

### The NOT option
Every condition has a **NOT** checkbox that inverts it: "NOT Has Item" is true when you do *not* have the item.

### Comparators
Numeric conditions (HP, Prayer, Run Energy, Spec, Skill Level, Bank Contains, Var Compare) use an **Op** symbol:

| Op | Meaning |
|----|---------|
| `<`  | less than |
| `<=` | less than or equal |
| `>`  | greater than |
| `>=` | greater than or equal |
| `==` | equal |

If the op is left blank (e.g. on a legacy script), it defaults to `>=`.

### Multiple conditions example
```
Task: Withdraw Herbs
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Vial of water"
  └─ NOT Has Item: "Guam leaf"
```
Runs only when the bank is open AND you have vials AND you don't already have herbs.

### Grace on animation conditions
**Animating** and **Idle** conditions take a **grace period** (default 2 ticks) and optional **grace variance (%)** — see [Section 12](#12-delays-ticks-grace-and-the-idle-watchdog).

---

## 10. Variables & Repetition

Variables are named integer counters shared across your script. They are the way to count and to repeat a fixed number of times. Repetition is done by turning the **Loop** toggle ON and using variables + **Condition: Var Compare** + **Stop Script** to end the run.

### How variables behave
- Variables are **integers**. A variable that was never set reads as **0**.
- All variables are **cleared when you press START** (every counter starts at 0 for a fresh run).
- They **persist across loop cycles** within a single run.

### One-per-cycle behaviour
Like every task, **Set/Inc/Dec Var run at most once per cycle**. With **Loop ON**, they become eligible again next cycle. So an `Inc Var count by 1` placed in a Loop-ON script **counts one per cycle** — effectively counting iterations. They fire instantly and never stall the script.

### The counting pattern
Put `Inc Var` after the work in a Loop-ON script; it increments once per cycle. Then gate the work or the stop on the count with **Var Compare**.

### Recipe: do N times, then stop
```
(Loop ON; "n" auto-zeroes at START)
1. ...work tasks...
2. Inc Var:  n  by 1
3. Condition: Var Compare  (n >= N)
     └─ Stop Script
```
When `n` reaches `N`, the Var Compare block's child **Stop Script** stops the run.

> Do **not** put `Set Var n = 0` inside the loop body — with Loop ON it would re-zero every cycle and the counter would never climb. Rely on the START-time clear instead, or gate the Set behind a condition so it only runs once.

### Worked examples
- **Count crafts per cycle:** `Inc Var crafted by 1` after the make action, in a Loop-ON script.
- **Stop after 100 cycles:** Var Compare `cycles >= 100` → Stop Script (with `Inc Var cycles by 1` each cycle).
- **Track a one-time flag:** `Set Var setup = 1` gated behind `Var Compare setup == 0`, so setup tasks only run on the first eligible cycle.

---

## 11. Reactive PvM Prayer

Reactive prayer triggers swap your **overhead protection prayer** automatically in response to game events. They react to events instantly and **don't wait their turn in your task list** — so an overhead flips with no delay.

### Trigger types

| Trigger | Reacts to | Key fields |
|---------|-----------|-----------|
| **Prayer Trigger: Chat** | A chat message phrase | Trigger Phrase, Prayer, Priority |
| **Prayer Trigger: Projectile** | An incoming projectile by id | Projectile ID, Prayer, Lead (ticks), Priority |
| **Prayer Trigger: Animation** | An NPC attack animation by id | Anim ID, Prayer, Hit-delay (ticks), Priority |

- **Prayer** is a prayer name such as `PROTECT_FROM_MELEE`, `PROTECT_FROM_MAGIC`, `PROTECT_FROM_MISSILES`.
- **Priority** ranks competing triggers — **lower wins**.
- **Chat** triggers **HOLD** the prayer until the next chat trigger overrides it (good for attacks that persist).
- **Projectile** triggers **pre-pray**: the **lead** is in ticks (default 1 = active on the impact tick), so the prayer is up exactly in time.
- **Animation** triggers fire **hit-delay** ticks after the matching animation.
- Chat matching is **case-insensitive substring**, so suffixes (e.g. "... your prayers have been sapped.") are ignored.

### Olm Basics preset (Presets → Boss → Olm Basics)
The Great Olm prayer setup is a **preset group** with **three editable chat triggers**, one per basic-attack sphere:

| Sphere phrase (substring) | Prayer |
|---------------------------|--------|
| `sphere of aggression` | PROTECT_FROM_MELEE |
| `sphere of magical power` | PROTECT_FROM_MAGIC |
| `sphere of accuracy and dexterity` | PROTECT_FROM_MISSILES |

Open the group to view/edit the three triggers. They HOLD until the next sphere is announced.

### Combining with tasks
Reactive triggers pair well with normal tasks and numeric conditions: eat below an HP threshold (Condition: HP), spec when ready (Special Attack + Condition: Spec), drink prayer potions (Condition: Prayer), and gate behaviour on Condition: In Combat — all while the trigger layer keeps overheads correct. Use the trigger tasks for prayer swaps rather than trying to script them by hand.

---

## 12. Delays, Ticks, Grace, and the Idle Watchdog

### Ticks
A **tick** is ~600 ms. Many timing fields accept ticks or milliseconds.

### Post-action delay
Every task has a **Post-action delay** — how long to wait after the task acts before the next action.

| Setting | Meaning |
|---------|---------|
| **Post-action delay** | Base wait after the task completes. |
| **Ticks** checkbox | On = value is in ticks (2 = 1200 ms). Off = milliseconds. |
| **Timing variance (%)** | Random ± variance per execution. 0% = exact. |

### Grace period
After you click an action, there's a short delay before the animation appears. The **grace period** (default 2 ticks) is how long the plugin waits for that animation before deciding you're idle. Increase it if the script skips actions or thinks you're idle too soon. Available on Wait Animation tasks, Wait Idle, Wait Animation Start, Wait Animation Cycle, and Animating/Idle conditions. **Grace variance (%)** randomizes it.

| Activity | Suggested Grace |
|----------|-----------------|
| Gem Cutting | 2–3 ticks |
| Potion Making | 2 ticks |
| Glass Blowing | 2–3 ticks |
| Fletching | 2 ticks |
| Smithing | 3 ticks |
| Slow / laggy actions | 3–4 ticks |

### Idle-retry watchdog (optional)
For action tasks you can enable a watchdog that re-clicks if nothing happens:

| Field | Meaning |
|-------|---------|
| **Expect animation** | Turns the watchdog on for this task. |
| **Retry if idle (ms)** | If no animation starts within this window, re-click. 0 = disabled. |
| **Max retries** | Re-click attempts before giving up. 0 = use the default (3). |
| **Timeout (ticks)** | Hard per-task ceiling. 0 = use the default (50). |
| **Expected animation id** | Require this exact anim id as "progress." -1 = any animation counts. |

Use it for laggy interactions where a click can be dropped.

### Click type (interaction mode)
Each task can set a **click type**: **Legit**, **Bypass**, or **Hybrid**. Leave it at Legit unless you have a specific reason to change it; Legit preserves default behaviour.

---

## 13. Wait Tasks Explained

### Wait Ticks
Pauses a fixed number of ticks. Use for simple, known delays. Supports timing variance.

### Wait Animation
Waits while animating, with a max timeout (so the script continues even if the animation never stops). Use after starting an activity. Fields: Anim timeout (ticks), Grace.

### Wait Idle
Waits until the player is idle. Use before an action that requires the player to be free. Field: Grace.

### Wait Animation Start
Waits for an animation to begin within the grace window. Use right after clicking to confirm the action started. Field: Grace.

### Wait Animation Cycle
Waits for a full start-then-stop cycle. Use when you need one whole action to finish. Field: Grace.

### Quick Comparison

| Task | Waits for... | Has timeout? | Best for... |
|------|-------------|-------------|-------------|
| **Wait Ticks** | Fixed ticks | N/A | Simple pauses |
| **Wait Animation** | Animation to finish | Yes (max ticks) | Waiting out a known activity |
| **Wait Idle** | Player to be idle | No | Confirming player is ready |
| **Wait Animation Start** | Animation to begin | Grace | Confirming an action started |
| **Wait Animation Cycle** | Full start-to-stop | Grace + stop | One complete action |

---

## 14. Example Script Patterns

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

### Full Skilling Loop with a counter + stop
```
(Loop ON; "made" auto-zeroes at START)
1. Open Bank          [Bank Closed, NOT Has Item: "uncut gem"]
2. Deposit All        [Bank Open, Has Item: "cut gem"]
3. Withdraw Uncut     [Bank Open, NOT Has Item: "uncut gem"]
4. Close Bank         [Bank Open, Has Item: "uncut gem"]
5. Use Chisel on Gem  [Bank Closed, Has Item: "uncut gem", Idle]
6. Select Menu        [Menu Open]
7. Inc Var: made by 1
8. Condition: Var Compare (made >= 500)
      └─ Stop Script
```

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

### Spell Loop (e.g. High Alchemy)
```
1. Open Bank           [NOT Has Item: "Rune platebody"]
2. Withdraw Item       [Bank Open, NOT Has Item: "Rune platebody"]
3. Close Bank          [Bank Open, Has Item: "Rune platebody"]
4. Cast Spell on Item  [Has Item: "Rune platebody"]
   Spell: "High Level Alchemy", Item: "Rune platebody", Delay: 3 ticks
```

### Combat Loop (with reactive prayer + consumables)
```
(Loop ON, plus reactive Prayer Trigger tasks for the boss)
1. Eat Food           [Condition: HP < 40]
2. Drink Potion       [Condition: Prayer < 20]  (prayer potion)
3. Special Attack     [Condition: Spec >= 50, Condition: In Combat]
4. Attack NPC         [NOT In Combat]
5. Pickup Ground Item [drops list]
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
| Wait Animation timeout | 20–40 ticks |

---

## 15. Presets

Presets are ready-made templates. Add one, then edit it freely.

### How to use presets
1. Click **Presets ▾**.
2. Pick a category, then a preset (Boss presets are inserted directly).
3. Click **Edit** to adjust tasks, conditions, and timing.
4. Enable the script and start it.

### Available presets

| Category | Activities |
|----------|-----------|
| **Herblore** | Attack, Strength, Prayer, Super Attack, Super Strength, Super Restore, Saradomin Brew |
| **Unf Potions** | Guam, Ranarr, Toadflax, Snapdragon |
| **Crafting** | Green / Blue / Black D'hide Bodies |
| **Gem Cutting** | Sapphires, Emeralds, Rubies, Diamonds |
| **Fletching** | Arrow Shafts, Yew Longbows, Magic Longbows |
| **Bow Stringing** | Yew Longbows, Magic Longbows |
| **Magic** | Plank Make (Oak / Mahogany), High Alch |
| **Battlestaves** | Air, Water, Earth, Fire |
| **Glass Blowing** | Unpowered Orbs, Lantern Lens |
| **Boss** | **Olm Basics** (3 reactive chat prayer triggers) |

---

## 16. Detailed Script Guides

For in-depth guides — full task breakdowns, XP rates, profit, and tips — see the per-category docs:

| Guide | Description | Link |
|-------|-------------|------|
| **Herblore** | Finished potions from unf potions | [scripts/Herblore.md](scripts/Herblore.md) |
| **Unfinished Potions** | Unf potions from herbs + vials | [scripts/UnfPotions.md](scripts/UnfPotions.md) |
| **Crafting (Leather)** | Dragonhide bodies | [scripts/Crafting.md](scripts/Crafting.md) |
| **Gem Cutting** | Cutting uncut gems | [scripts/GemCutting.md](scripts/GemCutting.md) |
| **Fletching** | Cutting logs into bows | [scripts/Fletching.md](scripts/Fletching.md) |
| **Bow Stringing** | Stringing unstrung bows | [scripts/BowStringing.md](scripts/BowStringing.md) |
| **Magic** | Plank Make and High Alchemy | [scripts/Magic.md](scripts/Magic.md) |
| **Battlestaves** | Attaching orbs to staves | [scripts/Battlestaves.md](scripts/Battlestaves.md) |
| **Glass Blowing** | Blowing molten glass | [scripts/GlassBlowing.md](scripts/GlassBlowing.md) |

---

## 17. Importing and Exporting Scripts

### Exporting
1. Click **Export** on a script.
2. Choose a location. The script is saved as a **JSON file**.

Use export to back up, share, or move scripts.

### Importing
1. Click **Import**.
2. Select a compatible JSON file. The script is added to your list.

### Important notes
- **Only import from trusted sources** — scripts run actions automatically. Review them in the editor first.
- **Older scripts are safe to load.** If a script file contains a task type that no longer exists, that task is quietly skipped and the rest of the script still imports fine.
- **Manually edited JSON** with invalid values may not run correctly — prefer the in-game editor.
- **Keep backups.** Export a working script before editing it.

### What an export contains
The script name and settings, all tasks (types, fields, conditions, timing), and task order/structure. It does **not** contain account data or plugin settings.

---

## 18. Tips and Best Practices

1. **Always use conditions.** A task with no conditions runs constantly and spam-clicks. Gate it.
2. **Conditions encode order.** Use conditions (Bank Open, Has Item, NOT Has Item, Idle) to make tasks fire in the right sequence.
3. **Wait for animations** between clicking and selecting a menu option, or the menu may not be open yet.
4. **Add an Idle condition** to "Use Item" tasks so the script doesn't interrupt an ongoing animation.
5. **Count with variables.** For "do N then stop," use Inc Var + Var Compare → Stop Script.
6. **Use reactive prayer triggers for PvM**, not scripted prayer swaps — they're event-driven and faster.
7. **Test in small steps.** Add a few tasks, test, add more.
8. **Export working scripts** as backups.
9. **Tune grace** if the script skips actions (try 3 ticks; 3–4 for lag).
10. **Use timing variance** (10–20%) for less predictable pacing.
11. **Name things clearly.** "Cut Sapphires" beats "Script 1".

---

## 19. Troubleshooting

### Script does nothing
- Is the script **enabled** and are you **logged in**?
- Does any task have conditions that can currently all be true? If not, nothing fires.
- For repeating activities, is **Loop** ON?

### Tasks repeat too often / clicking too fast
- Add an **Idle** condition and/or **Wait Animation** before menu clicks.
- Increase **Post-action delay**; enable **Timing variance**.

### Menu option not selected
- Add a **Menu Open** condition to the Select Menu Option task.
- Verify the **Option Number** (1 = first).
- Add **Wait Animation** before the menu task.

### Bank keeps opening
- Add **NOT Has Item** (the item you're gathering) to Open Bank.
- Gate withdraw tasks so they don't run when you already have the items.

### A counter never increases / never stops
- Make sure **Loop** is ON (Inc Var only counts again each cycle when looping).
- Don't put **Set Var = 0** inside the loop body (it re-zeros every cycle).
- Check the **Var Compare** op and threshold (blank op defaults to `>=`).

### Reactive prayer doesn't swap
- Confirm the trigger task is in an **enabled** script and the script is **running**.
- For Chat: the **phrase** must be a lowercased substring of the in-game message.
- For Projectile/Animation: verify the **ID** (use the in-client debug tools to capture it).
- Check **priority** if two triggers compete (lower wins).

### Animation / wait timing seems wrong
- Increase **grace** (3–4 ticks for lag) and the Wait Animation **timeout**.
- Consider enabling the **idle watchdog** for dropped clicks.

### Imported script doesn't work
- Open it in the editor; check task types and required fields.
- Any task type that no longer exists is skipped on import — rebuild that part with current task types.
- Re-export from a working script and compare.

---

## 20. FAQ

**Can I run multiple scripts at once?**
Yes. All enabled scripts run together. Disable the ones you don't want.

**What does Loop do?**
When ON, the task list repeats once it finishes. With Loop OFF, each task runs at most once; after the list has run through, the script just idles (the button stays red **STOP**) until you click **STOP**. There is no auto-stop — only the **Stop Script** task (or your click) ends a run.

**Why doesn't the script run my tasks strictly top-to-bottom?**
Because it runs the first task whose conditions are true and re-checks the rest a moment later. Use conditions to enforce order. See [Section 3](#3-how-scripts-run-read-this).

**How do I repeat a script over and over?**
Turn the **Loop** toggle ON. The task list will repeat once it finishes. To make it stop after a set number of times, use **Inc Var + Var Compare → Stop Script** (see below). See [Section 10](#10-variables--repetition).

**How do I do "repeat N times then stop"?**
`Inc Var n by 1` each cycle, plus a **Var Compare** block `n >= N` whose child is **Stop Script**.

**How do I set up Olm prayers?**
Presets → **Boss → Olm Basics**. It adds three editable chat triggers that HOLD the right overhead per sphere.

**What's the difference between a chat, projectile, and animation prayer trigger?**
Chat HOLDs an overhead while a phrase is present (good for persistent attacks). Projectile pre-prays from an incoming projectile id (timed to be up on the impact tick). Animation prays a set number of ticks after an NPC's attack animation.

**Can I use item IDs instead of names?**
Yes for Inventory Click and for NPC/Object/Ground-item conditions. Names elsewhere support wildcards like `rune*`.

**What does NOT mean on a condition?**
It inverts the check: "NOT Has Item" is true when you do *not* have the item.

**What does Timing variance do?**
Adds random ± variation to delays/timeouts. 20% on a 2-tick delay gives roughly 1.6–2.4 ticks. 0% = exact.

**What is the grace period for?**
It's how long the plugin waits for an animation to start after a click before deciding you're idle. Increase it if actions get skipped.

---

## 21. Support

- Ask in the **Discord** support channel.
- Share your script JSON if you need debugging help.
- Include any error messages, and describe expected vs actual behaviour.
