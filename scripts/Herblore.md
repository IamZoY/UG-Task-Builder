# Herblore Scripts

Complete potion-making scripts for training Herblore at a bank.

---

## Overview

These scripts automate the process of combining unfinished potions with secondary ingredients to create finished potions.

**How to load one:** open the **Presets ▾** menu on the panel toolbar, choose the **Herblore** category, then pick a potion (for example **Attack Potion**). The generated Script is added to the side panel as an editable group of tasks. Open the group to view or tweak the steps, then press **START**.

**Requirements:**
- Be standing at a bank
- Have unfinished potions in your bank
- Have secondary ingredients in your bank

**Runs out cleanly:** the Script stops itself the moment the bank can no longer restock you. If you have no unfinished potions left in your inventory *and* none in the bank, a **Stop Script** task fires; the same happens when you run out of the secondary ingredient. The button flips back to green **START** on its own, so you can leave it unattended without it spinning in place.

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

### Task 3: Stop (out of unfinished potion)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Unf Potion]
  └─ Condition: Bank Contains [Unf Potion] (inverted: fewer than 1)
```
Stops the Script when you have no unfinished potions in your inventory and the bank can't supply any more.

---

### Task 4: Stop (out of secondary)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Secondary]
  └─ Condition: Bank Contains [Secondary] (inverted: fewer than 1)
```
Stops the Script when both your inventory and the bank are out of the secondary ingredient.

---

### Task 5: Withdraw Unfinished Potion
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

### Task 6: Withdraw Secondary Ingredient
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

### Task 7: Close Bank
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

### Task 8: Use Items Together
```
Type: Use Item on Item
Item 1: [Unf Potion]
Item 2: [Secondary]
Delay: 1 tick
Expect animation (retry if idle): ON
  ├─ Idle window: 1800 ms
  └─ Max retries: 3
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Unf Potion]
  ├─ Has Item: [Secondary]
  ├─ Condition: Menu Open (inverted / NOT)
  └─ Condition: Idle
```
Combines the unfinished potion with the secondary ingredient.

**Note:** The **Condition: Idle** check waits until you haven't been animating for a few ticks before clicking (this debounce means it won't re-click during the brief animation gaps between potions in a batch), and **Condition: Menu Open** is inverted so the click only fires while the make menu is closed.

**Idle-retry watchdog:** This step has **Expect animation (retry if idle)** turned ON. After the click, if the player goes idle without a mixing animation starting within about 1.8 seconds, the click is re-issued (up to 3 retries). This keeps the script from getting stuck if a click is missed.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Max Ticks: 25
```
Waits for the mixing animation before clicking the menu. This prevents spam-clicking. The **Max Ticks** value is the timeout, in game ticks, that it will wait for the animation.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: Make
Delay: 1 tick
Conditions:
  └─ Condition: Menu Open
```
Clicks the **Make** button in the make/skill menu once it is open.

---

## Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    START                                 │
└─────────────────────┬───────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Have unf potions?  ──NO──► Open Bank → Deposit All     │
│         │                   → (bank empty? STOP)        │
│        YES                  → Withdraw Unf (14)         │
│         │                   → Withdraw Secondary (14)   │
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
│  Click "Make"                                           │
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

Instead of `Wait Animation`, you can swap in a `Wait Animation Cycle` task:

```
Wait for Mixing to Complete
├─ Type: Wait Animation Cycle
├─ Grace Period: 2 ticks
└─ (Waits for animation to start AND stop)
```

---

## Tips

1. **Use a close bank** - Grand Exchange or Crafting Guild recommended
2. **Pre-buy supplies** - Have enough materials for your session. When the bank runs dry, the Script stops itself and flips the button back to green **START** - no manual babysitting needed to end it
3. **Enable Loop** - Toggle loop ON for continuous operation
4. **Monitor occasionally** - Check for random events or issues
5. **Animation Issues?** - If the script stalls after clicking, the idle-retry watchdog on the **Use Item on Item** step will normally re-issue the click on its own. If it still skips, raise the **Max Ticks** on the `Wait Animation` task (try 30 ticks)
6. **Wait Animation Cycle** - Use `Wait Animation Cycle` to wait for the complete mixing animation
