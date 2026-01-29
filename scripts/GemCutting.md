# Gem Cutting Scripts

Scripts for cutting gems at a bank.

---

## Overview

These scripts automate cutting uncut gems with a chisel.

**Requirements:**
- Be standing at a bank
- Have a chisel in your bank
- Have uncut gems in your bank
- Required Crafting level

---

## Available Presets

| Preset | Uncut Gem | Result | Level | XP |
|--------|-----------|--------|-------|-----|
| Sapphires | Uncut sapphire | Sapphire | 20 | 50 |
| Emeralds | Uncut emerald | Emerald | 27 | 67.5 |
| Rubies | Uncut ruby | Ruby | 63 | 85 |
| Diamonds | Uncut diamond | Diamond | 43 | 107.5 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Uncut Gem]
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Uncut Gem]
```

---

### Task 3: Withdraw Chisel
```
Type: Withdraw Item
Item: "Chisel"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Chisel"
```

---

### Task 4: Withdraw Gems
```
Type: Withdraw Item
Item: [Uncut Gem Name]
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Chisel"
  └─ NOT Has Item: [Uncut Gem]
```

---

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Chisel"
  └─ Has Item: [Uncut Gem]
```

---

### Task 6: Use Chisel on Gem
```
Type: Use Item on Item
Item 1: "Chisel"
Item 2: [Uncut Gem]
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Uncut Gem]
  ├─ NOT Menu Open
  └─ Idle (Grace: 2t)
```

**Note:** The `Idle (Grace: 2t)` condition waits up to 2 ticks to confirm the player is truly idle before triggering. This prevents the task from firing while an animation is still starting.

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
Option: 1
Delay: 1 tick
Conditions:
  └─ Menu Open
```

---

## XP Rates (Approximate)

| Gem | XP/Gem | Gems/Hour | XP/Hour |
|-----|--------|-----------|---------|
| Sapphire | 50 | ~2,700 | ~135,000 |
| Emerald | 67.5 | ~2,700 | ~182,000 |
| Ruby | 85 | ~2,700 | ~230,000 |
| Diamond | 107.5 | ~2,700 | ~290,000 |

---

## Profit Information

Gem cutting is usually profitable:

| Gem | Typical Profit |
|-----|----------------|
| Sapphire | ~50-150 gp/gem |
| Emerald | ~100-200 gp/gem |
| Ruby | ~150-300 gp/gem |
| Diamond | ~200-400 gp/gem |

*Prices fluctuate - check GE*

---

## Alternative: Using Wait Animation Cycle

Instead of `Wait Animation` + `Select Menu`, you can use `Wait Animation Cycle` for a cleaner approach:

```
Task 7: Wait for Cutting to Complete
├─ Type: Wait Animation Cycle
├─ Grace Period: 2 ticks
└─ (Waits for animation to start AND stop - handles the full cycle)
```

This is useful when you want the script to wait for the entire cutting animation to complete before proceeding.

---

## Tips

1. **Fast XP** - Gem cutting is one of the fastest Crafting methods
2. **Profitable** - Usually makes money while training
3. **AFK-friendly** - Long action time per inventory
4. **Crushed gems** - Low level gems can be crushed (failed cut), higher level = less fails
5. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
6. **Wait Animation Cycle** - Use `Wait Animation Cycle` to wait for the complete crafting animation instead of just checking animation state
