# Crafting Scripts (Leather)

Scripts for crafting dragonhide bodies at a bank.

---

## Overview

These scripts automate crafting dragonhide bodies using needle, thread, and dragon leather.

**Requirements:**
- Be standing at a bank
- Have needle in your bank
- Have thread in your bank
- Have dragon leather in your bank
- Required Crafting level

---

## Available Presets

| Preset | Leather | Product | Level | XP |
|--------|---------|---------|-------|-----|
| Green D'hide | Green dragon leather | Green d'hide body | 63 | 186 |
| Blue D'hide | Blue dragon leather | Blue d'hide body | 71 | 210 |
| Black D'hide | Black dragon leather | Black d'hide body | 79 | 258 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Leather]
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Leather]
```

---

### Task 3: Withdraw Needle
```
Type: Withdraw Item
Item: "Needle"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Needle"
```
Needle is only withdrawn if you don't have one (it's reusable).

---

### Task 4: Withdraw Thread
```
Type: Withdraw Item
Item: "Thread"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Needle"
  └─ NOT Has Item: "Thread"
```
Thread is consumed during crafting.

---

### Task 5: Withdraw Leather
```
Type: Withdraw Item
Item: [Leather Name]
Quantity: 26
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Needle"
  └─ NOT Has Item: [Leather]
```
Withdraws 26 leather (leaves room for needle + thread).

---

### Task 6: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Needle"
  └─ Has Item: [Leather]
```

---

### Task 7: Use Needle on Leather
```
Type: Use Item on Item
Item 1: "Needle"
Item 2: [Leather]
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Leather]
  ├─ NOT Menu Open
  └─ Idle (Grace: 2t)
```

**Note:** The `Idle (Grace: 2t)` condition waits up to 2 ticks to confirm the player is truly idle before triggering.

---

### Task 8: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 35
```

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 9: Select Menu Option
```
Type: Select Menu Option
Option: 1
Delay: 1 tick
Conditions:
  └─ Menu Open
```

---

## XP Rates (Approximate)

| D'hide Body | XP/Body | Bodies/Hour | XP/Hour |
|-------------|---------|-------------|---------|
| Green | 186 | ~950 | ~177,000 |
| Blue | 210 | ~950 | ~200,000 |
| Black | 258 | ~950 | ~245,000 |

---

## Profit/Loss

D'hide bodies often sell for LESS than the leather cost:

| Body | Typical Loss |
|------|--------------|
| Green | ~200-500 gp/body |
| Blue | ~300-700 gp/body |
| Black | ~500-1000 gp/body |

*This is a training method, not a money maker*

---

## Tips

1. **Thread consumption** - Thread has 5 uses, you'll need more over time
2. **Needle doesn't break** - You only need 1 needle
3. **3 leather per body** - 26 leather = 8 bodies with 2 leftover
4. **Crafting Guild** - Best bank location for crafting
5. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
