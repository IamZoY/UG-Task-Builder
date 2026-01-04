# Unfinished Potion Scripts

Scripts for creating unfinished potions by combining herbs with vials of water.

---

## Overview

These scripts automate creating unfinished potions, which are the base for all herblore potions.

**Requirements:**
- Be standing at a bank
- Have clean herbs in your bank
- Have vials of water in your bank

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

### Task 3: Withdraw Herbs
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

### Task 4: Withdraw Vials
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

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Herb]
  └─ Has Item: "Vial of water"
```

---

### Task 6: Use Herb on Vial
```
Type: Use Item on Item
Item 1: [Herb]
Item 2: "Vial of water"
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Herb]
  ├─ Has Item: "Vial of water"
  ├─ NOT Menu Open
  └─ Idle
```

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Max Ticks: 18
```

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
