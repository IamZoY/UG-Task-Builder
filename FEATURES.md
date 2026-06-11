# UG Task Builder — Complete Feature Reference

UG Task Builder is a **visual, no-code task-scripting plugin**. You build a script in the side panel
by stacking **tasks** (and **condition blocks**), press **Start**, and the plugin runs them. This is
the full reference for the **current shipped feature set**: every task type, every condition, the
condition blocks, the per-task idle-retry option, the per-task click type, the **Loop** toggle,
variables, **Stop Script**, the three reactive prayer triggers, and the **Presets** menu.

Everything below is named exactly as it appears **on screen** — in the **Task Type** dropdown, in the
task editor field labels, and in the **Presets ▾** menu. If you are authoring a script file by hand,
see [Section 14 — The exported script file format](#14-the-exported-script-file-format) for the exact
keys an exported `.json` uses.

---

## Table of contents

- [1. How the engine runs your script (read this first)](#1-how-the-engine-runs-your-script-read-this-first)
- [2. The task editor — common fields](#2-the-task-editor--common-fields)
- [3. Click type (Legit / Bypass / Hybrid)](#3-click-type-legit--bypass--hybrid)
- [4. The idle-retry option (Expect animation)](#4-the-idle-retry-option-expect-animation)
- [5. Task types — actions](#5-task-types--actions)
- [6. Condition blocks (📋)](#6-condition-blocks-)
- [7. Inline conditions (per-task)](#7-inline-conditions-per-task)
- [8. The comparator](#8-the-comparator)
- [9. Variables & repetition (Set/Inc/Dec Var + Var Compare + Stop Script)](#9-variables--repetition)
- [10. Reactive PvM prayer (the three Prayer Trigger types)](#10-reactive-pvm-prayer)
- [11. The Loop toggle](#11-the-loop-toggle)
- [12. Presets](#12-presets)
- [13. Repeating and branching](#13-repeating-and-branching)
- [14. The exported script file format](#14-the-exported-script-file-format)

---

## 1. How the engine runs your script (read this first)

The plugin does **not** run your tasks like a numbered program with a "current line". Instead it
**re-scans your task list from the top, over and over**, and runs the first task whose conditions are
currently met.

Each pass:

1. It reads the list **top to bottom**.
2. For each task it checks:
   - Is the task **enabled**? (Disabled tasks are skipped.)
   - Has it **already run this cycle**? (Each task runs at most once per cycle.)
   - Are **all of its conditions** true? (See §7.)
   - Is the task's own built-in precondition true? (e.g. **Deposit Item** only fires while the bank is
     open; **Walk To** only fires while you are more than 2 tiles away.)
3. It runs the **first** task that's ready, then waits that task's delay before the next pass.
4. A task that **isn't ready yet** is simply skipped this pass and re-checked next pass. This is what
   lets a condition-gated script **wait for the right game state** instead of racing ahead.

**End of the list:**
- **Loop ON** → every task is re-armed and the list runs again. (See §11.) This is what makes a
  banking/skilling script refill and repeat automatically.
- **Loop OFF** → the script runs the list once, then just idles. It does **not** auto-stop: the button stays
  red **STOP** until you click it (or a **Stop Script** task runs).

**Reactive prayer is special.** The three **Prayer Trigger** tasks (§10) are not run in the top-to-bottom
scan. They react instantly to game events (a chat line, an incoming projectile, an NPC attack
animation) and flip your overhead prayer immediately — they never wait for task order.

> Tip: because the engine waits for conditions, you usually write each step with the conditions that
> describe **"when should this happen"** rather than trying to order steps perfectly. The presets in
> §12 are great examples to copy.

---

## 2. The task editor — common fields

Double-click a task (or click **+** to add one) to open the editor. The fields that appear depend on
the **Task Type** you pick; these are the common ones.

> Container tasks behave differently: double-clicking a **condition block** (📋) or group opens its
> **sub-task list** instead of the editor. To edit the block's own settings (its test, coordinates,
> radius, …), use the ✏ **Edit** button on its card.

| Field (on screen) | What it does |
|-------------------|--------------|
| **Task Name** | The label shown for the task in the list. |
| **Task Type** | The dropdown that chooses what the task does (see §5). A one-line description shows under it. |
| **Enabled** | Unchecking it skips the task entirely. |
| **Invert Condition (NOT)** | For condition blocks: flips the test (run children when the condition is **false**). |
| **Item Name** (relabeled per type) | The item/NPC/object/food/prayer the task acts on. The label changes per type (e.g. "Item 1:", "Food list (comma):", "NPC Name/ID:", "Prayer name:"). |
| **Item 2 / Action** | The second item, or the right-click action, depending on type. |
| **Spell Name** | The spell to cast. |
| **Quantity** (relabeled per type) | A number whose meaning depends on type (e.g. "Wait (ticks):", "Min Count:", "Region ID:", "Max Distance:", "Value:", "By:"). |
| **Menu Option** | The right-click action / make-menu option (label is "Action:", "Option Name:", or "Widget Info:" per type). |
| **Post-action delay** + **Ticks** toggle + **Timing variance (%)** | How long to wait after this task, and a random ± wobble on that wait. |
| **Coordinates (X / Y / Z)** | A tile, for movement / location tasks. |
| **Anim detect grace** + **Grace variance (%)** | For animation-related tasks: how long to wait for an animation to begin. |
| **Click type** | How the click is performed (see §3). |
| **Conditions (all must be true)** | The inline condition editor (see §7). |

**Delays in ticks.** One game tick is ~600 ms. If you tick the **Ticks** box, your delay is counted in
ticks; otherwise it's milliseconds. **Timing variance (%)** randomizes it (e.g. 20% on a 5-tick delay
means roughly 4–6 ticks each time) to keep timings human.

**Example — eat with humanized pacing:**
```
Type: Eat Food
Food list (comma): Shark, Manta ray
Post-action delay: 1 tick
Timing variance: 15%
```

---

## 3. Click type (Legit / Bypass / Hybrid)

Every interaction task has a **Click type** dropdown:

| Option | What it does |
|--------|--------------|
| **Legit** | A normal in-game click. The default; behaves like you clicking yourself. |
| **Bypass** | A direct interaction with no visible mouse click. |
| **Hybrid** | A mix of the two. |

This applies to item, NPC, prayer, and quick-prayer tasks. **Object clicks always use Legit** regardless
of what you pick. Reactive prayer flips (§10) always handle their own clicks internally.

**Example — equip a weapon without a visible click:**
```
Type: Equip Item
Item Name: Abyssal whip
Click type: Bypass
```

---

## 4. The idle-retry option (Expect animation)

Several action tasks offer an **Expect animation (retry if idle)** checkbox. When ticked, after the
task clicks, the plugin watches for **progress** (an animation starting / you interacting). If nothing
happens within the window, it **clicks again**. If it's still stuck after the limits, the task is
marked stuck and the engine moves on (it never spins forever). This is perfect for "click the resource
and keep going if I idle between actions" loops (battlestaves, mining, fishing, woodcutting).

Extra fields shown when it's ticked:

| Field | What it does |
|-------|--------------|
| **Idle window (ms)** | How long to wait for progress before re-clicking. 0 = off. |
| **Max retries** | How many times to re-click. 0 = default (3). |
| **Timeout (ticks)** | Hard ceiling for the task before it gives up. 0 = default (50). |
| **Expected anim id** | If set, only that exact animation counts as progress. `-1` = any animation counts. |

**Safety:** before re-clicking, the task re-checks its conditions. So a one-shot/consuming action
(eat, drink, special attack) guarded by a condition won't over-fire — once it's done, the condition has
already flipped.

This option is offered on: **Eat Food, Drink Potion, Special Attack, Attack NPC, Click Object, Click
NPC, Use Item on Object, Use Item on Item, Pickup Ground Item, Inventory Click, Walk To Object, Walk To
NPC**. (The making step of every skilling preset uses it — see §12.)

**Example — mine until the rock is gone, re-clicking if you idle:**
```
Type: Click Object
Object Name/ID: Iron rocks
Action: Mine
Expect animation (retry if idle): ✓
Idle window: 1800 ms
Max retries: 3
```

---

## 5. Task types — actions

These are the actions you can pick in the **Task Type** dropdown. (Condition blocks have their own
section, §6.) Field labels below are the ones the editor shows for that type.

### Bank operations

**Open Bank** — opens the nearest bank. Only fires while the bank is closed. No fields.

**Close Bank** — closes the bank. Only fires while the bank is open. No fields.

**Deposit All** — deposits your whole inventory, or all of one item if you fill in **Item Name**. Fires
while the bank is open.

**Deposit Item** — deposits a specific item.
- **Item Name** (name or id), **Quantity** (`-1` = all).
- Example: `Type: Deposit Item, Item Name: Logs, Quantity: -1`.

**Withdraw Item** — withdraws an item from the bank.
- **Item Name** (name or id), **Quantity** (`-1` or any number over 28 = withdraw-all).
- Example: `Type: Withdraw Item, Item Name: Ranarr potion (unf), Quantity: 14`.

**Deposit All Except** — deposits everything except a keep-list.
- **Keep list (comma)**.
- Example: `Type: Deposit All Except, Keep list (comma): Coins, Abyssal whip`.

**Deposit Equipment** — deposits all worn gear. No fields. Fires while the bank is open.

**Withdraw Noted** — withdraws an item in noted form.
- **Item Name/ID** (an id works best), **Withdraw qty**.
- Example: `Type: Withdraw Noted, Item Name/ID: 1515, Withdraw qty: 1000`.

### Item operations

**Use Item on Item** — combines two inventory items. Fires while the bank is closed.
- **Item 1**, **Item 2 / Action**.
- Example: `Type: Use Item on Item, Item 1: Pestle and mortar, Item 2: Snape grass`.

**Use Item on Object** — uses an inventory item on a game object.
- **Item Name** (the item), **Item 2 / Action** (the object name), **Action**.

**Drop Item** — drops a specific item. **Item Name**.

**Equip Item** — wears/wields an item from your inventory. **Item Name**.

**Inventory Click** — clicks an inventory item with a chosen action.
- **Item Name** (name or id), **Item 2 / Action** (blank = first action).
- Example: `Type: Inventory Click, Item Name: Dramen staff, Item 2 / Action: Wield`.

**Pickup Ground Item** — takes a ground item. Fires while the bank is closed.
- **Ground item(s)** (comma-separated).
- Example: `Type: Pickup Ground Item, Ground item(s): Big bones, Dragon bones`.

### Interaction operations

**Click Object** — interacts with the nearest matching object.
- **Object Name/ID**, **Action** (blank = first option).
- Example: `Type: Click Object, Object Name/ID: Bank booth, Action: Bank`.

**Click NPC** — interacts with the nearest matching NPC.
- **NPC Name/ID**, **Action** (blank = first option).
- Example: `Type: Click NPC, NPC Name/ID: Banker, Action: Bank`.

**Click Widget** — clicks an on-screen interface element by its `group:child` numbers.
- **Widget Info** = `group:child`.
- Example: `Type: Click Widget, Widget Info: 12:34`.

**Attack NPC** — attacks an NPC; skips itself if you're already attacking a matching target. Fires
while the bank is closed.
- **NPC Name/ID**, **Item 2 / Action** (default "Attack").
- Example: `Type: Attack NPC, NPC Name/ID: Goblin`.

### Spell operations

**Cast Spell** — casts a spell. **Spell Name**.

**Cast Spell on Item** — casts a spell onto an inventory item. Fires while the bank is closed.
- **Spell Name**, **Item Name**.
- Example: `Type: Cast Spell on Item, Spell Name: High Level Alchemy, Item Name: Yew longbow`.

### Wait / delay operations

**Wait Ticks** — waits a fixed number of game ticks.
- **Wait (ticks)**, optional **Timing variance (%)**.
- Example: `Type: Wait Ticks, Wait (ticks): 3, Timing variance: 20%`.

**Wait Animation** — waits while you are animating (until you go idle or it times out).
- **Anim timeout (ticks)**, plus the grace fields.

**Wait Idle** — waits until you stop animating.

**Wait Animation Start** — waits for an animation to **begin** (use right after a click). Uses the
grace window as its timeout.

**Wait Animation Cycle** — waits for a full start→stop animation cycle. Best for "do the action and
wait until it finishes".

### Movement & navigation

**Walk To** — walks to a tile and waits until you're within 2 tiles. Only fires while you're more than
2 tiles away.
- **Coordinates (X / Y / Z)**.
- Example: `Type: Walk To, X: 3164, Y: 3486, Z: 0`.

**Walk To Object** — optionally walks to a staging tile, then interacts with an object. Fires while the
bank is closed.
- **Object Name/ID**, **Item 2 / Action**, optional **Coordinates** (0,0,0 = use the object's location).
- Example: `Type: Walk To Object, Object Name/ID: Tree, Item 2 / Action: Chop down`.

**Walk To NPC** — optionally walks to a staging tile, then interacts with an NPC.
- **NPC Name/ID**, **Item 2 / Action** (default "Attack"), optional **Coordinates**.

**Teleport** — teleports via a spell or an item.
- **Spell/Item name**, and an **Action** for the item (default tries Rub/Break/Teleport).
- Example: `Type: Teleport, Spell/Item name: Varrock teleport`.

**Step To Tile** — one single walk click on an exact tile, with no pathing and no "am I there yet"
loop. This is the dodge primitive: sidestep an AoE, then get back to attacking.
- **Coordinates (X / Y / Z)** — the tile, or offsets when **Mode** is `relative`.
- **Mode (blank/relative)** — `relative` makes X/Y offsets from your current tile (negative = west/south),
  which also works inside boss instances. Example: `Mode: relative, X: 2, Y: 0` steps 2 tiles east.

**Toggle Run** — flips run via the minimap orb.
- **on / off (blank=toggle)** — `on`/`off` ensure that state; blank just toggles.

### Menu operations

**Select Menu Option** — clicks an option in the make/skill menu (the "what would you like to make?"
popup).
- **Option Name** = "Make" or an option number (default = the first make button). Fires only while the
  menu is open.
- Example: `Type: Select Menu Option, Option Name: Make`.

### Dialog operations

**Dialog Continue** — clicks "Click here to continue" whenever a dialog with continue-text is up.
Fires only while such a dialog is open. No fields.

**Dialog Select Option** — picks a dialog option by its text. Fires only while an option menu is up.
- **Option text** — e.g. `Yes.` or a number like `2`.

### Custom

**Custom Action** — runs a custom command string.
- **Custom** = the command string.

### Combat / consumable / prayer / spec

**Eat Food** — eats the first food present from your list.
- **Food list (comma)**.
- Example: `Type: Eat Food, Food list (comma): Shark, Manta ray, Karambwan`.

**Drink Potion** — drinks the first potion present (any dose).
- **Potion list (comma)**.
- Example: `Type: Drink Potion, Potion list (comma): Prayer potion, Super restore`.

**Activate Prayer** / **Deactivate Prayer** — turns one prayer on / off.
- **Prayer name**.
- Example: `Type: Activate Prayer, Prayer name: PROTECT_FROM_MELEE`.

**Deactivate All Prayers** — turns off every active prayer.
- **Max disables** (default 10).

**Toggle Quick Prayer** — turns quick prayers on (or off if **Invert Condition (NOT)** is ticked).

**Special Attack** — clicks the special-attack orb when your spec energy is at least your minimum.
- **Min spec %** (default 50).
- Example: `Type: Special Attack, Min spec %: 50`.

**Gear Swap** — equips a whole comma-list of items in one game tick, skipping pieces you already
wear. This is the PvM switch primitive (mage set ↔ range set, spec weapon in/out).
- **Gear list (comma)**, **Click type**.
- Example: `Type: Gear Swap, Gear list (comma): Toxic blowpipe, Ava's assembler, Black d'hide body`.

**Combo Eat** — consumes every present item from a comma-list back-to-back in one tick — the classic
food + karambwan (+ brew/restore) combo.
- **Consume list (comma)**, **Click type**.
- Example: `Type: Combo Eat, Consume list (comma): Manta ray, Cooked karambwan`.

> Note: there are also three simple **If Has Item / If Bank Open / If Animating** tasks in the dropdown,
> but they **do not gate the tasks after them** — they're left over from older scripts. To make steps
> conditional, use a **condition block** (§6) or attach **inline conditions** to a task (§7).

---

## 6. Condition blocks (📋)

A **condition block** is a container task whose name starts with the 📋 clipboard icon. The tasks you
drop **inside** it only run when the block's test is **true**; otherwise the whole group is skipped.
Tick **Invert Condition (NOT)** to flip the test. Blocks can be nested.

**Viewing the tasks inside a block.** A container's card shows **▸ name (count)** with the number of
sub-tasks it holds. Double-click the card (or click its 📁 **Open** button) to open the sub-task list —
you can add, edit, reorder and delete sub-tasks there just like at the top level, and the **back arrow**
returns you up a level. While the script is running, the green highlight follows execution everywhere:
the container card glows while anything inside it runs, and inside the block view the exact running
sub-task glows. The ✏ **Edit** button on the card edits the block's own settings (double-click is
reserved for opening it).

### Simple (yes/no) blocks

| Block (in the dropdown) | Runs its children when… | Fields |
|-------------------------|-------------------------|--------|
| 📋 Condition: Has Item | inventory has the item | Item Name |
| 📋 Condition: No Item | inventory does **not** have the item | Item Name |
| 📋 Condition: Bank Open | the bank is open | — |
| 📋 Condition: Bank Closed | the bank is closed | — |
| 📋 Condition: Animating | you are animating | — |
| 📋 Condition: Idle | you are idle | — |
| 📋 Condition: Inv Full | inventory is full | — |
| 📋 Condition: Inv Empty | inventory is empty | — |
| 📋 Condition: Inv Count | you have at least **Min Count** of the item | Item Name, Min Count |
| 📋 Condition: Menu Open | the make/skill menu is open | — |
| 📋 Condition: NPC Exists | a matching NPC is nearby | NPC Name/ID, Max Distance (0 = any) |
| 📋 Condition: Object Exists | a matching object is nearby | Object Name/ID, Max Distance |
| 📋 Condition: In Region | you are in the region | Region ID |
| 📋 Condition: In Combat | you are in combat / interacting | — |
| 📋 Condition: Item Equipped | any of the listed items is worn | Item(s) (comma) |
| 📋 Condition: Ground Item | a matching ground item is present | Ground item(s), Max Distance |
| 📋 Condition: Target Name | the NPC you're fighting matches a name | Target name(s) (comma) |
| 📋 Condition: NPC Attacking Me | a matching NPC is attacking **you** (blank name = any) | NPC Name/ID, Max Distance (0 = any) |
| 📋 Condition: Dialog Open | a dialog is up (optionally: a specific option exists) | Option text (optional) |
| 📋 Condition: Widget Visible | an interface element is on screen | Widget group:child |

### Numeric blocks (use the comparator + threshold — see §8)

| Block | Compares | Fields |
|-------|----------|--------|
| 📋 Condition: HP | your HP (or % of full if you tick **%**) | Comparator, Threshold, % |
| 📋 Condition: Prayer | your prayer points (or %) | Comparator, Threshold, % |
| 📋 Condition: Run Energy | your run energy | Comparator, Threshold |
| 📋 Condition: Spec Energy | your special-attack % | Comparator, Threshold |
| 📋 Condition: Skill Level | a skill's level (boosted or real) | Comparator, Threshold, Skill name, Boosted |
| 📋 Condition: Bank Contains | how many of an item the bank holds (bank must be open) | Comparator, Min Count, Item Name |
| 📋 Condition: Var Compare | one of your variables (§9) | Comparator, Value, Variable name |
| 📋 Condition: Target HP | your current target's HP — tick **%** for percent of its healthbar (needs a visible healthbar) | Comparator, Threshold, % |
| 📋 Condition: Varbit | a game varbit's value (quest stages, boss phases, cooldowns…) | Varbit ID, Comparator, Threshold |
| 📋 Condition: VarPlayer | a game varplayer's value (e.g. auto-retaliate) | VarPlayer ID, Comparator, Threshold |

### Location block

| Block | Runs its children when… | Fields |
|-------|-------------------------|--------|
| 📋 Condition: At Location | you are within the radius of a tile | Coordinates (X/Y/Z), Threshold = radius |

**Example — eat only when HP is below 50%:**
```
📋 Condition: HP    Comparator: <    Threshold: 50    % : ✓
   └─ Eat Food      Food list (comma): Shark
```

**Example — bank only when the inventory is full:**
```
📋 Condition: Inv Full
   ├─ Open Bank
   ├─ Deposit All
   └─ Close Bank
```

---

## 7. Inline conditions (per-task)

Instead of wrapping a single step in a block, you can attach conditions directly to a task in the
editor's **Conditions (all must be true)** section. Click the **+** there to add a row. **Every** row
must be true for the task to run that pass. They use the **same tests as the blocks** in §6.

Each condition row lets you set: the condition type, an item/NPC/object name (where it applies), a
**dist** (for the "exists" checks), a **comparator + value** (for numeric checks), a **%** toggle
(HP/Prayer), a **skill + B** (boosted) for Skill Level, a **variable name** for Var Compare, a grace
window, and a **NOT** toggle to invert that single row.

**Example — attack only while a Goblin is within 8 tiles AND you're not already fighting:**
```
Type: Attack NPC, NPC Name/ID: Goblin
Conditions (all must be true):
  ├─ NPC Exists   name: Goblin   dist: 8
  └─ In Combat    NOT: ✓
```

---

## 8. The comparator

Numeric conditions (blocks and inline rows) compare with a **Comparator** dropdown:

| Symbol | Meaning |
|--------|---------|
| `<`  | less than |
| `<=` | less than or equal |
| `>`  | greater than |
| `>=` | greater than or equal |
| `==` | equal |

If you leave it blank, it defaults to `>=`.

---

## 9. Variables & repetition

Variables are simple **named whole-number counters** you can set, increase, decrease, and compare. A
variable you've never set reads as **0**. All variables reset to empty (read 0) every time you press
**Start**.

### Set Var
Sets a variable to a number.
- **Variable name**, **Value**.
- Example: `Type: Set Var, Variable name: crafted, Value: 0`.

### Inc Var
Increases a variable.
- **Variable name**, **By** (default 1).
- Example: `Type: Inc Var, Variable name: crafted, By: 1`.

### Dec Var
Decreases a variable.
- **Variable name**, **By** (default 1).

### 📋 Condition: Var Compare
A condition block (or inline condition) that compares a variable against a value (§6/§7).
- **Variable name**, **Comparator**, **Value**.

### Stop Script
Stops the whole script immediately. No fields.

### "Do N times, then stop"

Because each task runs **once per cycle** with Loop ON, an **Inc Var** counts cycles. To craft/gather
20 cycles and then stop:

```
(Loop ON)
... your work tasks ...
Inc Var          Variable name: n   By: 1
📋 Condition: Var Compare   Variable name: n   Comparator: >=   Value: 20
   └─ Stop Script
```

Don't put a **Set Var n = 0** inside the loop body — the start-of-run reset already zeroes it, and a
Set Var in the body would re-zero `n` every cycle.

---

## 10. Reactive PvM prayer

The three **Prayer Trigger** tasks are **not** run in the normal top-to-bottom scan. When you press
**Start**, each one becomes a rule that flips your overhead prayer the instant a matching game event
happens — pre-empting the rest of the script. When you Stop (or reload), the rules are dropped and any
reactive overhead is turned back off.

Common fields:

| Field | What it does |
|-------|--------------|
| **Trigger prayer** | The overhead prayer to flip on. Dropdown: `PROTECT_FROM_MELEE`, `PROTECT_FROM_MAGIC`, `PROTECT_FROM_MISSILES`. |
| **Priority** | If two triggers fire at once, the **lower** number wins. Default 1. |

### Prayer Trigger: Chat
When an incoming chat line **contains** your phrase, switch to your prayer and **hold** it until the
next chat trigger replaces it.
- **Chat phrase** (case-insensitive; just the distinctive fragment), **Trigger prayer**.
- Example:
```
Type: Prayer Trigger: Chat
Chat phrase: sphere of magical power
Trigger prayer: PROTECT_FROM_MAGIC
```
This is what the **Olm Basics** preset uses (§12).

### Prayer Trigger: Projectile
Pre-prays based on an **incoming projectile id** that you capture in-client. The prayer comes on a set
number of ticks before the hit and holds through the impact tick.
- **Projectile id**, **Trigger prayer**, **Lead ticks** (how early to pray; default 1 = on the impact
  tick), **Priority**.
- Example:
```
Type: Prayer Trigger: Projectile
Projectile id: 2681
Trigger prayer: PROTECT_FROM_MAGIC
Lead ticks: 1
```

### Prayer Trigger: Animation
Schedules the prayer a set number of ticks **after** a matching NPC attack animation (for melee/contact
attacks with no projectile and no chat line). Capture the animation id in-client.
- **NPC anim id**, **Trigger prayer**, **Anim->hit ticks** (default 0), **Priority**.

> A trigger with a blank/unknown prayer, a blank chat phrase, or an id of `-1` simply never fires —
> nothing crashes. Fill in the real projectile/animation id captured in a live client first.

You can freely mix triggers with normal tasks — e.g. an **Eat Food** under a **Condition: HP < 50%**
block, a **Special Attack** under **Condition: Spec Energy >= 50**, plus three chat triggers, all in
the same Loop-ON script.

---

## 11. The Loop toggle

The side panel has a **Loop** toggle (on by default). It controls what happens at the end of each scan:

- **Loop ON** → at the end of the list, every task is re-armed and the list runs again after a short
  delay. This is what makes variables count per cycle and what makes a banking/skilling script refill
  and repeat forever.
- **Loop OFF** → the list runs once top-to-bottom, then idles. There is no auto-stop: the button stays red
  **STOP** until you click it (or a **Stop Script** task runs).

**Example:** a banking/crafting script you want running forever → **Loop ON**. A one-time setup
(withdraw gear, walk somewhere, equip) → **Loop OFF**.

The panel shows the short version right next to the toggle — **"Loop — ON: repeat · OFF: run once"** —
and hovering it shows the full explanation.

### Trigger hotkeys (Loop OFF)

When **Loop is OFF**, you can bind a keyboard hotkey that **runs a script once** on demand — turning a
script into a one-press macro (a gear-swap, a spec rotation, a "bank now" sequence). Pressing the key
**triggers a run**; it does **not** turn the script on or off.

How to set one:
- The hotkey option only appears with **Loop OFF** (with Loop ON the list already repeats, so there's
  nothing to trigger).
- On the script card, tick the **Hotkey** checkbox — a capture button appears next to it.
- Click the button, press your key (or combo). It saves instantly. Un-tick the checkbox (or left-click
  the button) to clear it.

Rules and behavior:
- **Any key works** — it's captured at the game window and **blocked from the game while it's bound**,
  so it won't type in chat. Because of that, avoid binding a key you still need to play (a letter you
  type, your run/prayer key, etc.); an F-key or a Ctrl/Alt combo is usually safest.
- Pressing the key runs the script's task list once. If the script isn't started yet it starts it;
  if it's already started and idle, it re-runs the list. The status line confirms (e.g. "Hotkey: ran
  Spec dump"). Press **STOP** to stop as usual.
- The key only intercepts input while **Loop is OFF**; switch Loop ON and the key is released back to
  the game.
- Holding the key fires once, not repeatedly.
- Hotkeys are per-setup: **Export keeps them, Import clears them** (rebind after importing so two
  scripts never share one key). Bind each key to only one script.

## 12. Presets

The **Presets ▾** button drops a ready-made, complete script into your list. Pick a category, then an
item. The skilling presets are full "bank → withdraw → make → bank" loops that repeat automatically
when **Loop** is on; the making step has the idle-retry option turned on so a missed click is re-issued.

The complete **Presets ▾** menu:

- **Herblore** — Attack Potion, Strength Potion, Prayer Potion, Super Attack, Super Strength, Super
  Restore, Saradomin Brew.
- **Unf Potions** — Guam (unf), Ranarr (unf), Toadflax (unf), Snapdragon (unf).
- **Crafting** — Green D'hide, Blue D'hide, Black D'hide.
- **Gem Cutting** — Sapphires, Emeralds, Rubies, Diamonds.
- **Fletching** — Arrow Shafts, Yew Longbows, Magic Longbows.
- **Bow Stringing** — Yew Longbows, Magic Longbows.
- **Magic** — Plank Make (Oak), Plank Make (Mahogany), High Alch.
- **Battlestaves** — Air, Water, Earth, Fire.
- **Glass Blowing** — Unpowered Orbs, Lantern Lens.
- **Boss** — Olm Basics.

There is no Auto-Fighter or Gatherer item in the menu — those you build yourself with the tasks above.

### What a skilling preset looks like

All the skilling presets share the same shape (each step has conditions describing when it should run):

1. **Open Bank** — when the bank is closed and you don't have the main material.
2. **Deposit All** — when you don't have the main material.
3. **Withdraw** the tool/material — when the bank is open and you don't have it yet.
4. (more **Withdraw** steps for secondary materials, in order)
5. **Close Bank** — when the bank is open and you have everything.
6. **Use Item on Item** (the making step, idle-retry on) — when the bank is closed, you have the
   materials, the make menu is closed, and you're idle.
7. **Wait Animation** — waits for the making to finish.
8. **Select Menu Option** "Make" — when the make menu is open.

Two presets are spell-based (**Magic → Plank Make** and **Magic → High Alch**): they withdraw a full
inventory, close the bank, then **Cast Spell on Item** on each slot — no make-menu step. (Detailed
step-by-step guides for each category live in the `scripts/` folder.)

### Boss → Olm Basics

Inserts a group named **"Olm Basics"** containing **three editable Prayer Trigger: Chat tasks**, one
per Great Olm basic-attack sphere. Open the group to view or edit them:

| Task | Chat phrase | Trigger prayer |
|------|-------------|----------------|
| Olm: Melee sphere | sphere of aggression | PROTECT_FROM_MELEE |
| Olm: Magic sphere | sphere of magical power | PROTECT_FROM_MAGIC |
| Olm: Range sphere | sphere of accuracy and dexterity | PROTECT_FROM_MISSILES |

Each holds its overhead prayer until the next sphere message flips it. There is **no "Olm" task type and
no Olm checkbox** — it is only this preset of generic chat triggers, which you can copy for any boss
that announces its attacks in chat.

---

## 13. Repeating and branching

There is no "jump to step" or "repeat this block" control — repetition and branching are built entirely
from the **Loop** toggle, conditions, and variables.

**To repeat forever:** turn **Loop ON** — the list runs again from the top after each pass.

**To repeat a set number of times then stop:** turn **Loop ON**, then use **variables +
📋 Condition: Var Compare + Stop Script** (the counting recipe in §9).

**To branch (run some steps only sometimes):** use **condition blocks** (§6) or **inline conditions**
(§7) to gate the steps you want.

**Olm prayers** are set up from **Presets ▾ → Boss → Olm Basics** — a group of three editable chat
triggers (§12); there is no "Olm" task type or checkbox.

If you import an old script that contains a task type this version doesn't recognize, that one entry is
simply skipped — the rest of the script still loads.

---

## 14. The exported script file format

The **Export** button on a script saves it as a pretty-printed `.json` file; **Import** loads one back.
This section documents that file so you (or an AI agent) can author scripts by hand.

A file is a single script. **Every entry — the script itself, a condition block, and each task — is the
same object shape.** A script/block holds its children in `childScripts`; a plain task has `null`
there. Conditions attached to a task/block live in its `conditions` array.

### Keys on every entry

| Key | Type | Default | Meaning |
|-----|------|---------|---------|
| `id` | string | (8-char id) | Unique id; a new one is generated on import. |
| `name` | string | `""` | The task's display name. |
| `type` | string | — | The task type (see the list below). A missing/unknown value makes the entry skip on import. |
| `enabled` | bool | `true` | Disabled entries are skipped. |
| `priority` | int | `0` | Not used for ordering (order = position in the array). |
| `itemName1` | string | `""` | Main item/NPC/object/food/prayer (meaning per type). |
| `itemName2` | string | `""` | Second item, or the action. |
| `quantity` | int | `1` | A number whose meaning depends on type (count / region / distance / value / timeout ticks). |
| `delay` | int | `0` | Post-action delay (ms, or ticks if `delayInTicks`). |
| `delayInTicks` | bool | `false` | Treat `delay` as ticks. |
| `randomizePercent` | int | `0` | ± wobble on the delay. |
| `spellName` | string | `""` | The spell to cast. |
| `customAction` | string | `""` | The menu option / custom command / teleport verb. |
| `invertCondition` | bool | `false` | Invert a condition block's test. |
| `walkX`, `walkY`, `walkZ` | int | `0` | Tile / staging tile / location-block tile. |
| `graceMs` | int | `2` | Animation grace; with `graceInTicks` true this is ticks. |
| `graceInTicks` | bool | `true` | Treat `graceMs` as ticks. |
| `graceRandomizePercent` | int | `0` | ± wobble on the grace window. |
| `isGroup` | bool | — | `true` for the script and for condition blocks; `false` for plain tasks. |
| `childScripts` | array | `null` | Child entries (for the script and for condition blocks). |
| `conditions` | array | `null` | Inline conditions (all must be true). See below. |
| `op` | string | `""` | Comparator symbol for a numeric block; `""` means `>=`. |
| `threshold` | int | `0` | Numeric threshold for a numeric block. |
| `thresholdIsPercent` | bool | `false` | HP/Prayer block: percent vs absolute. |
| `skillName` | string | `""` | Skill Level block (e.g. `MINING`). |
| `useBoostedLevel` | bool | `true` | Skill block: boosted vs real level. |
| `clickType` | int | `0` | Click type: 0 Legit, 1 Bypass, 2 Hybrid. |
| `minSpecPercent` | int | `50` | Special Attack minimum spec %. |
| `varName` | string | `""` | Variable name (Set/Inc/Dec Var and Var Compare). |
| `expectAnimation` | bool | `false` | Idle-retry option on/off. |
| `retryIfIdleMs` | int | `0` | Idle-retry: no-progress window in ms (0 = off). |
| `maxRetries` | int | `0` | Idle-retry: max re-clicks (0 = default 3). |
| `timeoutTicks` | int | `0` | Idle-retry: hard ceiling in ticks (0 = default 50). |
| `expectedAnimationId` | int | `-1` | Idle-retry: require this exact animation id (-1 = any). |
| `triggerMatchId` | int | `0` | Prayer trigger: projectile id / NPC anim id (Chat ignores it). |
| `triggerPrayer` | string | `""` | Prayer trigger: the overhead prayer, e.g. `PROTECT_FROM_MAGIC`. |
| `triggerPhrase` | string | `""` | Chat trigger: case-insensitive substring to match. |
| `triggerHitDelayTicks` | int | `0` | Animation: anim→hit ticks. Projectile: lead (default 1). Chat: 0 (hold). |
| `triggerPriority` | int | `1` | Prayer trigger: lower wins when several fire at once. |

### A condition row (inside `conditions`)

| Key | Type | Default | Meaning |
|-----|------|---------|---------|
| `type` | string | — | One of the condition types. |
| `itemName` | string | `""` | Item/NPC/object name (or comma list). |
| `quantity` | int | `1` | Count / region / **the numeric value to compare against** for inline numeric conditions. |
| `invert` | bool | `false` | "NOT". |
| `graceMs` | int | `2` | Grace window. |
| `graceInTicks` | bool | `true` | Treat `graceMs` as ticks. |
| `graceRandomizePercent` | int | `0` | ± wobble on the grace window. |
| `distance` | int | `0` | Max distance for NPC/Object/Ground-item checks (0 = any). |
| `op` | string | `""` | Comparator; `""` means `>=`. |
| `thresholdIsPercent` | bool | `false` | HP/Prayer: percent vs absolute. |
| `skillName` | string | `""` | Skill Level skill. |
| `useBoostedLevel` | bool | `true` | Boosted vs real. |
| `varName` | string | `""` | Var Compare variable. |

> Important difference: a **block's** numeric value is in `threshold`; an **inline condition's** numeric
> value is in `quantity`.

### `type` values

Use these exact strings in the `type` key. Each is followed by its on-screen name.

**Actions:**
`OPEN_BANK` (Open Bank), `CLOSE_BANK` (Close Bank), `DEPOSIT_ALL` (Deposit All), `DEPOSIT_ITEM`
(Deposit Item), `WITHDRAW_ITEM` (Withdraw Item), `DEPOSIT_ALL_EXCEPT` (Deposit All Except),
`DEPOSIT_EQUIPMENT` (Deposit Equipment), `WITHDRAW_NOTED` (Withdraw Noted), `USE_ITEM_ON_ITEM` (Use Item
on Item), `USE_ITEM_ON_OBJECT` (Use Item on Object), `DROP_ITEM` (Drop Item), `EQUIP_ITEM` (Equip Item),
`INVENTORY_CLICK` (Inventory Click), `PICKUP_GROUND_ITEM` (Pickup Ground Item), `CLICK_OBJECT` (Click
Object), `CLICK_NPC` (Click NPC), `CLICK_WIDGET` (Click Widget), `ATTACK_NPC` (Attack NPC), `CAST_SPELL`
(Cast Spell), `CAST_SPELL_ON_ITEM` (Cast Spell on Item), `WAIT_TICKS` (Wait Ticks), `WAIT_ANIMATION`
(Wait Animation), `WAIT_IDLE` (Wait Idle), `WAIT_ANIMATION_START` (Wait Animation Start),
`WAIT_ANIMATION_CYCLE` (Wait Animation Cycle), `WALK_TO` (Walk To), `WALK_TO_OBJECT` (Walk To Object),
`WALK_TO_NPC` (Walk To NPC), `TELEPORT` (Teleport), `SELECT_MENU_OPTION` (Select Menu Option), `CUSTOM`
(Custom Action), `EAT_FOOD` (Eat Food), `DRINK_POTION` (Drink Potion), `ACTIVATE_PRAYER` (Activate
Prayer), `DEACTIVATE_PRAYER` (Deactivate Prayer), `DEACTIVATE_ALL_PRAYERS` (Deactivate All Prayers),
`TOGGLE_QUICK_PRAYER` (Toggle Quick Prayer), `SPECIAL_ATTACK` (Special Attack).

**Variables & control:**
`SET_VAR` (Set Var), `INC_VAR` (Inc Var), `DEC_VAR` (Dec Var), `STOP_SCRIPT` (Stop Script).

**Prayer triggers:**
`PRAYER_TRIGGER_CHAT` (Prayer Trigger: Chat), `PRAYER_TRIGGER_PROJECTILE` (Prayer Trigger: Projectile),
`PRAYER_TRIGGER_ANIMATION` (Prayer Trigger: Animation).

**Condition blocks (also used as inline condition types):**
`CONDITION_HAS_ITEM`, `CONDITION_NO_ITEM`, `CONDITION_BANK_OPEN`, `CONDITION_BANK_CLOSED`,
`CONDITION_ANIMATING`, `CONDITION_IDLE`, `CONDITION_INV_FULL`, `CONDITION_INV_EMPTY`,
`CONDITION_INV_CONTAINS` (Inv Count), `CONDITION_MENU_OPEN`, `CONDITION_NPC_EXISTS`,
`CONDITION_OBJECT_EXISTS`, `CONDITION_IN_REGION`, `CONDITION_IN_COMBAT`, `CONDITION_HP`,
`CONDITION_PRAYER`, `CONDITION_RUN_ENERGY`, `CONDITION_SPEC` (Spec Energy), `CONDITION_SKILL_LEVEL`,
`CONDITION_BANK_CONTAINS`, `CONDITION_ITEM_EQUIPPED`, `CONDITION_GROUND_ITEM_EXISTS` (Ground Item),
`CONDITION_AT_LOCATION`, `CONDITION_VAR_COMPARE`.

### Example exported task — eat when HP is low

```json
{
  "id": "a1b2c3d4",
  "name": "If HP Low",
  "type": "CONDITION_HP",
  "enabled": true,
  "op": "<",
  "threshold": 50,
  "thresholdIsPercent": true,
  "isGroup": true,
  "delay": 1,
  "delayInTicks": true,
  "randomizePercent": 15,
  "childScripts": [
    {
      "id": "e5f6a7b8",
      "name": "Eat Food",
      "type": "EAT_FOOD",
      "enabled": true,
      "itemName1": "Shark, Manta ray",
      "delay": 1,
      "delayInTicks": true,
      "randomizePercent": 15,
      "isGroup": false,
      "childScripts": null
    }
  ]
}
```

### Example exported task — attack with inline conditions

```json
{
  "id": "11aa22bb",
  "name": "Attack Goblin",
  "type": "ATTACK_NPC",
  "enabled": true,
  "itemName1": "Goblin",
  "itemName2": "Attack",
  "delay": 2,
  "delayInTicks": true,
  "isGroup": false,
  "childScripts": null,
  "conditions": [
    { "type": "CONDITION_NPC_EXISTS", "itemName": "Goblin", "distance": 8 },
    { "type": "CONDITION_IN_COMBAT", "invert": true }
  ]
}
```

### Example exported task — an Olm chat trigger

```json
{
  "id": "33cc44dd",
  "name": "Olm: Magic sphere",
  "type": "PRAYER_TRIGGER_CHAT",
  "enabled": true,
  "triggerPhrase": "sphere of magical power",
  "triggerPrayer": "PROTECT_FROM_MAGIC",
  "triggerPriority": 1,
  "isGroup": false,
  "childScripts": null
}
```
