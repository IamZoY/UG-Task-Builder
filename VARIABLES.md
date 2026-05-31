# Variables & Looping — Set Var / Inc Var / Dec Var / Condition: Var Compare

This guide explains the **variable system** in UG Task Builder and how to use it for repeating and branching.
UG Task Builder is a visual, no-code task-scripting plugin: you assemble a script out of tasks and conditions,
with no programming. Variables add a small amount of memory to that — just enough to **count** and to **stop
after N times** — without turning your script into a program.

> **There is no "jump to step" or "repeat block" control.** Everything you might want — repeating, counting,
> looping N times, stopping, branching — is done with the **Loop** toggle + **Set Var / Inc Var / Dec Var** +
> **Condition: Var Compare** + **Stop Script**, which this guide covers in full.

---

## Table of Contents

1. [How the script runs (why variables work the way they do)](#1-how-the-script-runs)
2. [What a variable is](#2-what-a-variable-is)
3. [The four variable building blocks](#3-the-four-variable-building-blocks)
4. [Once-per-cycle behavior](#4-once-per-cycle-behavior)
5. [How counting works with Loop ON](#5-how-counting-works-with-loop-on)
6. [The "do N times then stop" recipe](#6-the-do-n-times-then-stop-recipe)
7. [Repeating, looping, and branching with Loop + variables](#7-repeating-looping-and-branching-with-loop--variables)
8. [Worked examples](#8-worked-examples)
9. [Comparator symbols](#9-comparator-symbols)
10. [Naming, tips, and gotchas](#10-naming-tips-and-gotchas)
11. [Importing older scripts](#importing-older-scripts)

---

## 1. How the script runs

UG Task Builder runs your task list **top to bottom, over and over**. There is **no "jump to" and no "repeat
N" step** — instead, on each pass the script walks your list from the top and runs *every* task whose
conditions are currently true. A task that isn't ready yet (its condition is false, or it's still waiting on
the game) is simply **skipped and tried again** next pass — it waits in place. One real in-game action happens
per pass, then the list starts over from the top.

The **Loop** toggle (on the toolbar, **ON** by default) controls what happens at the bottom of the list:

- **Loop ON** — at the end of each pass the whole list is re-armed so every task can fire again, and the list
  repeats forever. This repeating *is* your loop.
- **Loop OFF** — the list runs through once and then idles in place (no auto-stop): each task has fired once
  and stays done, and the run keeps the **STOP** button lit until you click it or a **Stop Script** task fires.

Variables fit neatly into this. Reading or comparing a variable is instant and never makes the script wait, so
a variable check behaves like any other condition: it decides whether a task runs this pass, and it's
re-checked every pass.

---

## 2. What a variable is

A variable is a named **whole number** that the script remembers while it's running.

- **Whole numbers only.** Every variable is an integer. There are no text, decimal, or true/false values — use
  `0` and `1` if you want a yes/no flag.
- **An unset variable counts as `0`.** If you compare or read a variable you never set, it's `0`. You don't
  have to create a variable before using it — the first **Inc Var** on a brand-new `count` makes it `1` (up
  from `0`).
- **Cleared every time you press Start.** When you press **Start**, all variables reset to `0`, so every
  counter begins fresh. You don't need a "Set Var x = 0" step at the top — Start already does that for you.
- **Kept across cycles within one run.** While the script is running, variables are **not** wiped between Loop
  cycles. This is what makes counting possible: a counter you raise each cycle keeps growing until the run
  stops.
- **Shared across the whole run.** Every task in the running script sees the same variables. A variable named
  `count` is the same `count` everywhere. Pick different names if you need separate counters.

```
Press Start ──► all variables reset to 0
   │
   ├─ cycle 1 ─► tasks read / change variables ─┐
   ├─ cycle 2 ─► values carry over  ◄───────────┘
   ├─ cycle 3 ─► values carry over
   │   ...
   └─ Stop Script (or you press STOP) ──► run ends
Press Start again ──► variables reset to 0 again
```

Variables are just temporary bookkeeping for one run. They aren't saved to disk and they don't change anything
in the game itself.

---

## 3. The four variable building blocks

There are three **action** tasks that change a variable, and one **condition** that reads one. You add them
exactly like any other task: press **+** (Add), pick the **Type** from the dropdown, and fill in the fields.

### Set Var

Sets a variable to an exact number.

```
Type:   Set Var
Fields: Variable name:   the variable to write, e.g. "count"
        Value:           the number to store
Result: count = Value
```

A **Set Var** with an empty **Variable name** does nothing.

### Inc Var

Adds to a variable (counts up).

```
Type:   Inc Var
Fields: Variable name:   the variable to raise
        By:              how much to add (leave 0 to add 1)
Result: count = count + By   (or + 1 if By is 0)
```

A brand-new variable counts up from `0`, so the first **Inc Var** on `count` makes `count = 1`.

### Dec Var

Subtracts from a variable (counts down).

```
Type:   Dec Var
Fields: Variable name:   the variable to lower
        By:              how much to subtract (leave 0 to subtract 1)
Result: count = count - By   (or - 1 if By is 0)
```

> The three variable actions are **instant** — they change the number and immediately let the script carry on.
> They don't use up the "one action per pass" budget, so they never slow your script down or stall it.

### Condition: Var Compare

Reads a variable and compares it to a number. It comes in two forms.

**As a block** — pick **📋 Condition: Var Compare** as a task Type. It becomes a container; any tasks you put
inside it only run when the comparison is true:

```
Type:    📋 Condition: Var Compare   (a block — holds tasks inside)
Fields:  Variable name:   e.g. "count"
         Comparator:      one of  <  <=  >  >=  ==
         Value:           the number to compare against
Result:  if  count (comparator) Value  → run the tasks inside this block
```

**As an inline condition** — open any task's **Conditions (all must be true)** editor, add a row, and choose
**Var Compare**. That single task then only runs when the comparison is true:

```
Inline condition: Var Compare
Fields:  Variable name:   e.g. "count"
         Comparator:      one of  <  <=  >  >=  ==
         Value:           the number to compare against
Result:  the task runs only if  count (comparator) Value
```

A blank **Variable name** makes the comparison count as **not met**. Since an unset variable reads as `0`, a
test like `count >= 5` is false until you've raised `count` to 5.

---

## 4. Once-per-cycle behavior

This is the single most important rule.

Once a task's conditions are true and it runs, it won't run again until the next cycle re-arms it. This
"run once, then wait to be re-armed" behavior applies to the variable tasks exactly like every other task:

- **Set Var**, **Inc Var**, and **Dec Var** each fire **at most once per cycle.**
- With **Loop OFF**, the list runs through once, so each variable task fires **exactly once**, total.
- With **Loop ON**, every task is re-armed at the end of each cycle, so each variable task fires **once per
  cycle.**

That last point is what turns **Inc Var** into a **counter**:

> An **Inc Var** task (by 1) in a Loop-ON script raises its variable **once every cycle**. After 10 cycles it's
> `10`. In other words, it counts how many times the loop has run.

Because the variable tasks are instant and fire once per cycle, they don't get in the way of the rest of the
script — they tick the number forward once, then step aside.

---

## 5. How counting works with Loop ON

Put the counting step **inside the loop**, after the work it should count:

```
[Loop: ON]
├─ ... do one unit of work (e.g. make a potion) ...
└─ Inc Var  craftCount  by 1        ← fires once per cycle → counts cycles
```

Each cycle the loop:

1. runs the work tasks,
2. fires **Inc Var craftCount by 1** once (it gets re-armed each cycle),
3. starts over from the top.

So `craftCount` equals the number of completed cycles. You then **gate** something on that count using
**Condition: Var Compare**.

> **Don't put a "Set Var count = 0" inside a Loop-ON list** expecting it to be a one-time "start at zero" step.
> With Loop ON it re-runs every cycle and **re-zeros your counter**, so the count never climbs. You don't need
> it anyway — Start already zeroes everything. If you must reset a counter mid-run, put the **Set Var** inside
> a **📋 Condition: Var Compare** block so it only fires when you actually want it (see
> [example 4](#example-4--track-a-flag--do-something-once)).

---

## 6. The "do N times then stop" recipe

This is the standard pattern that replaces "repeat N times":

```
[Loop: ON]
├─ ... work tasks (the thing you want to do N times) ...
├─ Inc Var  n  by 1                                  ← count one completed pass
└─ 📋 Condition: Var Compare  n >= N
   └─ Stop Script                                    ← ends the script when the count is reached
```

How it plays out for `N = 100`:

```
cycle 1 : work → Inc n (n=1)    → is 1 >= 100?  no  → loop
cycle 2 : work → Inc n (n=2)    → is 2 >= 100?  no  → loop
   ...
cycle 100: work → Inc n (n=100) → is 100 >= 100? YES → Stop Script → run ends
```

Notes:

- **Don't add a "Set Var n = 0" at the top of the loop** — Start already zeroes everything, and a **Set Var**
  inside a Loop-ON list would reset the counter every cycle (see §5).
- **Stop Script** ends the whole run immediately. Variables reset the next time you press Start.
- Put **Inc Var** **after** the work, so you count completed passes, not attempts.

---

## 7. Repeating, looping, and branching with Loop + variables

Everything you might reach for a "jump" or "repeat" step to do is built from the **Loop** toggle, conditions,
and variables:

| What you want to do | Do this |
|---------------------|---------|
| **Repeat a block N times** | **Loop ON** + **Inc Var n** + **📋 Condition: Var Compare n >= N** → **Stop Script** (the [recipe above](#6-the-do-n-times-then-stop-recipe)) |
| **Loop forever** | **Loop ON** *is* the loop. Gate work with conditions; the list already returns to the top every cycle. |
| **Stop when a goal is met** | **📋 Condition: Var Compare** (or any condition) → **Stop Script** |
| **Conditionally skip / branch tasks** | attach an **inline condition** to the task(s), or wrap them in a **📋 Condition** block |

The mental shift:

- **There is no "jump."** The list runs top-to-bottom every pass. "Looping back" is automatic with Loop ON.
  "Skipping" is just a task whose condition is false this pass (it waits and tries again).
- **State lives in variables, not in a position in the list.** Instead of "jump back 5 tasks," you keep a
  number and let conditions decide what runs.
- **"Stop" is an explicit task** (**Stop Script**), usually inside a **📋 Condition: Var Compare** block.

---

## 8. Worked examples

### Example 1 — Make exactly 100 potions, then stop

Assume the "make one potion" work is already built (withdraw, combine, etc.). Add a counter and a stop:

```
Script: "100 potions" — [Loop: ON]
├─ Open Bank
├─ Withdraw Item  "Vial of water"
├─ Withdraw Item  "Clean ranarr"            (your real ingredients)
├─ Use Item on Item  "Clean ranarr" → "Vial of water"
├─ Wait Animation Cycle                      (wait for the make to finish)
├─ Inc Var  potions  by 1                    ← one potion done this cycle
└─ 📋 Condition: Var Compare  potions >= 100
   └─ Stop Script
```

- After the 100th cycle, `potions` is `100`, the comparison passes, **Stop Script** fires, and the run ends.
- No "Set Var potions = 0" is needed — Start already zeroed it.
- If your "make" actually produces a batch per cycle (e.g. 14 unfinished potions at once), count by that batch
  size instead (**Inc Var potions by 14**) and compare `potions >= 100`.

### Example 2 — Kill N monsters, then stop

Here "one kill per cycle" is up to you. A simple version raises the counter once per Loop cycle and assumes one
kill per cycle:

```
Script: "Kill 50" — [Loop: ON]
├─ Attack NPC  "Goblin"
├─ Wait Idle                                  (wait until the fight is over / you're idle)
├─ Inc Var  kills  by 1
└─ 📋 Condition: Var Compare  kills >= 50
   └─ Stop Script
```

If you want a kill to count only when something specific happens (e.g. the target is gone), attach an **inline
condition** to the **Inc Var** task so it only counts on a real kill:

```
├─ Inc Var  kills  by 1
│   └─ inline condition: 📋 Condition: NPC Exists  "Goblin"  (NOT)   (no goblin left = it died)
```

(Use whichever condition reliably marks "a kill happened" for your target.)

### Example 3 — Bank only every 5th cycle

Use the counter with a comparison to do something periodically. Count every cycle, and run a block only when
the count reaches the limit — then reset it:

```
Script: "bank every 5" — [Loop: ON]
├─ ... gather work each cycle ...
├─ Inc Var  sinceBank  by 1
└─ 📋 Condition: Var Compare  sinceBank >= 5
   ├─ Open Bank
   ├─ Deposit All Except  "Pickaxe"
   └─ Set Var  sinceBank = 0                  ← reset, only runs inside this block
```

The **Set Var sinceBank = 0** is **safe here** because it's inside the `>= 5` block — it only fires on the
banking cycle, not every cycle. This is the correct way to reset a counter mid-run.

### Example 4 — Track a flag / do something once

A variable also works as a one-time flag. `0` = not done, `1` = done:

```
Script — [Loop: ON]
└─ 📋 Condition: Var Compare  setupDone == 0       (runs only while the flag is 0)
   ├─ ... one-time setup tasks (e.g. equip gear) ...
   └─ Set Var  setupDone = 1                       ← flip the flag so the block never runs again
```

After the first cycle, `setupDone` is `1`, so the setup block is skipped for the rest of the run. (On the next
**Start**, variables reset and setup runs again.)

### Example 5 — Count down instead of up

**Dec Var** lets you count down from a target. Seed it once inside a gated **Set Var**, then count down and
stop at 0:

```
Script: "20 trips" — [Loop: ON]
├─ 📋 Condition: Var Compare  trips == 0           (seed once, only while still 0)
│   └─ Set Var  trips = 20                          (runs on cycle 1 only; after that trips != 0)
├─ ... do one trip of work ...
├─ Dec Var  trips  by 1
└─ 📋 Condition: Var Compare  trips <= 0
   └─ Stop Script
```

> Counting up (Example 1) is usually simpler because Start already zeroes everything and you don't need a seed
> step. Prefer **Inc Var** + `n >= N` → **Stop Script** unless you specifically want a visible "remaining"
> count.

---

## 9. Comparator symbols

**Condition: Var Compare** (and every other numeric condition) uses these comparators in the **Comparator**
dropdown:

| Symbol | Meaning            |
|--------|--------------------|
| `<`    | less than          |
| `<=`   | less than or equal |
| `>`    | greater than       |
| `>=`   | greater or equal   |
| `==`   | equal              |

**If no comparator is chosen, the default is `>=`** (greater-or-equal).

---

## 10. Naming, tips, and gotchas

- **Names are case-sensitive and trimmed.** `count` and `Count` are different variables; leading/trailing
  spaces are removed. Use short, consistent names (`count`, `kills`, `trips`, `sinceBank`).
- **No setup needed.** First use of a name creates it; names you never set read as `0`.
- **One shared set of variables.** Every task in the run shares the same variables. Use distinct names for
  separate counters.
- **Variable tasks fire once per cycle.** A single **Inc Var** fires once per cycle (Loop ON) or once total
  (Loop OFF). To count per-iteration, make one iteration = one cycle, or count by your batch size.
- **Don't re-zero a counter in the loop** (§5). Either rely on Start clearing variables, or put the **Set Var**
  inside a condition block (Examples 3 and 5).
- **Count after the work, not before**, so you count *completed* work.
- **Stop Script ends the run**, not just one cycle — there is no "stop one cycle." Put it inside the **Condition:
  Var Compare** block that detects your goal.
- **Variable tasks never make the script wait.** They don't slow your pacing, so adding counters is free.
- Variables are temporary bookkeeping for one run only — they aren't saved and aren't visible in the game.

---

## Importing older scripts

If you import an older script that contains a task type this version doesn't recognize, those steps are simply
**ignored** — they won't run and won't break the rest of the script. Rebuild that behavior with the patterns in
this guide:

- repeat N times → [the recipe in §6](#6-the-do-n-times-then-stop-recipe)
- loop forever → just turn **Loop ON**
- stop on a goal → **📋 Condition: Var Compare** → **Stop Script**
- branch / skip → inline conditions or **📋 Condition** blocks

(For Olm prayers, use the **Olm Basics** preset under **Presets ▾ → Boss** — see [`scripts/Olm.md`](scripts/Olm.md).)
