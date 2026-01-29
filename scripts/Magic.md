# Magic Scripts

Scripts for magic training at a bank (Plank Make, High Alchemy).

---

## Overview

These scripts automate casting magic spells on inventory items for training and profit.

**Requirements:**
- Be standing at a bank
- Have required runes equipped or in inventory
- Have target items in your bank
- Required Magic level

---

## Available Presets

| Preset | Spell | Target | Level | XP |
|--------|-------|--------|-------|-----|
| Plank Make (Oak) | Plank Make | Oak logs | 86 | 90 |
| Plank Make (Mahogany) | Plank Make | Mahogany logs | 86 | 90 |
| High Alch | High Level Alchemy | Any item | 55 | 65 |

---

## Plank Make Script

### Requirements
- 86 Magic
- Runes: 1 Nature, 2 Astral, 15 Earth (per cast)
- Coins for plank cost (70 gp oak, 1050 gp mahogany)
- Lunar spellbook active

### Script Breakdown

#### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Logs]
```

#### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Logs]
```

#### Task 3: Withdraw Logs
```
Type: Withdraw Item
Item: [Log Name]
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Logs]
```

#### Task 4: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  └─ Has Item: [Logs]
```

#### Tasks 5-31: Cast Plank Make (x27)
```
Type: Cast Spell on Item
Spell: "Plank Make"
Item: [Logs]
Delay: 3 ticks
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Logs]
  └─ Idle (Grace: 2t)
```
*Repeated 27 times for each log in inventory*

**Note:** The `Idle (Grace: 2t)` condition prevents casting too quickly by waiting for the previous spell's animation to complete.

---

## High Alchemy Script

### Requirements
- 55 Magic
- Runes: 1 Nature, 5 Fire (per cast)
- Standard or Ancient spellbook

### Script Breakdown

#### Task 1: Open Bank
```
Type: Open Bank
Delay: 2 ticks
Conditions:
  ├─ Bank Closed
  └─ NOT Has Item: [Alch Item]
```

#### Task 2: Deposit All
```
Type: Deposit All
Delay: 1 tick
Conditions:
  └─ NOT Has Item: [Alch Item]
```

#### Task 3: Withdraw Items
```
Type: Withdraw Item
Item: [Alch Item Name]
Quantity: 27
Delay: 2 ticks
Conditions:
  ├─ Bank Open
  └─ NOT Has Item: [Alch Item]
```

#### Task 4: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  └─ Has Item: [Alch Item]
```

#### Tasks 5-31: Cast High Alchemy (x27)
```
Type: Cast Spell on Item
Spell: "High Level Alchemy"
Item: [Alch Item]
Delay: 5 ticks
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Alch Item]
  └─ Idle (Grace: 2t)
```
*Repeated 27 times for each item*

**Note:** The `Idle (Grace: 2t)` condition prevents casting too quickly by waiting for the previous spell's animation to complete.

---

## XP Rates (Approximate)

| Method | XP/Cast | Casts/Hour | XP/Hour |
|--------|---------|------------|---------|
| Plank Make | 90 | ~1,800 | ~162,000 |
| High Alch | 65 | ~1,200 | ~78,000 |

---

## Profit Information

### Plank Make
| Plank | Cost | Sell Price | Profit |
|-------|------|------------|--------|
| Oak | Log + 70gp + runes | ~500 gp | ~200-300 gp |
| Mahogany | Log + 1050gp + runes | ~2,500 gp | ~400-800 gp |

### High Alchemy
Common profitable items:
- Rune items (platebodies, 2h swords)
- Battlestaves (all types)
- Dragonhide bodies
- Onyx bolts (e)

*Always check alch values vs buy price!*

---

## Tips

### Plank Make
1. **Bring coins** - Need coins for plank conversion
2. **Use Lunar Isle bank** - Close to altar if you need to swap books
3. **Rune pouch** - Saves inventory space
4. **Staff of earth** - Eliminates earth rune cost

### High Alchemy
1. **Explorer's ring** - Free alchs per day
2. **Nature rune cost** - Factor into profit calculations
3. **Fire staff** - Eliminates fire rune cost
4. **Buy limits** - Can only buy 70 of most items per 4 hours
5. **Alch value lookup** - Use wiki or GE to find profitable items

### Animation Grace Period
- **Animation Issues?** - If the script casts spells too quickly or skips casts, add an `Idle (Grace: 2t)` condition to spell casting tasks
- **Increase if needed** - Try 3 ticks if spells are still being skipped
