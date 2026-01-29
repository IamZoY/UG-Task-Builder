# Fletching Scripts

Scripts for fletching logs into bows at a bank.

---

## Overview

These scripts automate cutting logs with a knife to create unstrung bows or arrow shafts.

**Requirements:**
- Be standing at a bank
- Have a knife in your bank
- Have logs in your bank
- Required Fletching level

---

## Available Presets

| Preset | Logs | Product | Level | XP |
|--------|------|---------|-------|-----|
| Arrow Shafts | Logs | Arrow shaft (x15) | 1 | 5 |
| Yew Longbows | Yew logs | Yew longbow (u) | 70 | 75 |
| Magic Longbows | Magic logs | Magic longbow (u) | 85 | 91.5 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Logs]
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Logs]
```

---

### Task 3: Withdraw Knife
```
Type: Withdraw Item
Item: "Knife"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Knife"
```

---

### Task 4: Withdraw Logs
```
Type: Withdraw Item
Item: [Log Name]
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Knife"
  └─ NOT Has Item: [Logs]
```

---

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Knife"
  └─ Has Item: [Logs]
```

---

### Task 6: Use Knife on Logs
```
Type: Use Item on Item
Item 1: "Knife"
Item 2: [Logs]
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Logs]
  ├─ NOT Menu Open
  └─ Idle (Grace: 2t)
```

**Note:** The `Idle (Grace: 2t)` condition waits up to 2 ticks to confirm the player is truly idle before triggering.

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 35
```

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 8: Select Menu Option
```
Type: Select Menu Option
Option: 1 (or specific bow type)
Delay: 1 tick
Conditions:
  └─ Menu Open
```

**Note:** The menu option depends on what you want to make:
- Arrow shafts: Option 1
- Shortbow (u): Option 1
- Longbow (u): Option 2

---

## XP Rates (Approximate)

| Product | XP/Item | Items/Hour | XP/Hour |
|---------|---------|------------|---------|
| Arrow Shafts | 5 (x15 = 75) | ~1,350 sets | ~101,000 |
| Yew Longbow (u) | 75 | ~1,500 | ~112,500 |
| Magic Longbow (u) | 91.5 | ~1,500 | ~137,000 |

---

## Profit Information

| Product | Typical Profit/Loss |
|---------|---------------------|
| Arrow Shafts | ~10-30 gp profit/log |
| Yew Longbow (u) | ~50-150 gp profit |
| Magic Longbow (u) | ~100-300 gp profit |

---

## Tips

1. **Menu selection matters** - Make sure you select the right bow type
2. **Longbows > Shortbows** - Longbows give more XP
3. **String bows for more XP** - Combine with Bow Stringing script
4. **Arrow shafts** - Good for ironmen or low levels
5. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
