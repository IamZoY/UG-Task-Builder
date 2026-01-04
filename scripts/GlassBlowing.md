# Glass Blowing Scripts

Scripts for blowing molten glass into various items.

---

## Overview

These scripts automate using a glassblowing pipe on molten glass to create glass items.

**Requirements:**
- Be standing at a bank
- Have a glassblowing pipe in your bank
- Have molten glass in your bank
- Required Crafting level

---

## Available Presets

| Preset | Product | Level | XP |
|--------|---------|-------|-----|
| Unpowered Orbs | Unpowered orb | 46 | 52.5 |
| Lantern Lens | Lantern lens | 49 | 55 |

---

## Other Glass Products (Not in Presets)

You can modify these scripts for other products:

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
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: "Molten glass"
```

---

### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: "Molten glass"
```

---

### Task 3: Withdraw Glassblowing Pipe
```
Type: Withdraw Item
Item: "Glassblowing pipe"
Quantity: 1
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: "Glassblowing pipe"
```

---

### Task 4: Withdraw Molten Glass
```
Type: Withdraw Item
Item: "Molten glass"
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Glassblowing pipe"
  └─ NOT Has Item: "Molten glass"
```

---

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: "Glassblowing pipe"
  └─ Has Item: "Molten glass"
```

---

### Task 6: Use Pipe on Glass
```
Type: Use Item on Item
Item 1: "Glassblowing pipe"
Item 2: "Molten glass"
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: "Molten glass"
  ├─ NOT Menu Open
  └─ Idle
```

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Max Ticks: 40
```

---

### Task 8: Select Menu Option
```
Type: Select Menu Option
Option: [See table above for correct option]
Delay: 1 tick
Conditions:
  └─ Menu Open
```

**Menu Options:**
- Unpowered orb: Option 6
- Lantern lens: Option 7

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

## Superglass Make Method

For maximum efficiency, use the Lunar spell "Superglass Make":

1. Withdraw giant seaweed (3) + bucket of sand (18)
2. Cast Superglass Make
3. Get ~30 molten glass per cast
4. Blow into orbs

This is the meta for Crafting + Magic XP!

---

## Tips

1. **Unpowered orbs** - Main use is charging for battlestaves
2. **Menu options vary** - Make sure you select the right product
3. **Make your own glass** - Much better profit than buying
4. **Crafting Guild** - Close bank + pottery wheel for making items
5. **Light orbs** - Best XP at 87 Crafting, also profitable
