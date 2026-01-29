# Herblore Scripts

Complete potion-making scripts for training Herblore at a bank.

---

## Overview

These scripts automate the process of combining unfinished potions with secondary ingredients to create finished potions.

**Requirements:**
- Be standing at a bank
- Have unfinished potions in your bank
- Have secondary ingredients in your bank

---

## Available Presets

| Preset | Unf Potion | Secondary | Level |
|--------|-----------|-----------|-------|
| Attack Potion | Guam potion (unf) | Eye of newt | 3 |
| Strength Potion | Tarromin potion (unf) | Limpwurt root | 12 |
| Prayer Potion | Ranarr potion (unf) | Snape grass | 38 |
| Super Attack | Irit potion (unf) | Eye of newt | 45 |
| Super Strength | Kwuarm potion (unf) | Limpwurt root | 55 |
| Super Restore | Snapdragon potion (unf) | Red spiders' eggs | 63 |
| Saradomin Brew | Toadflax potion (unf) | Crushed nest | 81 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Unf Potion]
```
Opens the bank when you don't have the unfinished potion in your inventory.

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Unf Potion]
```
Deposits everything when you're out of unfinished potions (finished potions get banked).

---

### Task 3: Withdraw Unfinished Potion
```
Type: Withdraw Item
Item: [Unf Potion Name]
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Unf Potion]
```
Withdraws 14 unfinished potions from the bank.

---

### Task 4: Withdraw Secondary Ingredient
```
Type: Withdraw Item
Item: [Secondary Name]
Quantity: 14
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Unf Potion]
  └─ NOT Has Item: [Secondary]
```
Withdraws 14 secondary ingredients only after you have the unf potions.

---

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Unf Potion]
  └─ Has Item: [Secondary]
```
Closes the bank once you have both items.

---

### Task 6: Use Items Together
```
Type: Use Item on Item
Item 1: [Unf Potion]
Item 2: [Secondary]
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Unf Potion]
  ├─ Has Item: [Secondary]
  ├─ NOT Menu Open
  └─ Idle (Grace: 2t)
```
Combines the unfinished potion with the secondary ingredient.

**Note:** The `Idle (Grace: 2t)` condition waits up to 2 ticks to confirm the player is truly idle before triggering.

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 25
```
Waits for the mixing animation before clicking the menu. This prevents spam-clicking.

**Grace Period:** Waits up to 2 ticks for animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 8: Select Menu Option
```
Type: Select Menu Option
Option: 1 (Make All)
Delay: 1 tick
Conditions:
  └─ Menu Open
```
Clicks "Make All" in the skill menu.

---

## Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    START                                 │
└─────────────────────┬───────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Have unf potions?  ──NO──► Open Bank → Deposit All     │
│         │                   → Withdraw Unf (14)         │
│        YES                  → Withdraw Secondary (14)   │
│         │                   → Close Bank                │
│         ▼                          │                    │
│  ◄─────────────────────────────────┘                    │
│         │                                               │
│         ▼                                               │
│  Use Unf on Secondary                                   │
│         │                                               │
│         ▼                                               │
│  Wait for Animation                                     │
│         │                                               │
│         ▼                                               │
│  Click "Make All"                                       │
│         │                                               │
│         ▼                                               │
│  [Potions being made...]                                │
│         │                                               │
│         ▼                                               │
│  Loop back to start                                     │
└─────────────────────────────────────────────────────────┘
```

---

## XP Rates (Approximate)

| Potion | XP Each | XP/Hour (est.) |
|--------|---------|----------------|
| Attack | 25 | ~60,000 |
| Strength | 50 | ~120,000 |
| Prayer | 87.5 | ~210,000 |
| Super Attack | 100 | ~240,000 |
| Super Strength | 125 | ~300,000 |
| Super Restore | 142.5 | ~340,000 |
| Saradomin Brew | 180 | ~430,000 |

*Rates vary based on bank location and ping*

---

## Alternative: Using Wait Animation Cycle

Instead of `Wait Animation` + `Select Menu`, you can use `Wait Animation Cycle`:

```
Task 7: Wait for Mixing to Complete
├─ Type: Wait Animation Cycle
├─ Grace Period: 2 ticks
└─ (Waits for animation to start AND stop)
```

---

## Tips

1. **Use a close bank** - Grand Exchange or Crafting Guild recommended
2. **Pre-buy supplies** - Have enough materials for your session
3. **Enable Loop** - Toggle loop ON for continuous operation
4. **Monitor occasionally** - Check for random events or issues
5. **Animation Issues?** - If the script skips tasks after clicking, increase the Grace Period on the `Idle` condition or `Wait Animation` task (try 3 ticks)
6. **Wait Animation Cycle** - Use `Wait Animation Cycle` to wait for the complete mixing animation
