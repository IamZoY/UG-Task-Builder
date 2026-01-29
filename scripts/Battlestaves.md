# Battlestaff Scripts

Scripts for attaching orbs to battlestaves.

---

## Overview

These scripts automate attaching charged orbs to battlestaves to create elemental battlestaves.

**Requirements:**
- Be standing at a bank
- Have battlestaves in your bank
- Have charged orbs in your bank
- 54 Crafting (for all types)

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

### Task 3: Withdraw Battlestaves
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

### Task 4: Withdraw Orbs
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

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Battlestaff"
  └─ Has Item: [Orb]
```

---

### Task 6: Attach Orb to Staff
```
Type: Use Item on Item
Item 1: "Battlestaff"
Item 2: [Orb]
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Battlestaff"
  ├─ Has Item: [Orb]
  ├─ NOT Menu Open
  └─ Idle (Grace: 2t)
```

**Note:** The `Idle (Grace: 2t)` condition waits up to 2 ticks to confirm the player is truly idle before triggering.

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 25
```

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 8: Select Menu Option
```
Type: Select Menu Option
Option: 1
Delay: 1 tick
Conditions:
  └─ Menu Open
```

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
6. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
