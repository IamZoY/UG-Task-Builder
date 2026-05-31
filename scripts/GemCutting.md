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

Open the **Presets ▾** menu, hover **Gem Cutting**, then pick the gem you want:

| Preset (Presets ▾ → Gem Cutting →) | Uncut Gem | Result | Level | XP |
|--------|-----------|--------|-------|-----|
| Sapphires | Uncut sapphire | Sapphire | 20 | 50 |
| Emeralds | Uncut emerald | Emerald | 27 | 67.5 |
| Rubies | Uncut ruby | Ruby | 63 | 85 |
| Diamonds | Uncut diamond | Diamond | 43 | 107.5 |

Picking one adds a new Script to the side panel with the 10 tasks below already set up. Make sure the **Loop**
toggle is ON (it is by default) so the Script keeps refilling from the bank, then press **START** (the button
turns red and reads **STOP** while running). Press **STOP** when you want to finish — there is no auto-stop; the
Script keeps cycling until you stop it or it runs out of gems/your chisel (see below).

Your **Chisel stays in your inventory** the whole time. The bank step is a **Deposit All Except** that keeps
your Chisel, so only finished gems get banked and the chisel is never re-withdrawn every trip.

### Stops when you run out

This preset stops itself cleanly when the bank can no longer supply what it needs. Two **Stop Script** tasks
are built in:

- **Stop (out of [Uncut Gem])** — fires when you have no uncut gems in your inventory AND the bank is empty of
  them.
- **Stop (no Chisel)** — fires when you have no chisel in your inventory AND the bank has none either.

When either condition is met the Script stops on its own, so you don't burn cycles clicking an empty bank.

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

### Task 2: Deposit All Except (keeps Chisel)
```
Type: Deposit All Except
Item 1 (keep list): "Chisel"
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Uncut Gem]
```

This deposits everything except your **Chisel**, so the chisel stays in your inventory between trips and only
the finished gems get banked.

---

### Task 3: Stop (out of [Uncut Gem])
```
Type: Stop Script
Conditions:
  ├─ Condition: Bank Open
  ├─ NOT Has Item: [Uncut Gem]
  └─ Condition: Bank Contains [Uncut Gem]  (inverted / NOT — i.e. fewer than 1 in the bank)
```

Ends the Script when you have no gems left and the bank can't refill you.

---

### Task 4: Stop (no Chisel)
```
Type: Stop Script
Conditions:
  ├─ Condition: Bank Open
  ├─ NOT Has Item: "Chisel"
  └─ Condition: Bank Contains "Chisel"  (inverted / NOT — i.e. none in the bank)
```

Ends the Script when your chisel is gone and the bank has no spare.

---

### Task 5: Withdraw Chisel
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

### Task 6: Withdraw Gems
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

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Chisel"
  └─ Has Item: [Uncut Gem]
```

---

### Task 8: Use Chisel on Gem
```
Type: Use Item on Item
Item 1: "Chisel"
Item 2: [Uncut Gem]
Delay: 1 tick
Expect animation (retry if idle): ON
Conditions:
  ├─ Condition: Bank Closed
  ├─ Condition: Has Item: "Chisel"
  ├─ Condition: Has Item: [Uncut Gem]
  ├─ Condition: Menu Open  (inverted / NOT)
  └─ Condition: Idle (Grace: 3t)
```

**Note:** The `Condition: Idle (Grace: 3t)` waits until you've not been animating for about 3 ticks to confirm you're truly idle before triggering. This stops the task from firing during the brief animation gap between each gem, so it won't re-click mid-batch.

**Idle-retry:** This step has **Expect animation (retry if idle)** turned on. If you click but the player briefly idles and no cutting animation starts, the step re-issues the click instead of stalling, so the Script keeps cutting without your help.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 35
```

**Grace Period:** Waits up to 2 ticks for the cutting animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: Make
Delay: 1 tick
Conditions:
  └─ Condition: Menu Open
```

**Note:** The Menu Option is set to `Make`, which clicks the first "Make" button in the gem-cutting menu (cut all of the selected gem).

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
Task 9: Wait for Cutting to Complete
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
5. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition (try 4 ticks) or on the `Wait Animation` task (try 3 ticks)
6. **Wait Animation Cycle** - Use `Wait Animation Cycle` to wait for the complete crafting animation instead of just checking animation state
