# Bow Stringing Scripts

Scripts for stringing unstrung bows with bowstrings.

---

## Overview

These scripts automate attaching bowstrings to unstrung bows to create finished bows.

**Requirements:**
- Be standing at a bank
- Have unstrung bows in your bank
- Have bow strings in your bank
- Required Fletching level

---

## Available Presets

| Preset | Unstrung Bow | Result | Level | XP |
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

### Task 3: Withdraw Unstrung Bows
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

### Task 4: Withdraw Bow Strings
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

### Task 5: Close Bank
```
Type: Close Bank
Delay: 1 tick
Conditions:
  ├─ Bank Open
  ├─ Has Item: [Unstrung Bow]
  └─ Has Item: "Bow string"
```

---

### Task 6: Attach String to Bow
```
Type: Use Item on Item
Item 1: [Unstrung Bow]
Item 2: "Bow string"
Delay: 1 tick
Conditions:
  ├─ Bank Closed
  ├─ Has Item: [Unstrung Bow]
  ├─ Has Item: "Bow string"
  ├─ NOT Menu Open
  └─ Idle
```

---

### Task 7: Wait for Animation
```
Type: Wait Animation
Max Ticks: 25
```

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
4. **No tool needed** - Just bows and strings
