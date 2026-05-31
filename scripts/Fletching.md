# Fletching Scripts

Scripts for fletching logs into bows at a bank.

---

## Overview

These scripts automate cutting logs with a knife to create unstrung bows or arrow shafts.

To create one, click **Presets ▾** in the Task Builder panel, open the **Fletching** category, and choose one of the three items below. The selected script is added to the panel ready to **START**. Leave the **Loop** toggle ON so it keeps re-banking and fletching the next inventory automatically.

**Requirements:**
- Be standing at a bank
- Have a knife in your bank
- Have logs in your bank
- Required Fletching level

---

## Available Presets

Found under **Presets ▾ → Fletching**:

| Preset (menu item) | Logs withdrawn | Level | XP |
|--------|------|-------|-----|
| Arrow Shafts | Logs | 1 | 5 |
| Yew Longbows | Yew logs | 70 | 75 |
| Magic Longbows | Magic logs | 85 | 75 |

Each preset deposits your finished items (keeping the **Knife**), withdraws a full load (27) of the matching logs, then makes the first product offered by the fletching menu (see the Select Menu Option task). Your knife is your tool, so it stays in your inventory between trips rather than being banked and re-withdrawn each cycle. The script **stops cleanly when you run out of materials** - if your inventory has no logs AND the bank has no more logs (or you have no knife and none in the bank), the script ends itself instead of looping forever (see Tasks 3 and 4).

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

### Task 2: Deposit All Except (keeps Knife)
```
Type: Deposit All Except
Keep: "Knife"
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Logs]
```

**Note:** This deposits everything **except your Knife**. The knife is your fletching tool, so it stays in your inventory and is not re-banked or re-withdrawn every trip - only the finished bows/shafts get deposited. The Withdraw Knife step below is just a safety net for the first run (or if you ever lose the knife).

---

### Task 3: Stop (out of logs)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: [Logs]
  └─ Bank Contains: [Logs] < 1
```

**Note:** If your inventory has no logs AND the bank is also out of them, this step **stops the script** so it never spins in place. The Start/Stop button turns back to green **START**.

---

### Task 4: Stop (no Knife)
```
Type: Stop Script
Conditions:
  ├─ Bank Open
  ├─ NOT Has Item: "Knife"
  └─ Bank Contains: "Knife" < 1
```

**Note:** Fletching needs a knife. If you have no knife in your inventory AND none in the bank, this step **stops the script**.

---

### Task 5: Withdraw Knife
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

### Task 6: Withdraw Logs
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

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Knife"
  └─ Has Item: [Logs]
```

---

### Task 8: Use Knife on Logs
```
Type: Use Item on Item
Item 1: "Knife"
Item 2: [Logs]
Delay: 1 tick
Expect animation (retry if idle): ON
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Knife"
  ├─ Has Item: [Logs]
  ├─ NOT Menu Open
  └─ Idle
```

**Note:** This step has **Expect animation (retry if idle)** turned on, so if the click is missed and you stay idle, it re-clicks the knife on the logs instead of stalling the script. The **Idle** condition is debounced (it means "not animating for a few ticks"), so the step won't re-click during the brief animation gaps between each fletched item.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Max Ticks: 35
```

**Max Ticks:** Waits while you are animating, up to 35 ticks, before letting the menu step run. Raise it if the script skips ahead too early.

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: "Make"
Delay: 1 tick
Conditions:
  └─ Menu Open
```

**Note:** This step clicks **Make** — the first option in the fletching "What would you like to make?" menu. The preset always makes that first option (for example a longbow when the bow menu opens). If you want a different item, open this task and change the **Menu Option** field to the exact option text or to its number.

> The tasks are checked by their conditions, not run in a fixed line order — each task fires only when its conditions are all true, so the bank/withdraw/fletch cycle repeats on its own with the **Loop** toggle ON.

---

## XP Rates (Approximate)

| Product | XP/Item | Items/Hour | XP/Hour |
|---------|---------|------------|---------|
| Arrow Shafts | 5 (x15 = 75) | ~1,350 sets | ~101,000 |
| Yew Longbow (u) | 75 | ~1,500 | ~112,500 |
| Magic Longbow (u) | 75 | ~1,500 | ~112,500 |

---

## Profit Information

| Product | Typical Profit/Loss |
|---------|---------------------|
| Arrow Shafts | ~10-30 gp profit/log |
| Yew Longbow (u) | ~50-150 gp profit |
| Magic Longbow (u) | ~100-300 gp profit |

---

## Tips

1. **Make sure the right option is first** - The preset always picks the top option of the fletching menu (Make). If your menu offers more than one item, edit the **Select Menu Option** task's **Menu Option** field to the exact item name or number you want.
2. **String bows for more XP** - The Yew/Magic Longbows presets give you unstrung bows; run the Bow Stringing presets afterward to finish them.
3. **Arrow shafts** - Good for ironmen or low levels (uses regular Logs).
4. **Keep Loop ON** - The conditions handle re-banking on their own, so the script refills and fletches the next load automatically each cycle. With Loop OFF it would only run one load and then idle until you press **STOP**.
5. **It stops itself when you run out** - When your inventory is empty and the bank has no more logs (or no knife), the script ends on its own and the button goes back to green **START**. No need to babysit it overnight.
6. **Script skipping the make step?** - If it stops clicking after the menu opens, open the **Wait for Animation** task and raise its **Max Ticks** a little, or re-check that a knife and logs are still in your bank.
