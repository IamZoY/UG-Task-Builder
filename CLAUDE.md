# UG Task Builder — Script Authoring Guide (CLAUDE.md / AGENTS.md)

> **This file is the canonical authoring reference.** `AGENTS.md` in this folder is a thin pointer back here.
> Audience: **users** building scripts in the side panel, and **AI agents** writing script files for users.
> No coding required — you build scripts entirely from the panel, by combining tasks, condition blocks,
> inline conditions, variables, and reactive prayer triggers.

UG Task Builder is a **visual, no-code task-scripting plugin** for Old School RuneScape. Open the side panel
(toolbar tooltip **UG Task Builder**) and you'll see your **Scripts** (groups). Open a Script to view and edit
its ordered list of **Tasks**. The toolbar has a `Presets ▾` button, an **Import** button, a **+** (Add Task)
button, and an info/Documentation button. A **START** / **STOP** button runs the selected script, and a
**Loop** toggle (default **ON**) controls whether it repeats.

This document covers the mental model, the complete task vocabulary as it appears in the dropdown, the
condition blocks and inline conditions, variables, reactive prayer, the import/export file format, and
fully-worked example scripts you can Import and run.

---

## Table of Contents

1. [The mental model (read this first)](#1-the-mental-model-read-this-first)
2. [The complete task vocabulary](#2-the-complete-task-vocabulary)
3. [Inline conditions](#3-inline-conditions)
4. [Variables and the repetition idiom](#4-variables-and-the-repetition-idiom)
5. [Reactive PvM prayer](#5-reactive-pvm-prayer)
6. [The exported script file format](#6-the-exported-script-file-format)
7. [Fully-worked example scripts](#7-fully-worked-example-scripts)
8. [Guardrails — what NOT to do](#8-guardrails--what-not-to-do)

---

## 1. The mental model (read this first)

The panel checks your tasks **in order, top to bottom, over and over**, running each one only when its
conditions are met. There is no "line counter" and no "jump to step 3" — understanding this one model is
everything.

### How a pass works

A **script** is a top-level Script (group). Each pass over its task list goes **top to bottom**:

```
for each task, in order (top -> bottom):
    if the task's conditions are ALL true:
        run it
        (if it was a "real" action that does something in-game, the pass ends here
         and the next pass starts again from the top)
    else:
        skip it for now — it will be re-checked next pass
```

- **Conditions are the gate.** A task only runs when it is enabled, it hasn't already run this cycle, and
  **all** of its conditions are true. A task whose conditions aren't met is simply **skipped and retried next
  pass** — that is how a script *waits* for the right game state instead of racing ahead. (For example, an
  **Open Bank** task only fires while the bank is closed; a **📋 Condition: Bank Open** block only runs its
  children while the bank is open.)
- **One real action per pass.** Most tasks are "blocking": once one runs, the pass ends and the next pass
  starts over from the top. This paces the bot and lets game state settle between clicks. A handful of tasks
  are **instant / non-blocking** and don't end the pass — the **Set Var / Inc Var / Dec Var** tasks,
  **Stop Script**, the three **Prayer Trigger** tasks, and the flat **If …** tasks. The scan just keeps going
  past those.
- **Each task runs at most once per cycle.** Once a task fires, it won't fire again until the cycle restarts.

### The Loop toggle (the only repetition control)

There is a **Loop** toggle in the panel.

- **Loop OFF:** the script runs each task once (top to bottom, respecting conditions), then just **idles** — the
  tasks stay "done" and nothing else fires. There is no auto-stop: the button stays red **STOP** until you click
  it (or a **Stop Script** task runs).
- **Loop ON** (default): when the scan reaches the bottom, every task is **re-armed** and the scan starts over
  after a short delay. This is what makes a bankstander run forever, and what makes an **Inc Var** task count
  once per cycle.

There is **no "jump to step" or "repeat this block" control** — repetition is just **Loop ON**, and "do N times
then stop" and conditional branching are done with **variables** + the **📋 Condition: Var Compare** block +
the **Stop Script** task (see §4).

### Reactive prayer pre-empts the scan

Overhead protection-prayer flips do **not** wait their turn in the scan. They are **event-driven**: they react
the moment an attack telegraphs (a chat line, an incoming projectile, or an NPC attack animation), so the
prayer flips far faster than waiting for a task's turn. You set these up with the three **Prayer Trigger**
tasks. See §5.

---

## 2. The complete task vocabulary

Every entry below is a real task type **exactly as it appears in the Type dropdown** when you Add a task. The
`📋` prefix marks **condition blocks** — these wrap child tasks and only run their children when the condition
holds (and you can invert them with the **NOT** option). The right-hand "file type" column is the value that
appears under `type` in an exported `.json` (see §6); agents authoring files need it.

Most action tasks share these dialog fields: **Name**, **Type**, a **Delay** value with a **ticks/ms** toggle
and a **± randomize %**, an optional **Conditions** list, a **NOT** (invert) option, and a **Click type**
dropdown (LEGIT / BYPASS / HYBRID — leave on LEGIT unless you know you need otherwise).

### Bank operations
| Task (dropdown name) | File `type` | What it does |
|---|---|---|
| Open Bank | `OPEN_BANK` | Opens the nearest bank (only fires when the bank is closed). |
| Close Bank | `CLOSE_BANK` | Closes the bank (only fires when the bank is open). |
| Deposit All | `DEPOSIT_ALL` | Deposits everything. |
| Deposit Item | `DEPOSIT_ITEM` | Deposits a specific item (Quantity; -1 = all). |
| Withdraw Item | `WITHDRAW_ITEM` | Withdraws an item from the bank (Quantity; -1 or >28 = withdraw-all). |
| Deposit All Except | `DEPOSIT_ALL_EXCEPT` | Deposits everything except a keep-list (comma-separated names). |
| Deposit Equipment | `DEPOSIT_EQUIPMENT` | Deposits all worn equipment. |
| Withdraw Noted | `WITHDRAW_NOTED` | Withdraws an item as noted (name/ID + Quantity). |

### Item operations
| Task | File `type` | What it does |
|---|---|---|
| Use Item on Item | `USE_ITEM_ON_ITEM` | Uses Item 1 on Item 2. |
| Use Item on Object | `USE_ITEM_ON_OBJECT` | Uses an item (Item 1) on a game object (Item 2). |
| Drop Item | `DROP_ITEM` | Drops a specific item. |
| Equip Item | `EQUIP_ITEM` | Equips/wears/wields an item from inventory. |
| Inventory Click | `INVENTORY_CLICK` | Clicks an inventory item with a chosen action (action goes in Item 2 / Action). |
| Pickup Ground Item | `PICKUP_GROUND_ITEM` | Takes a ground item by name/ID (comma list). |

### Interaction
| Task | File `type` | What it does |
|---|---|---|
| Click Object | `CLICK_OBJECT` | Interacts with a game object (Menu Option = the action, e.g. "Mine"). |
| Click NPC | `CLICK_NPC` | Interacts with an NPC (Menu Option = the action). |
| Click Widget | `CLICK_WIDGET` | Clicks a widget by `group:child`. |
| Select Menu Option | `SELECT_MENU_OPTION` | Clicks a make/skill-menu option (Menu Option = "Make" or a number). |
| Attack NPC | `ATTACK_NPC` | Attacks an NPC by name/ID; skips if already attacking a match. |
| Custom Action | `CUSTOM` | Runs a custom action/command string. |

### Spells
| Task | File `type` | What it does |
|---|---|---|
| Cast Spell | `CAST_SPELL` | Casts a magic spell (Spell Name). |
| Cast Spell on Item | `CAST_SPELL_ON_ITEM` | Casts a spell on an inventory item (Spell Name + Item). |
| Teleport | `TELEPORT` | Teleports via a spell or an item. |

### Waits / delays
| Task | File `type` | What it does |
|---|---|---|
| Wait Ticks | `WAIT_TICKS` | Waits a number of game ticks (Quantity = ticks). |
| Wait Animation | `WAIT_ANIMATION` | Waits while the player is animating (Quantity = timeout ticks). |
| Wait Idle | `WAIT_IDLE` | Waits until the player is idle. |
| Wait Animation Start | `WAIT_ANIMATION_START` | Waits for an animation to START (use right after a click). |
| Wait Animation Cycle | `WAIT_ANIMATION_CYCLE` | Waits for a full animation cycle (start then stop). |

### Movement
| Task | File `type` | What it does |
|---|---|---|
| Walk To | `WALK_TO` | Walks to X,Y,Z and waits until arrived (within ~2 tiles). |
| Walk To Object | `WALK_TO_OBJECT` | Walks to (optional staging tile) then interacts with an object. |
| Walk To NPC | `WALK_TO_NPC` | Walks to (optional staging tile) then interacts with an NPC. |

### Combat / consumables / prayer / spec
| Task | File `type` | What it does |
|---|---|---|
| Eat Food | `EAT_FOOD` | Eats the first present food from a comma-separated list. |
| Drink Potion | `DRINK_POTION` | Drinks the first present potion (any dose) from a comma list. |
| Activate Prayer | `ACTIVATE_PRAYER` | Activates a prayer by name. |
| Deactivate Prayer | `DEACTIVATE_PRAYER` | Deactivates a prayer by name. |
| Deactivate All Prayers | `DEACTIVATE_ALL_PRAYERS` | Turns off all active prayers. |
| Toggle Quick Prayer | `TOGGLE_QUICK_PRAYER` | Enables/disables quick prayers (use NOT to force off). |
| Special Attack | `SPECIAL_ATTACK` | Uses the spec orb if current spec ≥ **Min spec %** (default 50). |

### Flat conditional checks (note: these do NOT gate the tasks after them)
| Task | File `type` | What it does |
|---|---|---|
| If Has Item | `IF_HAS_ITEM` | A leftover simple check that does **not** gate following tasks. Use a 📋 Condition block instead. |
| If Bank Open | `IF_BANK_OPEN` | Same — does not gate. Use 📋 Condition: Bank Open. |
| If Animating | `IF_ANIMATING` | Same — does not gate. Use 📋 Condition: Animating. |

> **Important:** the flat **If …** tasks are inert as gates — they don't control whether the tasks below them
> run. To make steps conditional, either drop them inside a **📋 Condition block** or attach an **inline
> condition** to the task itself (§3). Always prefer those.

### Condition BLOCKS (`📋`) — wrap child tasks; children run only when the condition is true
You can invert any of these with the **NOT** option.

**Boolean blocks:**
| Block (dropdown name) | File `type` | Runs children when … |
|---|---|---|
| 📋 Condition: Has Item | `CONDITION_HAS_ITEM` | inventory HAS the item |
| 📋 Condition: No Item | `CONDITION_NO_ITEM` | inventory does NOT have the item |
| 📋 Condition: Bank Open | `CONDITION_BANK_OPEN` | the bank is open |
| 📋 Condition: Bank Closed | `CONDITION_BANK_CLOSED` | the bank is closed |
| 📋 Condition: Animating | `CONDITION_ANIMATING` | the player is animating |
| 📋 Condition: Idle | `CONDITION_IDLE` | the player is idle |
| 📋 Condition: Inv Full | `CONDITION_INV_FULL` | inventory is full |
| 📋 Condition: Inv Empty | `CONDITION_INV_EMPTY` | inventory is empty |
| 📋 Condition: Inv Count | `CONDITION_INV_CONTAINS` | inventory has X+ of the item (Quantity) |
| 📋 Condition: Menu Open | `CONDITION_MENU_OPEN` | the make/skill menu is open |
| 📋 Condition: NPC Exists | `CONDITION_NPC_EXISTS` | an NPC by name/ID exists (optional max distance) |
| 📋 Condition: Object Exists | `CONDITION_OBJECT_EXISTS` | an object by name/ID exists (optional max distance) |
| 📋 Condition: In Region | `CONDITION_IN_REGION` | the player is in a region id (Quantity) |
| 📋 Condition: In Combat | `CONDITION_IN_COMBAT` | the player is in combat / interacting |
| 📋 Condition: Bank Contains | `CONDITION_BANK_CONTAINS` | the bank has X+ of the item |
| 📋 Condition: Item Equipped | `CONDITION_ITEM_EQUIPPED` | the item(s) (comma list) are equipped |
| 📋 Condition: Ground Item | `CONDITION_GROUND_ITEM_EXISTS` | a ground item exists (optional max distance) |

**Numeric blocks** (you pick a **Comparator** `<` `<=` `>` `>=` `==` and a **Threshold/Value**; a blank
comparator defaults to `>=`):
| Block | File `type` | Compares |
|---|---|---|
| 📋 Condition: HP | `CONDITION_HP` | HP vs threshold (percent or absolute) |
| 📋 Condition: Prayer | `CONDITION_PRAYER` | prayer points vs threshold (percent or absolute) |
| 📋 Condition: Run Energy | `CONDITION_RUN_ENERGY` | run energy vs threshold |
| 📋 Condition: Spec Energy | `CONDITION_SPEC` | special-attack % vs threshold |
| 📋 Condition: Skill Level | `CONDITION_SKILL_LEVEL` | a skill level (boosted or real) vs threshold |
| 📋 Condition: At Location | `CONDITION_AT_LOCATION` | within radius (Quantity) of a tile (X,Y,Z) |
| 📋 Condition: Var Compare | `CONDITION_VAR_COMPARE` | a variable vs a value (see §4) |

### Variables / control
| Task | File `type` | What it does |
|---|---|---|
| Stop Script | `STOP_SCRIPT` | Stops the whole script immediately. |
| Set Var | `SET_VAR` | Sets a variable to a value (value = Quantity). |
| Inc Var | `INC_VAR` | Increases a variable (step = Quantity; defaults to +1). |
| Dec Var | `DEC_VAR` | Decreases a variable (step = Quantity; defaults to -1). |

### Reactive PvM prayer triggers (see §5)
| Task | File `type` | What it does |
|---|---|---|
| Prayer Trigger: Chat | `PRAYER_TRIGGER_CHAT` | Reactive overhead prayer driven by a chat phrase. |
| Prayer Trigger: Projectile | `PRAYER_TRIGGER_PROJECTILE` | Reactive overhead prayer pre-prayed from an incoming projectile id. |
| Prayer Trigger: Animation | `PRAYER_TRIGGER_ANIMATION` | Reactive overhead prayer from an NPC attack animation id. |

### Click type
A **Click type** dropdown on most interaction tasks offers **LEGIT** / **BYPASS** / **HYBRID** (default
LEGIT). In an exported file this is the `clickType` number: `0` = LEGIT, `1` = BYPASS, `2` = HYBRID. Leave it on
LEGIT unless you specifically need otherwise.

### Idle-retry watchdog ("Expect animation (retry if idle)")
Several action tasks (Eat Food, Drink Potion, Special Attack, Attack NPC, Click Object, Click NPC, Use Item on
Object, Use Item on Item, Pickup Ground Item, Inventory Click, Walk To Object, Walk To NPC) offer an
**"Expect animation (retry if idle)"** checkbox. Turn it on and you get extra fields: **Idle window (ms)**,
**Max retries** (default 3), **Timeout (ticks)** (default 50), and **Expected anim id** (-1 = any animation).
When enabled, after clicking, the task watches for progress (an animation start / the expected animation /
becoming engaged). If nothing happens within the window, it re-clicks — and it re-checks the condition first,
so consuming actions (eat / drink / spec / withdraw) never over-fire. If it still can't make progress it gives
up and the script moves on (it never hangs forever). This is handy for "make" steps where the player can idle
between actions.

---

## 3. Inline conditions

Any task can carry one or more **inline conditions** (the "Conditions (all must be true)" editor in the task
dialog). **All** of them must be true for that task to run on a given pass — that's how you gate a single task
without wrapping it in a block.

The condition types you can choose for an inline condition are the same set as the **📋 Condition** blocks in
§2. Per inline condition you can set: the type, the item/NPC/object name, a max distance (for the "exists"
checks), a comparator + value (for numeric checks), a skill name + Boosted toggle (for Skill Level), a variable
name (for Var Compare), and a **NOT** toggle to invert it.

A common pattern, "only withdraw if I don't already have Vial of water":

```
Task: Withdraw Item — Vial of water (14)
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: Vial of water
```

In an exported file, inline conditions live in a `conditions` array on the task (see §6).

---

## 4. Variables and the repetition idiom

Variables are how you count things and do conditional/branching logic.

- A variable is a **named number**. A variable you've never set reads as **0**, so you can compare it before
  ever setting it.
- All variables **reset to 0 each time you press Start**, and **persist across loop cycles** during that run.

### The four building blocks
- **Set Var** — sets `var = Quantity`.
- **Inc Var** — adds Quantity to the variable (defaults to **+1** if Quantity is 0/blank).
- **Dec Var** — subtracts Quantity from the variable (defaults to **-1**).
- **📋 Condition: Var Compare** — a block (or an inline condition) that runs/passes only when
  `variable [comparator] value` is true.

All four are **instant** (they don't end the pass) and run **at most once per cycle**. With **Loop ON**, an
**Inc Var** placed in the body counts **once per cycle**.

### "Do N times, then stop" recipe
With **Loop ON**:
1. Do **not** put a `Set Var count = 0` in the loop body — it would re-zero every cycle. Rely on the
   start-of-run reset (every variable starts at 0).
2. Your work tasks…
3. An **Inc Var** `count` (by 1) after the work.
4. A **📋 Condition: Var Compare** block (`count >= N`) whose only child is a **Stop Script** task. When the
   count reaches N, the block passes and the script stops.

### Naming
Variable names are short free text: `count`, `trips`, `potionsMade`, `phase`. Keep them descriptive.

---

## 5. Reactive PvM prayer

Reactive prayer is **event-driven** and **pre-empts the task scan** — it flips your overhead protection prayer
the instant an attack telegraphs, instead of waiting for a task's turn. You set it up with the three **Prayer
Trigger** tasks. The triggers are activated when you press **Start**, and re-activated when you edit/reload the
script, so changes take effect on the next Start.

Each Prayer Trigger task has a **Trigger prayer** dropdown limited to **Protect from Melee**, **Protect from
Magic**, and **Protect from Missiles**, plus a **Priority** (lower number wins if two triggers fire on the same
tick; default 1). A trigger with a blank/unknown prayer (or, for Chat, a blank phrase) is simply ignored.

### Prayer Trigger: Chat
- Field: **Trigger phrase** — matched as a **case-insensitive substring** of an incoming chat line.
- A chat match **HOLDS** the chosen overhead prayer until the next chat trigger replaces it. Keep the phrase to
  the distinguishing fragment so trailing text (e.g. "...your prayers have been sapped.") still matches.
- This is exactly what the **Olm Basics** preset uses.

### Prayer Trigger: Projectile
- Fields: **Projectile id** + **Lead ticks** (pre-pray lead; default 1 = prayer active on the impact tick).
- Pre-prays the overhead based on an incoming projectile of that id, timed off the projectile's flight. Best
  for fast, telegraphed projectile specials. You must capture the projectile id live in-client first.

### Prayer Trigger: Animation
- Fields: **NPC anim id** + **Anim→hit ticks** (how many ticks after the animation the hit lands; default 0).
- Schedules the prayer that many ticks after a matching NPC attack animation. For melee/contact attacks with
  no projectile and no chat line. Capture the animation id live in-client first.

### The Olm Basics preset
**`Presets ▾ → Boss → Olm Basics`** inserts a Script (group) named **"Olm Basics"** containing **three editable
Prayer Trigger: Chat tasks** — there is no "Olm" task type or checkbox; it is just three generic chat triggers:

| Task name         | Trigger phrase                    | Trigger prayer        |
|-------------------|-----------------------------------|-----------------------|
| Olm: Melee sphere | `sphere of aggression`            | Protect from Melee    |
| Olm: Magic sphere | `sphere of magical power`         | Protect from Magic    |
| Olm: Range sphere | `sphere of accuracy and dexterity`| Protect from Missiles |

Open the group to view or edit the three tasks. Insert it, press Start, and fight Olm. You can extend it by
adding more Prayer Trigger tasks (Chat for any boss that announces its attack in chat, or Projectile/Animation
for attacks with no chat line).

---

## 6. The exported script file format

The **Export** button writes the selected Script to a `.json` file, and **Import** reads one back in. This is
the user-visible file format — editing one and importing it is equivalent to building it in the panel. Agents
authoring scripts should write this exact shape.

Every node — a Script, a 📋 Condition block, or a plain task — is the **same object shape**. Defaults are shown;
unused keys may simply be omitted (anything you leave out falls back to these defaults).

```
id                    (8-character id; a fresh one is generated on import)
name                  the task/script name shown on the card
type                  the file type value from §2 (e.g. "OPEN_BANK"); missing/unknown -> the entry is skipped
enabled               true
priority              0          (not used for ordering — order = position in the list)
itemName1             ""         the primary item / NPC / object / food list / etc. (per task)
itemName2             ""         the "Item 2 / Action" field
quantity              1          quantity / min-count / region id / distance / var value / timeout ticks (per task)
delay                 0          delay value (ms, or ticks if delayInTicks)
delayInTicks          false
randomizePercent      0          ± randomization on the delay
spellName             ""         spell name (Cast Spell / Teleport)
customAction          ""         "Menu Option" / custom command / teleport verb / interact action
invertCondition       false      the "NOT" toggle for a block / flat-If / numeric block
walkX, walkY, walkZ   0, 0, 0    tile coords / staging tile
graceMs               2          animation grace; with graceInTicks=true this is 2 ticks
graceInTicks          true
graceRandomizePercent 0
isGroup               true for a Script and for any 📋 Condition block; false for a plain task
childScripts          children of a group / condition block (null for a plain task)
conditions            inline conditions (an array; ALL must be true to run); null/absent = none
op                    ""         block comparator symbol ("<","<=",">",">=","=="); "" means ">="
threshold             0          block numeric threshold (also the At Location radius)
thresholdIsPercent    false      HP / Prayer block: percent vs absolute
skillName             ""         Skill Level block (e.g. "MINING")
useBoostedLevel       true       Skill Level block: boosted vs real level
clickType             0          0 = LEGIT, 1 = BYPASS, 2 = HYBRID
minSpecPercent        50         Special Attack minimum spec %
varName               ""         Set/Inc/Dec Var + Var Compare
expectAnimation       false      idle-retry watchdog on/off
retryIfIdleMs         0          watchdog idle window (ms); 0 = off
maxRetries            0          watchdog retries; 0 = default (3)
timeoutTicks          0          watchdog hard ceiling in ticks; 0 = default (50)
expectedAnimationId   -1         watchdog: require this exact animation id; -1 = any
triggerMatchId        0          Prayer Trigger: Projectile id / Animation npc anim id (Chat ignores it)
triggerPrayer         ""         overhead prayer name, e.g. "PROTECT_FROM_MAGIC"
triggerPhrase         ""         Prayer Trigger: Chat phrase (case-insensitive substring)
triggerHitDelayTicks  0          Animation: anim->hit ticks; Chat: 0 (hold); Projectile: lead (default 1)
triggerPriority       1          which trigger wins on the same tick (lower wins)
```

Inline-condition objects (entries in a task's `conditions` array) use this shape:

```
type                  one of the CONDITION_* file types from §2
itemName              ""         item / NPC / object name (per condition)
quantity              1          count / region / distance / numeric compare value (per condition)
invert                false      the "NOT" toggle
op                    ""         comparator symbol; "" means ">="
thresholdIsPercent    false      HP / Prayer inline: percent vs absolute
skillName             ""         Skill Level inline
useBoostedLevel       true       Skill Level inline: boosted vs real
varName               ""         Var Compare inline
distance              0          max distance for NPC / Object / Ground-item exists checks (0 = any)
graceMs               2
graceInTicks          true
graceRandomizePercent 0
```

### Structure rules (so a file imports and runs)
- The **top-level node is a Script**: `isGroup: true`, `type: "CUSTOM"`, with its tasks in `childScripts`.
- A **📋 Condition block** is also a group: `isGroup: true` with a `childScripts` array (its children run only
  when the block's condition passes). A numeric block also sets `op` + `threshold`.
- A **plain task** has `isGroup: false` and `childScripts: null`.
- For numeric **inline** conditions the compare value lives in the condition's `quantity`; for numeric
  **blocks** it lives in the block's `threshold`. A blank `op` means `>=`.
- Give every node a `type` and set `enabled: true`. Entries with a missing/unknown `type` are skipped on
  import, so an old or hand-edited file with an unrecognized `type` just drops that one entry cleanly.

---

## 7. Fully-worked example scripts

These are valid files you can save and **Import** (or paste into the Import dialog) and run. Remember to set the
**Loop** toggle as noted.

### 7.1 Herblore bankstander (Loop ON — runs forever)
Make unfinished potions: open bank → withdraw vials + herb → combine → repeat. The conditions make each step
**wait** for the right state; Loop ON re-runs the list. (Turn the **Loop** toggle ON.)

```json
{
  "id": "herb0001",
  "name": "Herblore - Unf Potions",
  "type": "CUSTOM",
  "enabled": true,
  "isGroup": true,
  "childScripts": [
    {
      "id": "herb0010", "name": "Open Bank", "type": "OPEN_BANK", "enabled": true,
      "isGroup": false, "childScripts": null,
      "delay": 2, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_CLOSED", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Guam leaf", "invert": false }
      ]
    },
    {
      "id": "herb0020", "name": "Deposit All", "type": "DEPOSIT_ALL", "enabled": true,
      "isGroup": false, "childScripts": null,
      "delay": 1, "delayInTicks": true,
      "conditions": [ { "type": "CONDITION_NO_ITEM", "itemName": "Guam leaf", "invert": false } ]
    },
    {
      "id": "herb0030", "name": "Withdraw Vials", "type": "WITHDRAW_ITEM", "enabled": true,
      "itemName1": "Vial of water", "quantity": 14,
      "isGroup": false, "childScripts": null,
      "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Vial of water", "invert": false }
      ]
    },
    {
      "id": "herb0040", "name": "Withdraw Herb", "type": "WITHDRAW_ITEM", "enabled": true,
      "itemName1": "Guam leaf", "quantity": 14,
      "isGroup": false, "childScripts": null,
      "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Guam leaf", "invert": false }
      ]
    },
    {
      "id": "herb0050", "name": "Close Bank", "type": "CLOSE_BANK", "enabled": true,
      "isGroup": false, "childScripts": null,
      "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_HAS_ITEM", "itemName": "Guam leaf", "invert": false },
        { "type": "CONDITION_HAS_ITEM", "itemName": "Vial of water", "invert": false }
      ]
    },
    {
      "id": "herb0060", "name": "Combine Vial + Herb", "type": "USE_ITEM_ON_ITEM", "enabled": true,
      "itemName1": "Vial of water", "itemName2": "Guam leaf",
      "isGroup": false, "childScripts": null,
      "delay": 1, "delayInTicks": true,
      "expectAnimation": true, "retryIfIdleMs": 1800,
      "conditions": [
        { "type": "CONDITION_BANK_CLOSED", "invert": false },
        { "type": "CONDITION_HAS_ITEM", "itemName": "Guam leaf", "invert": false },
        { "type": "CONDITION_MENU_OPEN", "invert": true },
        { "type": "CONDITION_IDLE", "invert": false }
      ]
    },
    {
      "id": "herb0070", "name": "Make-all Menu", "type": "SELECT_MENU_OPTION", "enabled": true,
      "customAction": "Make",
      "isGroup": false, "childScripts": null,
      "delay": 2, "delayInTicks": true,
      "conditions": [ { "type": "CONDITION_MENU_OPEN", "invert": false } ]
    },
    {
      "id": "herb0080", "name": "Wait Animation", "type": "WAIT_ANIMATION", "enabled": true,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true
    }
  ]
}
```

### 7.2 Olm Basics (reactive prayer group)
Three chat triggers that HOLD the matching overhead until the next sphere message — exactly what
`Presets ▾ → Boss → Olm Basics` inserts. The triggers activate on Start regardless of the Loop toggle.

```json
{
  "id": "olm00001",
  "name": "Olm Basics",
  "type": "CUSTOM",
  "enabled": true,
  "isGroup": true,
  "childScripts": [
    {
      "id": "olm00010", "name": "Olm: Melee sphere", "type": "PRAYER_TRIGGER_CHAT", "enabled": true,
      "isGroup": false, "childScripts": null,
      "triggerPhrase": "sphere of aggression", "triggerPrayer": "PROTECT_FROM_MELEE", "triggerPriority": 1
    },
    {
      "id": "olm00020", "name": "Olm: Magic sphere", "type": "PRAYER_TRIGGER_CHAT", "enabled": true,
      "isGroup": false, "childScripts": null,
      "triggerPhrase": "sphere of magical power", "triggerPrayer": "PROTECT_FROM_MAGIC", "triggerPriority": 1
    },
    {
      "id": "olm00030", "name": "Olm: Range sphere", "type": "PRAYER_TRIGGER_CHAT", "enabled": true,
      "isGroup": false, "childScripts": null,
      "triggerPhrase": "sphere of accuracy and dexterity", "triggerPrayer": "PROTECT_FROM_MISSILES",
      "triggerPriority": 1
    }
  ]
}
```

To cover a projectile special (an attack with no chat line), add a `PRAYER_TRIGGER_PROJECTILE` child **once you
have captured the real projectile id in-client** — put that id in `triggerMatchId`. Until you fill in a correct
id the trigger does nothing. The shape is:

```json
{ "type": "PRAYER_TRIGGER_PROJECTILE", "name": "Projectile special",
  "triggerMatchId": 0, "triggerPrayer": "PROTECT_FROM_MAGIC",
  "triggerHitDelayTicks": 1, "triggerPriority": 1, "isGroup": false, "childScripts": null }
```
(Replace `triggerMatchId: 0` with the projectile id you captured; `0` is a placeholder and will not fire.)

### 7.3 Counted grind with variables + Stop Script (Loop ON, stop after N)
Cut 200 inventories of gems, then stop. `count` starts at 0 (reset on Start); each cycle adds 1; when
`count >= 200` the Var Compare block runs Stop Script. **Turn Loop ON.**

```json
{
  "id": "grind001",
  "name": "Gem Cutting x200",
  "type": "CUSTOM",
  "enabled": true,
  "isGroup": true,
  "childScripts": [
    {
      "id": "grind010", "name": "Open Bank", "type": "OPEN_BANK", "enabled": true,
      "isGroup": false, "childScripts": null, "delay": 2, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_CLOSED", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Uncut sapphire", "invert": false }
      ]
    },
    {
      "id": "grind015", "name": "Deposit All", "type": "DEPOSIT_ALL", "enabled": true,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true,
      "conditions": [ { "type": "CONDITION_NO_ITEM", "itemName": "Uncut sapphire", "invert": false } ]
    },
    {
      "id": "grind020", "name": "Withdraw Chisel", "type": "WITHDRAW_ITEM", "enabled": true,
      "itemName1": "Chisel", "quantity": 1,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Chisel", "invert": false }
      ]
    },
    {
      "id": "grind025", "name": "Withdraw Gems", "type": "WITHDRAW_ITEM", "enabled": true,
      "itemName1": "Uncut sapphire", "quantity": 27,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_NO_ITEM", "itemName": "Uncut sapphire", "invert": false }
      ]
    },
    {
      "id": "grind030", "name": "Close Bank", "type": "CLOSE_BANK", "enabled": true,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true,
      "conditions": [
        { "type": "CONDITION_BANK_OPEN", "invert": false },
        { "type": "CONDITION_HAS_ITEM", "itemName": "Uncut sapphire", "invert": false }
      ]
    },
    {
      "id": "grind040", "name": "Chisel on Gem", "type": "USE_ITEM_ON_ITEM", "enabled": true,
      "itemName1": "Chisel", "itemName2": "Uncut sapphire",
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true,
      "expectAnimation": true, "retryIfIdleMs": 1800,
      "conditions": [ { "type": "CONDITION_HAS_ITEM", "itemName": "Uncut sapphire", "invert": false } ]
    },
    {
      "id": "grind045", "name": "Make-all", "type": "SELECT_MENU_OPTION", "enabled": true,
      "customAction": "Make",
      "isGroup": false, "childScripts": null, "delay": 2, "delayInTicks": true,
      "conditions": [ { "type": "CONDITION_MENU_OPEN", "invert": false } ]
    },
    {
      "id": "grind050", "name": "Wait Done", "type": "WAIT_IDLE", "enabled": true,
      "isGroup": false, "childScripts": null, "delay": 1, "delayInTicks": true
    },
    {
      "id": "grind060", "name": "Count Batch", "type": "INC_VAR", "enabled": true,
      "varName": "count", "quantity": 1,
      "isGroup": false, "childScripts": null
    },
    {
      "id": "grind070", "name": "Stop at 200?", "type": "CONDITION_VAR_COMPARE", "enabled": true,
      "isGroup": true,
      "varName": "count", "op": ">=", "threshold": 200,
      "childScripts": [
        { "id": "grind071", "name": "Stop", "type": "STOP_SCRIPT", "enabled": true,
          "isGroup": false, "childScripts": null }
      ]
    }
  ]
}
```

---

## 8. Guardrails — what NOT to do

- **There is no "jump to step" or "repeat this block" control.** For repetition use **Loop ON**; for "do N
  times then stop" or branching use **variables + 📋 Condition: Var Compare + Stop Script** (§4).
- **Don't expect a line counter.** There's no "go to step 3". Order only matters as scan order; control comes
  from conditions and the once-per-cycle re-scan. Design every task to be safe to *skip and retry*.
- **Don't put `Set Var x = 0` in a Loop-ON body** if it's meant to be an accumulator — it would re-zero each
  cycle. Rely on the start-of-run reset, or gate the Set behind a condition.
- **Loop OFF runs the list once, then idles.** It does not auto-stop — the run keeps the **STOP** button lit
  until you click it (or a **Stop Script** task fires). For anything continuous, turn **Loop ON**.
- **The flat If … tasks don't gate.** **If Has Item / If Bank Open / If Animating** do not control the tasks
  below them. Use a **📋 Condition block** or an **inline condition** instead.
- **A 📋 Condition block needs children.** A condition block with an empty child list does nothing useful — put
  the tasks you want gated inside it.
- **One real action per pass.** Stacking several blocking actions with no conditions just runs them
  one-per-pass top to bottom. If you need a specific order, gate them with conditions and let the re-scan
  sequence them.
- **Reactive prayer is separate from the scan.** Don't try to flip overheads for fast attacks with
  **Activate Prayer** tasks in the loop — they'll be too slow. Use the **Prayer Trigger** tasks.
- **Prayer Trigger prayer must be a real overhead** (Protect from Melee / Magic / Missiles). A blank or unknown
  prayer (or a blank Chat phrase) is silently ignored, and nothing will flip.
- **Projectile / Animation triggers need a real id.** They won't fire until you fill in the correct projectile
  or NPC-animation id — capture it live in-client first.
