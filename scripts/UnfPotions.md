# Unfinished Potion Scripts

Scripts for creating unfinished potions by combining herbs with vials of water.

---

## Overview

These scripts automate creating unfinished potions, which are the base for all herblore potions.

**To add one:** click `Presets ▾` on the Task Builder toolbar, open the **Unf Potions** category, and pick the herb you want (Guam (unf), Ranarr (unf), Toadflax (unf), or Snapdragon (unf)). The matching Script is added to the panel. Open it to see the tasks below; press `START` to run it. Leave the `Loop` toggle ON so it keeps re-banking and making potions automatically.

**Requirements:**
- Be standing at a bank
- Have clean herbs in your bank
- Have vials of water in your bank

**Runs out cleanly:** if the bank can't restock (you've run out of either the herb or Vials of water), the script stops itself instead of looping forever. See Tasks 3 and 4 below.

---

## Available Presets

| Preset | Herb | Result | Level |
|--------|------|--------|-------|
| Guam (unf) | Guam leaf | Guam potion (unf) | 3 |
| Ranarr (unf) | Ranarr weed | Ranarr potion (unf) | 30 |
| Toadflax (unf) | Toadflax | Toadflax potion (unf) | 34 |
| Snapdragon (unf) | Snapdragon | Snapdragon potion (unf) | 63 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Herb]
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Herb]
```

---

### Task 3: Stop (out of [Herb])
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Herb]
  └─ NOT Condition: Bank Contains [Herb] (quantity 1)
```

**Note:** This stops the whole script if you have no herbs left in your inventory AND the bank has none either. It keeps the script from spinning on an empty bank.

---

### Task 4: Stop (out of Vial of water)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: "Vial of water"
  └─ NOT Condition: Bank Contains "Vial of water" (quantity 1)
```

**Note:** Same idea as Task 3, but for Vials of water. When you run out of either ingredient, the script stops cleanly.

---

### Task 5: Withdraw Herbs
```
Type: Withdraw Item
Item: [Herb Name]
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Herb]
```

---

### Task 6: Withdraw Vials
```
Type: Withdraw Item
Item: "Vial of water"
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Herb]
  └─ NOT Has Item: "Vial of water"
```

---

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Herb]
  └─ Has Item: "Vial of water"
```

---

### Task 8: Use Herb on Vial
```
Type: Use Item on Item
Item 1: [Herb]
Item 2: "Vial of water"
Delay: 1 tick
Expect animation (retry if idle): ON
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Herb]
  ├─ Has Item: "Vial of water"
  ├─ NOT Menu Open
  └─ Idle
```

**Notes:**
- The `Idle` condition makes sure the player has stopped before clicking, so it won't spam the combine.
- **Expect animation (retry if idle)** is turned ON for this step. If you click but no making animation starts (you briefly idle), the script automatically re-clicks the herb on the vial instead of stalling. If it ever gets stuck it gives up after a few retries and moves on, so it never loops forever on one click.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 18
```

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: Make
Delay: 1 tick
Conditions:
  └─ Menu Open
```

**Note:** The Menu Option is set to `Make`, which clicks the first "Make" button in the make/skill menu. (You can also put a number here to pick a specific menu button.)

---

## Profit Information

Making unfinished potions can be profitable:

| Unf Potion | Typical Margin |
|------------|----------------|
| Guam (unf) | ~50-100 gp |
| Ranarr (unf) | ~200-400 gp |
| Toadflax (unf) | ~100-300 gp |
| Snapdragon (unf) | ~300-600 gp |

*Check GE prices - margins fluctuate*

---

## Tips

1. **Buy in bulk** - Herbs and vials are cheap
2. **No XP waste** - Making unf potions gives no XP, but is profitable
3. **Great for alts** - Low requirements, easy money
4. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
