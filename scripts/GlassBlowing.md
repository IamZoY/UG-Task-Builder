# Glass Blowing Scripts

Scripts for blowing molten glass into various items.

---

## Overview

These scripts automate using a glassblowing pipe on molten glass to create glass items. Your
glassblowing pipe is kept in your inventory across bank trips (the bank step deposits everything
**except** the pipe), so it is withdrawn once and not re-banked each refill.

**How to add one:** click `Presets ▾` on the Task Builder toolbar, open the **Glass Blowing**
category, and pick **Unpowered Orbs** or **Lantern Lens**. The preset is added to the side panel
as a Script group. Open the group to see its tasks, set the **Menu Option** on the last task to
your product (see the final task), make sure the **Loop** toggle is ON so it keeps refilling from
the bank, then press **START**. The button turns red and reads **STOP** while it runs — click it
to stop, or it stops itself when you run out of materials (see below).

**Requirements:**
- Be standing at a bank
- Have a glassblowing pipe in your bank
- Have molten glass in your bank
- The required Crafting level for your product
- **Loop** toggle ON (so it keeps refilling from the bank)

**Stops when you run out:** the preset ends cleanly on its own when the bank can't restock. If you
have no molten glass left in your inventory and none in the bank, it stops. It also stops if your
glassblowing pipe is missing from both inventory and bank. No babysitting needed — start it and
walk away.

---

## Available Presets

Open the **Presets ▾** menu, then **Glass Blowing**, and pick one of:

| Preset item | Product | Level | XP |
|--------|---------|-------|-----|
| Unpowered Orbs | Unpowered orb | 46 | 52.5 |
| Lantern Lens | Lantern lens | 49 | 55 |

Picking an item appends a ready-to-run Script to the side panel. Open it to see/edit the
tasks listed below. Both presets build the **identical** task list — they differ only in
the product you choose in the make menu (see the final task).

---

## Other Glass Products (Not in Presets)

You can adapt either preset for other products by changing the **Menu Option** on the
final task (see the last task):

| Product | Level | XP | Menu Option |
|---------|-------|-----|-------------|
| Beer glass | 1 | 17.5 | 1 |
| Candle lantern | 4 | 19 | 2 |
| Oil lamp | 12 | 25 | 3 |
| Vial | 33 | 35 | 4 |
| Fishbowl | 42 | 42.5 | 5 |
| Unpowered orb | 46 | 52.5 | 6 |
| Lantern lens | 49 | 55 | 7 |
| Light orb | 87 | 70 | 8 |

---

## Script Breakdown

### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks (±15%)
Conditions:
  ├─ Condition: Bank Closed
  └─ Condition: Has Item: "Molten glass"  (inverted / NOT)
```

---

### Task 2: Deposit All Except (keeps your Glassblowing pipe)
```
Type: Deposit All Except
Keep: "Glassblowing pipe"
Delay: 1 tick (±15%)
Conditions:
  └─ Condition: Has Item: "Molten glass"  (inverted / NOT)
```

**Note:** This is a **Deposit All Except** step with the **Item Name** set to `Glassblowing pipe`,
so it banks your finished glass items but keeps the pipe in your inventory. Because the pipe stays
with you, it is not re-withdrawn every bank trip — only the consumed molten glass is restocked.

---

### Task 3: Stop Script (out of Molten glass)
```
Type: Stop Script
Conditions:
  ├─ Condition: Bank Open
  ├─ Condition: Has Item: "Molten glass"  (inverted / NOT)
  └─ Condition: Bank Contains: "Molten glass"  (Comparator: <, Quantity: 1 — i.e. bank has none)
```

**Note:** This safety step stops the whole script when you have no molten glass in your inventory
and the bank can't supply any more.

---

### Task 4: Stop Script (no Glassblowing pipe)
```
Type: Stop Script
Conditions:
  ├─ Condition: Bank Open
  ├─ Condition: Has Item: "Glassblowing pipe"  (inverted / NOT)
  └─ Condition: Bank Contains: "Glassblowing pipe"  (Comparator: <, Quantity: 1 — i.e. bank has none)
```

**Note:** Stops the script if your glassblowing pipe is missing from both your inventory and the
bank, so it never spins without a tool.

---

### Task 5: Withdraw Glassblowing Pipe
```
Type: Withdraw Item
Item: "Glassblowing pipe"
Quantity: 1
Delay: 2 ticks (±15%)
Conditions:
  ├─ Condition: Bank Open
  └─ Condition: Has Item: "Glassblowing pipe"  (inverted / NOT)
```

---

### Task 6: Withdraw Molten Glass
```
Type: Withdraw Item
Item: "Molten glass"
Quantity: 27
Delay: 2 ticks (±15%)
Conditions:
  ├─ Condition: Bank Open
  ├─ Condition: Has Item: "Glassblowing pipe"
  └─ Condition: Has Item: "Molten glass"  (inverted / NOT)
```

---

### Task 7: Close Bank
```
Type: Close Bank
Delay: 1 tick (±15%)
Conditions:
  ├─ Condition: Bank Open
  ├─ Condition: Has Item: "Glassblowing pipe"
  └─ Condition: Has Item: "Molten glass"
```

---

### Task 8: Use Pipe on Glass
```
Type: Use Item on Item
Item 1: "Glassblowing pipe"
Item 2: "Molten glass"
Delay: 1 tick (±15%)
Expect animation (retry if idle): ON
Conditions:
  ├─ Condition: Bank Closed
  ├─ Condition: Has Item: "Glassblowing pipe"
  ├─ Condition: Has Item: "Molten glass"
  ├─ Condition: Menu Open  (inverted / NOT)
  └─ Condition: Idle
```

**Note:** This step only fires when the bank is closed, you still have your pipe and glass, the
make menu is not already open, and you are idle. The **Idle** condition is debounced — it means
"not animating for a few ticks" (this preset uses 3), so the click won't re-fire during the brief
animation gaps between individually blown items. The **Expect animation (retry if idle)** option
is turned on, so if you briefly idle and no glassblowing animation starts, the script re-issues the
"use pipe on glass" click instead of stalling.

---

### Task 9: Wait for Animation
```
Type: Wait Animation
Grace Period: 2 ticks
Max Ticks: 40 (±20%)
```

**Grace Period:** Waits up to ~2 ticks for the glassblowing animation to start before moving on.
**Max Ticks:** Caps how long this step will wait (~40 ticks).

---

### Task 10: Select Menu Option
```
Type: Select Menu Option
Menu Option: "Make"
Delay: 1 tick (±15%)
Conditions:
  └─ Condition: Menu Open
```

**Important — pick the right product:** Both presets ship with the **Menu Option** field set to
`Make`, which clicks the **first** button in the glassblowing make menu. If the product you want
is not the first button, change the **Menu Option** to the option number for your product:

| Product | Menu Option to use |
|---------|--------------------|
| Beer glass | 1 |
| Candle lantern | 2 |
| Oil lamp | 3 |
| Vial | 4 |
| Fishbowl | 5 |
| Unpowered orb | 6 |
| Lantern lens | 7 |
| Light orb | 8 |

So for the **Unpowered Orbs** preset set the Menu Option to `6`, and for **Lantern Lens** set it
to `7`. (The numbers match the order the products appear in your make menu and may shift as you
unlock higher-level items.)

---

## XP Rates (Approximate)

| Product | XP/Item | Items/Hour | XP/Hour |
|---------|---------|------------|---------|
| Unpowered Orb | 52.5 | ~1,400 | ~73,500 |
| Lantern Lens | 55 | ~1,400 | ~77,000 |
| Light Orb | 70 | ~1,400 | ~98,000 |

---

## Profit Information

Glass blowing profit depends on glass source:

### If Buying Molten Glass
| Product | Glass Cost | Product Value | Profit |
|---------|------------|---------------|--------|
| Unpowered Orb | ~150 gp | ~100 gp | -50 gp (loss) |
| Lantern Lens | ~150 gp | ~200 gp | +50 gp |
| Light Orb | ~150 gp | ~500 gp | +350 gp |

### If Making Your Own Glass (Giant Seaweed + Sand)
Much more profitable as molten glass costs ~20-50 gp to make!

---

## Superglass Make Method (build it yourself)

There is **no Superglass Make preset** — but for maximum efficiency you can build your own script
around the Lunar spell "Superglass Make" so you never have to buy molten glass:

1. **Withdraw Item** giant seaweed (e.g. 3) and **Withdraw Item** bucket of sand (e.g. 18)
2. **Cast Spell** "Superglass Make" (~30 molten glass per cast)
3. Then run the glassblowing steps above to blow the glass into orbs

This is a strong method for Crafting + Magic XP. Gate each step with conditions the same way the
preset does (Bank Open / Has Item / NOT Has Item) and keep the **Loop** toggle ON.

---

## Alternative: Using Wait Animation Cycle

Instead of `Wait Animation` + `Select Menu`, you can use `Wait Animation Cycle`:

```
Wait for Blowing to Complete
├─ Type: Wait Animation Cycle
├─ Grace Period: 2 ticks
└─ (Waits for animation to start AND stop)
```

---

## Tips

1. **Unpowered orbs** - Main use is charging for battlestaves
2. **Set the Menu Option** - The preset ships with `Make` (first menu button). Change the final task's **Menu Option** to your product's number (see Task 10) so you make the right item
3. **Make your own glass** - Much better profit than buying
4. **Crafting Guild** - Close bank + pottery wheel for making items
5. **Light orbs** - Best XP at 87 Crafting, also profitable
6. **Animation Issues?** - The preset's **Idle** condition already uses a 3-tick debounce. If the script still skips tasks after clicking, bump the Grace Period on the **Wait Animation** task (Task 9) up a tick or two
7. **Wait Animation Cycle** - Use `Wait Animation Cycle` to wait for the complete animation instead of just checking state
8. **Runs out of glass or pipe?** - The script stops itself cleanly when the bank can't restock molten glass, or when the glassblowing pipe is missing from both inventory and bank
