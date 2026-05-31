# Crafting Scripts (Leather)

Scripts for crafting dragonhide bodies at a bank.

---

## Overview

These scripts automate crafting dragonhide bodies using a needle, thread, and dragon leather.

**How to add it:** click **Presets ▾** in the Task Builder panel, open the **Crafting** category, and pick one of the three D'hide items. The preset is added as a Script group you can open and edit, then run with **START**. Leave the **Loop** toggle ON (it defaults ON) so it keeps re-banking and crafting batch after batch.

**Requirements:**
- Be standing at a bank
- Have a needle in your bank
- Have thread in your bank
- Have dragon leather in your bank
- Required Crafting level

**Stops when you run out of materials.** This preset cleanly stops itself when the bank can no longer restock what it needs: if you run out of leather, or run out of thread, a **Stop Script** task fires and the script ends instead of looping forever. The needle is the tool here — if the needle is missing from both your inventory and bank, the script stops too (it can't craft without it).

---

## Available Presets

| Preset | Leather | Product | Level | XP |
|--------|---------|---------|-------|-----|
| Green D'hide | Green dragon leather | Green d'hide body | 63 | 186 |
| Blue D'hide | Blue dragon leather | Blue d'hide body | 71 | 210 |
| Black D'hide | Black dragon leather | Black d'hide body | 79 | 258 |

---

## How the leather count gate works

Instead of "make until empty", this preset uses a **minimum-quantity gate**. It only crafts while you're holding **at least 3 leather** (the amount needed for one d'hide body). When you drop below 3, it re-banks: it deposits everything **except your Needle and Thread** (this also clears the finished bodies *and* any leftover leather), withdraws a fresh batch of leather, and carries on. When the bank can't supply a full body's worth of leather, it stops.

That "3" is editable. It's the **Quantity** on the **Inv Count** conditions inside the make/open/close/withdraw steps. If you adapt the preset to make chaps set it to **2**, and for vambraces set it to **1**.

---

## Script Breakdown

### Task 1 + 2: Open Bank (two triggers)

The preset has **two** Open Bank tasks. A task only runs when ALL of its conditions are true, so two separate triggers are used — either one will open the bank.

```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  ├─ Idle
  └─ Inv Count: [Leather] < 3
```
Opens the bank when you've dropped below the minimum leather. The **Idle** condition waits until the current batch finishes animating before opening (Idle is debounced — it means "not animating for a few ticks", so it won't trip during the brief gaps between made items).

```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  ├─ Idle
  └─ NOT Has Item: "Thread"
```
Also opens the bank when you've run out of thread.

---

### Task 3 + 4: Stop Script (out of materials)

```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ Inv Count: [Leather] < 3
  └─ Bank Contains: [Leather] < 3
```
Stops the script when you're short on leather AND the bank can't supply a full body's worth.

```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: "Thread"
  └─ Bank Contains: "Thread" < 1
```
Stops the script when you're out of thread AND the bank has none left.

---

### Task 5: Deposit All Except (keeps Needle + Thread)

```
Type: Deposit All Except
Keep: "Needle,Thread"
Delay: 1 tick
Conditions:
  ├─ Bank Open
  └─ Inv Count: [Leather] < 3
```
Deposits everything *except* your **Needle** and **Thread** when restocking. The needle is your tool and thread is a slow-consumed stackable, so both stay in your inventory and are not re-withdrawn every trip. This step clears the finished bodies and any leftover leather so you start each batch clean, while keeping the tools you need.

---

### Task 6: Withdraw Needle

```
Type: Withdraw Item
Item: "Needle"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Needle"
```
The needle is only withdrawn if you don't already have one (it's reusable and never breaks).

---

### Task 7: Withdraw Thread

```
Type: Withdraw Item
Item: "Thread"
Quantity: -1  (withdraw all)
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Thread"
```
Thread is stackable and gets consumed continuously, so the preset withdraws a **full stack** (Quantity -1 = withdraw all).

---

### Task 8: Withdraw Leather

```
Type: Withdraw Item
Item: [Leather Name]
Quantity: 26
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ Inv Count: [Leather] < 3
```
Withdraws a fresh batch of 26 leather (leaving room for the needle + stacked thread).

---

### Task 9: Close Bank

```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Inv Count: [Leather] >= 3
  ├─ Has Item: "Needle"
  └─ Has Item: "Thread"
```
Only closes the bank once you can actually craft: enough leather, plus a needle and thread.

---

### Task 10: Use Needle on Leather (the "make" step)

```
Type: Use Item on Item
Item 1: "Needle"
Item 2: [Leather]
Delay: 1 tick
Expect animation (retry if idle): ON
  ├─ Idle window: 1800 ms
  └─ Max retries: 3
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Needle"
  ├─ Has Item: "Thread"
  ├─ Inv Count: [Leather] >= 3
  ├─ NOT Menu Open
  └─ Idle
```

**Note:** This is the "make" step. The **Expect animation (retry if idle)** option is turned ON, so if you go idle and no crafting animation starts within ~1.8 seconds, the step re-clicks the needle on the leather (up to 3 retries) before moving on. The **Idle** condition makes sure you're not already animating before it fires, and because Idle is debounced it won't re-click during the brief 1-tick gaps between individually made bodies.

---

### Task 11: Wait for Animation

```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 35
```

**Grace Period:** Waits up to 2 ticks for the animation to start before checking. Increase to 3 ticks if the script skips this step.

---

### Task 12: Select Menu Option

```
Type: Select Menu Option
Menu Option: "Make"
Delay: 1 tick
Conditions:
  └─ Menu Open
```
This clicks "Make" when the make-all menu appears.

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

1. **Stocks up your bank** - Keep plenty of leather and thread in the bank. When either runs out the script stops cleanly on its own.
2. **Needle doesn't break** - You only need 1 needle; it's reused every batch.
3. **3 leather per body** - The minimum-quantity gate is set to 3 (a d'hide body). It's the Quantity on the Inv Count conditions; set it to 2 for chaps or 1 for vambraces if you adapt the script.
4. **Crafting Guild** - A handy bank location for crafting.
5. **Animation Issues?** - If the script skips the make step after clicking, increase the Grace Period on the `Wait Animation` task (try 3 ticks). The make step's **Expect animation (retry if idle)** option also re-clicks for you if you go idle.
6. **Loop toggle** - Leave Loop ON so it re-banks and keeps going. The script runs until it runs out of materials or you click **STOP** (the button is red **STOP** while running, green **START** when stopped).
