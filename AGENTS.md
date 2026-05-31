# AGENTS.md — UG Task Builder

> **This is a pointer file.** The full guide lives in [`CLAUDE.md`](CLAUDE.md) in this same folder.
> `CLAUDE.md` and `AGENTS.md` cover the same material — read `CLAUDE.md` for the complete walkthrough.

If you are a player (or an AI agent building a script for a player) and you want to **create or edit a UG Task
Builder script**, start with [`CLAUDE.md`](CLAUDE.md). It walks you through everything, end to end.

## What UG Task Builder is

UG Task Builder is a no-code automation panel in the RuneLite client. You open the **UG Task Builder** side
panel, create a **Script**, and fill it with an ordered list of **Tasks** (each Task is a single action you
pick from a dropdown, like **Open Bank**, **Withdraw Item**, **Use Item on Item**, **Attack NPC**, or
**Cast Spell on Item**). Press **START** to run it and **STOP** to halt.

## What `CLAUDE.md` covers

1. **How a script runs** — the panel checks your tasks in order, top to bottom, and runs each one only when its
   conditions are met; tasks that aren't ready yet simply wait and get re-checked. There is no jump/goto step —
   to repeat a script, turn the **Loop** toggle ON (it's on by default), and use conditions, variables, and the
   **Stop Script** task to decide when to keep going or finish.

2. **Every task type** — all the actions and the 📋 **Condition** blocks shown in the task dropdown, the inline
   **Conditions (all must be true)** editor you can attach to any task, the comparators (`<`, `<=`, `>`, `>=`,
   `==`), the **Click type** option (LEGIT / BYPASS / HYBRID), and the **Expect animation (retry if idle)**
   option that re-tries an action if your character stalls.

3. **Variables** — the **Set Var**, **Inc Var**, and **Dec Var** tasks plus the **📋 Condition: Var Compare**
   block, used together with the **Loop** toggle and **Stop Script** to build counted or conditional repeats
   (for example, "stop after 50 cycles").

4. **Reactive prayer** — the three **Prayer Trigger** tasks (**Prayer Trigger: Chat**, **Prayer Trigger:
   Projectile**, and **Prayer Trigger: Animation**) that flip your overhead prayer automatically in reaction to
   a chat phrase, an incoming projectile, or an NPC's attack animation. The **Presets ▾ → Boss → Olm Basics**
   preset is a ready-made group of three editable **Prayer Trigger: Chat** tasks.

5. **The script file format** — what you get from the **Export** button and can load with **Import**: a `.json`
   file, with the exact field names that appear in an exported script and example scripts you can copy.

6. **Worked examples** — full sample scripts (a Herblore bankstander, the Olm Basics group, and a counted grind
   that stops after a set number of cycles using variables and **Stop Script**).

7. **Tips** — common mistakes to avoid and how to keep scripts reliable.

## Ready-made presets

Click **Presets ▾** in the panel to drop in a finished script. The menu has: **Herblore**, **Unf Potions**,
**Crafting**, **Gem Cutting**, **Fletching**, **Bow Stringing**, **Magic**, **Battlestaves**, **Glass Blowing**,
and **Boss** (Olm Basics). Step-by-step pages for these live under [`scripts/`](scripts/).
