# Bow Stringing Scripts

Scripts for stringing unstrung bows with bowstrings.

---

## Overview

These scripts automate attaching bowstrings to unstrung bows to create finished bows. The script **stops cleanly when you run out of materials** - when you have no unstrung bows (or no bow strings) left in both your inventory and the bank, it stops itself instead of looping forever.

**Requirements:**
- Be standing at a bank
- Have unstrung bows in your bank
- Have bow strings in your bank
- Required Fletching level

---

## Available Presets

Open the **Presets ▾** menu in the panel toolbar, then choose **Bow Stringing** and pick one of these items. The generated script is appended to the side panel as a new group.

| Preset (Presets ▾ → Bow Stringing) | Unstrung Bow | Result | Level | XP |
|--------|--------------|--------|-------|-----|
| Yew Longbows | Yew longbow (u) | Yew longbow | 70 | 75 |
| Magic Longbows | Magic longbow (u) | Magic longbow | 85 | 91.5 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Unstrung Bow]
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Unstrung Bow]
```

---

### Task 3: Stop (out of unstrung bows)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Unstrung Bow]
  └─ NOT Bank Contains: [Unstrung Bow] (1+)
```

Stops the script cleanly when you have no unstrung bows in your inventory **and** none left in the bank.

---

### Task 4: Stop (out of Bow string)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: "Bow string"
  └─ NOT Bank Contains: "Bow string" (1+)
```

Stops the script cleanly when you have no bow strings in your inventory **and** none left in the bank.

---

### Task 5: Withdraw Unstrung Bows
```
Type: Withdraw Item
Item: [Unstrung Bow Name]
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Unstrung Bow]
```

---

### Task 6: Withdraw Bow Strings
```
Type: Withdraw Item
Item: "Bow string"
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Unstrung Bow]
  └─ NOT Has Item: "Bow string"
```

---

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Unstrung Bow]
  └─ Has Item: "Bow string"
```

---

### Task 8: Attach String to Bow
```
Type: Use Item on Item
Item 1: [Unstrung Bow]
Item 2: "Bow string"
Delay: 1 tick
Expect animation (retry if idle): ON
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Unstrung Bow]
  ├─ Has Item: "Bow string"
  ├─ NOT Menu Open
  └─ Idle (Grace: 3t)
```

**Note:** The `Idle (Grace: 3t)` condition requires the player to be not animating for 3 ticks (a debounced idle) before triggering, so it won't re-click during the brief animation gaps between strung bows.

**Idle-retry:** This step has **Expect animation (retry if idle)** turned on. After it uses the bow on the string, it watches for the stringing animation to start. If you briefly idle and nothing happens, it re-issues the click automatically (it re-checks you still hold both items first) so the script does not stall. After a few failed retries it marks the step stuck and moves on rather than looping forever.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 25
```

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase if the script skips this step.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: Make
Delay: 1 tick
Conditions:
  └─ Menu Open
```

---

## XP Rates (Approximate)

| Bow | XP/Bow | Bows/Hour | XP/Hour |
|-----|--------|-----------|---------|
| Yew Longbow | 75 | ~2,000 | ~150,000 |
| Magic Longbow | 91.5 | ~2,000 | ~183,000 |

---

## Profit Information

Stringing bows is often profitable:

| Bow | Typical Profit |
|-----|----------------|
| Yew Longbow | ~100-300 gp/bow |
| Magic Longbow | ~200-500 gp/bow |

*Strung bows alch for more than unstrung + string cost*

---

## Tips

1. **Fast method** - Stringing is faster than cutting
2. **Combine with High Alch** - String bows then alch for profit
3. **14+14 inventory** - Equal amounts of bows and strings
4. **No tool needed** - Just bows and strings (so no tool gate - it only stops when the bows or strings run out)
5. **Loop toggle** - Leave the Loop toggle ON so the bank/string cycle repeats; the script ends on its own when materials run out, or click the red **STOP** button any time
6. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Wait Animation` task or the grace on the `Idle` condition
