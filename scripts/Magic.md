# Magic Scripts

Scripts for magic training at a bank (Plank Make, High Alchemy).

---

## Overview

These scripts automate casting magic spells on inventory items for training and profit. Each one banks for a
fresh load of items, closes the bank, then casts the spell on every item in your inventory — over and over
while the **Loop** toggle is ON (it is ON by default).

**Stops when you run out:** if the bank can no longer restock the item you're working on (you have none in your
inventory and none left in the bank), the script ends itself cleanly with a **Stop Script** task instead of
spinning forever. When it stops, the **START/STOP** button flips back to green **START**.

**Requirements:**
- Be standing at a bank
- Have the required runes equipped or in your inventory
- Have the target items in your bank
- Meet the required Magic level

---

## Loading a preset

Open the **UG Task Builder** side panel, click **Presets ▾**, open the **Magic** category, and pick one:

| Preset | Spell | Target | Level | XP/cast |
|--------|-------|--------|-------|---------|
| Plank Make (Oak) | Plank Make | Oak logs | 86 | 90 |
| Plank Make (Mahogany) | Plank Make | Mahogany logs | 86 | 90 |
| High Alch | High Level Alchemy | Rune platebody | 55 | 65 |

Picking an item drops a ready-to-run Script group into the side panel. Open the group to view/edit the tasks,
then press **START**. Leave **Loop** ON so it keeps re-banking automatically; turn **Loop** OFF if you want it
to run a single inventory and stop.

> **Note on High Alch:** this preset is wired to alch **Rune platebody**. To alch a different item, open the
> Script and change the item name on the **Withdraw** task and on the **Cast Spell on Item** tasks to match.

---

## Plank Make Script

### Requirements
- 86 Magic
- Runes: 1 Nature, 2 Astral, 15 Earth (per cast)
- Coins for plank cost (70 gp oak, 1050 gp mahogany)
- Lunar spellbook active

### Script Breakdown

#### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Logs]
```

#### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Logs]
```

#### Task 3: Stop (out of logs)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Logs]
  └─ Bank Contains [Logs] < 1
```
*Ends the script cleanly the moment you've used up your logs and the bank can't restock — no infinite re-banking.*

#### Task 4: Withdraw Logs
```
Type: Withdraw Item
Item: [Log Name]
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Logs]
```

#### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  └─ Has Item: [Logs]
```

#### Tasks 6-32: Cast Plank Make (x27)
```
Type: Cast Spell on Item
Spell: "Plank Make"
Item: [Logs]
Delay: 3 ticks
Conditions:
  ├─ Bank Closed
  └─ Has Item: [Logs]
```
*One Cast Spell on Item task per inventory slot — 27 in total. Each one only fires while the bank is closed and
you still have logs left, so they cast straight down the inventory and then the list loops back to the bank.*

---

## High Alchemy Script

### Requirements
- 55 Magic
- Runes: 1 Nature, 5 Fire (per cast)
- Standard or Ancient spellbook

### Script Breakdown

#### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: Rune platebody
```

#### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: Rune platebody
```

#### Task 3: Stop (out of Rune platebody)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: Rune platebody
  └─ Bank Contains Rune platebody < 1
```
*Ends the script cleanly once you've alched your last item and the bank can't restock it.*

#### Task 4: Withdraw Items
```
Type: Withdraw Item
Item: Rune platebody
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: Rune platebody
```

#### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  └─ Has Item: Rune platebody
```

#### Tasks 6-32: Cast High Alchemy (x27)
```
Type: Cast Spell on Item
Spell: "High Level Alchemy"
Item: Rune platebody
Delay: 5 ticks
Conditions:
  ├─ Bank Closed
  └─ Has Item: Rune platebody
```
*One Cast Spell on Item task per inventory slot — 27 in total. Change the item name here (and on the Withdraw
task) if you want to alch something other than Rune platebody.*

---

## XP Rates (Approximate)

| Method | XP/Cast | Casts/Hour | XP/Hour |
|--------|---------|------------|---------|
| Plank Make | 90 | ~1,800 | ~162,000 |
| High Alch | 65 | ~1,200 | ~78,000 |

---

## Profit Information

### Plank Make
| Plank | Cost | Sell Price | Profit |
|-------|------|------------|--------|
| Oak | Log + 70gp + runes | ~500 gp | ~200-300 gp |
| Mahogany | Log + 1050gp + runes | ~2,500 gp | ~400-800 gp |

### High Alchemy
Common profitable items:
- Rune items (platebodies, 2h swords)
- Battlestaves (all types)
- Dragonhide bodies
- Onyx bolts (e)

*Always check alch values vs buy price!*

---

## Tips

### Plank Make
1. **Bring coins** - Need coins for plank conversion
2. **Use Lunar Isle bank** - Close to altar if you need to swap books
3. **Rune pouch** - Saves inventory space
4. **Staff of earth** - Eliminates earth rune cost

### High Alchemy
1. **Explorer's ring** - Free alchs per day
2. **Nature rune cost** - Factor into profit calculations
3. **Fire staff** - Eliminates fire rune cost
4. **Buy limits** - Can only buy 70 of most items per 4 hours
5. **Alch value lookup** - Use wiki or GE to find profitable items

### Casting pace
The delay on each cast task (3 ticks for Plank Make, 5 ticks for High Alch) is what spaces the casts out so
they don't fire on top of each other. If casts ever feel rushed or get skipped, open the cast tasks and bump
their **Delay** up by a tick.
