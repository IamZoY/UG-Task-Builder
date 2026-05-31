# Battlestaff Scripts

Scripts for attaching orbs to battlestaves.

---

## Overview

These scripts automate attaching charged orbs to battlestaves to create elemental battlestaves.

**How to add one:** click `Presets ▾` on the Task Builder toolbar, open the **Battlestaves** category, and pick **Air**, **Water**, **Earth**, or **Fire**. The preset is added to the side panel as a group (named e.g. "Air Battlestaves"). Open the group to see its tasks. Leave the **Loop** toggle ON (it is on by default) so the list repeats and keeps refilling from the bank, then press **START**. The button turns red and reads **STOP** while running; click it to stop (the preset also stops itself when you run out of materials — see below).

**Requirements:**
- Be standing at a bank
- Have battlestaves in your bank
- Have charged orbs in your bank
- The Crafting level for the orb type you want (Water 54, Earth 58, Fire 62, Air 66)
- **Loop** toggle ON (so the list repeats and keeps refilling from the bank)

**Stops automatically when you run out:** if a refill is needed but the bank has no more battlestaves (or no more of the orb), the script ends cleanly on its own instead of looping forever. You'll see it stop after the last batch is banked.

---

## Available Presets

| Preset | Orb | Result | Level | XP |
|--------|-----|--------|-------|-----|
| Air | Air orb | Air battlestaff | 66 | 137.5 |
| Water | Water orb | Water battlestaff | 54 | 100 |
| Earth | Earth orb | Earth battlestaff | 58 | 112.5 |
| Fire | Fire orb | Fire battlestaff | 62 | 125 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: "Battlestaff"
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: "Battlestaff"
```

---

### Task 3: Stop (out of Battlestaff)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: "Battlestaff"
  └─ Bank Contains "Battlestaff" < 1
```

Ends the script cleanly when you've run out of battlestaves in the bank.

---

### Task 4: Stop (out of Orb)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Orb]
  └─ Bank Contains [Orb] < 1
```

Ends the script cleanly when the bank has no more of the orb.

---

### Task 5: Withdraw Battlestaves
```
Type: Withdraw Item
Item: "Battlestaff"
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Battlestaff"
```

---

### Task 6: Withdraw Orbs
```
Type: Withdraw Item
Item: [Orb Name]
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Battlestaff"
  └─ NOT Has Item: [Orb]
```

---

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Battlestaff"
  └─ Has Item: [Orb]
```

---

### Task 8: Attach Orb (Use Item on Item)
```
Type: Use Item on Item
Item 1: "Battlestaff"
Item 2: [Orb]
Delay: 1 tick
Expect animation (retry if idle): ON
  ├─ Idle window: 1800 ms
  └─ Max retries: 3
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Battlestaff"
  ├─ Has Item: [Orb]
  ├─ NOT Menu Open
  └─ Idle (Grace: 3t)
```

**Note:** The `Idle (Grace: 3t)` condition means "not animating for 3 ticks", so it won't re-click during the brief one-tick gaps between staves while a batch is being made.

**Idle-retry watchdog:** This step has **Expect animation (retry if idle)** turned on. After it clicks, it watches for the making animation to start. If you idle and nothing happens within the idle window (1800 ms), it re-clicks (up to 3 times), re-checking the conditions first so it never double-makes. This keeps the loop from stalling if a click is missed.

---

### Task 9: Wait Animation
```
Type: Wait Animation
Max Ticks: 25
```

Waits while the making animation plays (up to 25 ticks). Increase the max ticks if your batches take longer.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: "Make"
Delay: 1 tick
Conditions:
  └─ Menu Open
```

This clicks the **Make** button on the make/craft menu that pops up after using the orb on the staff.

---

## XP Rates (Approximate)

| Battlestaff | XP/Staff | Staffs/Hour | XP/Hour |
|-------------|----------|-------------|---------|
| Water | 100 | ~1,800 | ~180,000 |
| Earth | 112.5 | ~1,800 | ~202,500 |
| Fire | 125 | ~1,800 | ~225,000 |
| Air | 137.5 | ~1,800 | ~247,500 |

---

## Profit Information

Battlestaves are usually profitable to make:

| Staff | Materials Cost | Staff Value | Profit |
|-------|----------------|-------------|--------|
| Water | ~9,000 gp | ~9,200 gp | ~200 gp |
| Earth | ~9,000 gp | ~9,300 gp | ~300 gp |
| Fire | ~9,500 gp | ~9,800 gp | ~300 gp |
| Air | ~10,000 gp | ~10,500 gp | ~500 gp |

*Prices vary - check GE*

---

## Pro Tip: Daily Battlestaves

You can buy discounted battlestaves daily:
- **Zaff's Superior Staffs** (Varrock) - Up to 120/day with diary
- **Baba Yaga** (Lunar Isle) - Additional daily stock

Buying daily staves and attaching orbs is consistent profit!

---

## Tips

1. **Good XP + profit** - One of the few skills where you can do both
2. **14+14 inventory** - Equal amounts of staves and orbs
3. **Air = best XP** - But also most expensive orbs
4. **Water = budget option** - Lower XP but cheaper
5. **Alch the staves** - For even more profit (and magic XP)
6. **Animation Issues?** - If the script skips or stalls after clicking, raise the **Max Ticks** on the `Wait Animation` step, or nudge the `Idle` condition grace up by a tick
