# Olm Basics (Boss Preset)

A reactive-prayer walkthrough for the **Olm Basics** preset: `Presets ▾ → Boss → Olm Basics`.

This preset flips your overhead protection prayer the instant the Great Olm announces a basic
attack, by reading the boss's chat message. It uses the Task Builder's **reactive prayer
triggers** — a layer that reacts to live in-game events and pre-empts the normal task order, so
your overhead is already correct when the hit lands.

---

## Overview

When you insert the preset you get a **group** named "Olm Basics" containing **three editable
tasks**, one per Olm basic-attack sphere:

| Task name         | Chat phrase matched (lowercased substring) | Overhead prayer         |
|-------------------|--------------------------------------------|-------------------------|
| Olm: Melee sphere | `sphere of aggression`                     | `PROTECT_FROM_MELEE`    |
| Olm: Magic sphere | `sphere of magical power`                  | `PROTECT_FROM_MAGIC`    |
| Olm: Range sphere | `sphere of accuracy and dexterity`         | `PROTECT_FROM_MISSILES` |

Each is a **Prayer Trigger: Chat** task. There is nothing else to configure for the basic
rotation: insert it, press **START**, fight Olm.

**Requirements:**
- Be in the Olm room of the Chambers of Xeric.
- Have the three protection prayers available (the trigger toggles the overhead directly, so you
  don't need quick-prayers set up — you just need the prayer levels and prayer points).
- The UG Task Builder plugin enabled and the script started (**START** button).

---

## How the script is achieved (step by step)

### 1. Insert the preset

`Presets ▾ → Boss → Olm Basics` adds a single group to the side panel. Open the group to see the
three child **Prayer Trigger: Chat** tasks. Double-click any task to view or edit its two fields —
a Chat trigger only shows **Chat phrase** and **Trigger prayer**:

```
Type: Prayer Trigger: Chat
  Chat phrase:    sphere of aggression
  Trigger prayer: PROTECT_FROM_MELEE
```

```
Type: Prayer Trigger: Chat
  Chat phrase:    sphere of magical power
  Trigger prayer: PROTECT_FROM_MAGIC
```

```
Type: Prayer Trigger: Chat
  Chat phrase:    sphere of accuracy and dexterity
  Trigger prayer: PROTECT_FROM_MISSILES
```

(A Chat trigger always **holds** its prayer, so the editor hides the match-id, hit/lead-ticks, and
priority fields you see on the other two trigger types. All three preset tasks carry a priority of
`1` internally — lower wins if two triggers ever fire at once — but you won't edit that on a Chat
trigger.)

### 2. Press START — the triggers arm

When you press **START**, every Prayer Trigger task in your script is armed. From then on the
three Chat triggers watch the game chat for their phrase. They do not take a slot in the normal
task order and never block — their only job is to react to chat. (Because of that, Olm Basics
works with **Loop** in either position; the prayer reaction is independent of the task scan.)

### 3. Olm fires — the chat message arrives

Olm announces every basic attack in the game chat, for example:

```
The Great Olm fires a sphere of magical power your way.
```

…or, when it has sapped your prayer:

```
The Great Olm fires a sphere of magical power your way. Your prayers have been sapped.
```

### 4. The phrase is matched (the "sapped" suffix doesn't matter)

A trigger fires when its **Trigger Phrase** appears anywhere in the chat line (case-insensitive).
Because each phrase is only the distinguishing fragment — `sphere of magical power` — the
trailing `your way.` and the optional `Your prayers have been sapped.` part are **irrelevant**:
both message variants match the same trigger. That's why the preset phrases are short fragments
and not the full sentence.

### 5. The matched prayer is HELD

A Chat match **holds** the matched overhead prayer: it switches your overhead on and keeps it
there until the *next* Chat trigger replaces it. A new sphere announcement takes over the hold;
an Olm basic keeps the same attack style for several hits, so holding (instead of a single
one-tick flip) is exactly what you want. When you press **STOP**, the held overhead is turned
back off so you aren't left with stale protection lit.

### Flow

```
Olm chat line ─▶ phrase appears in the line?  ──no──▶ ignore
                    │ yes
                    ▼
        switch overhead to the matched prayer
                    │
                    ▼
        HOLD it until the next phrase matches
```

---

## The exported script format

Use the **Export** button to save the group to a `.json` file (and **Import** to load one). In an
exported file each Prayer Trigger: Chat task carries these keys (the Olm group is just a group
entry whose `childScripts` holds the three tasks):

```json
{
  "name": "Olm Basics",
  "isGroup": true,
  "childScripts": [
    {
      "name": "Olm: Melee sphere",
      "type": "PRAYER_TRIGGER_CHAT",
      "triggerPhrase": "sphere of aggression",
      "triggerPrayer": "PROTECT_FROM_MELEE",
      "triggerPriority": 1
    },
    {
      "name": "Olm: Magic sphere",
      "type": "PRAYER_TRIGGER_CHAT",
      "triggerPhrase": "sphere of magical power",
      "triggerPrayer": "PROTECT_FROM_MAGIC",
      "triggerPriority": 1
    },
    {
      "name": "Olm: Range sphere",
      "type": "PRAYER_TRIGGER_CHAT",
      "triggerPhrase": "sphere of accuracy and dexterity",
      "triggerPrayer": "PROTECT_FROM_MISSILES",
      "triggerPriority": 1
    }
  ]
}
```

(An exported file also writes the other standard keys at their defaults; the keys above are the
ones that matter for a Chat trigger.)

---

## Editing / extending the preset

Every field is editable in the side panel — double-click a task to open the editor.

- **Change a prayer:** pick a different option from the **Trigger prayer** dropdown
  (`PROTECT_FROM_MELEE`, `PROTECT_FROM_MAGIC`, `PROTECT_FROM_MISSILES`).
- **Tune a phrase:** keep it to the *distinguishing fragment*, lowercased. Shorter, unique
  substrings are more robust to punctuation and the "sapped" suffix.
- **Add a phrase:** add another **Prayer Trigger: Chat** task. Any boss that announces its attack
  in chat can be prayed this way — no IDs needed.
- **Remove one:** delete the task you don't want.

Changes take effect on the next **START** (triggers re-arm from the current task list each time
you start).

---

## Extending to Olm specials / other bosses

The Chat trigger is the easiest because the phrase is human-readable. For attacks with **no chat
message** (Olm's crystal burst, lightning walk, acid pools, hand specials, or any other boss),
use the other two reactive trigger types. Both key off numeric in-game IDs that you must
**look up in-client first** — they are not shipped in the preset.

### Prayer Trigger: Projectile (pre-pray off an incoming projectile)

Pre-prays the overhead based on an **incoming projectile id**, timed off the projectile's flight
so the prayer is up before the hit lands.

```
Type: Prayer Trigger: Projectile
  Projectile id:   <projectile id>          (look up in-client)
  Trigger prayer:  PROTECT_FROM_MISSILES
  Lead ticks:      1                         (pre-pray lead; 1 = active on the impact tick)
  Priority:        1                         (lower wins if two triggers overlap)
```

This is the right choice for projectile-based specials because the timing is exact.

### Prayer Trigger: Animation (pray off an NPC attack animation)

Switches the prayer a fixed number of ticks **after** a matching NPC **animation id** plays — for
melee/contact attacks that have no projectile and no chat line.

```
Type: Prayer Trigger: Animation
  NPC anim id:      <npc animation id>       (look up in-client)
  Trigger prayer:   PROTECT_FROM_MELEE
  Anim->hit ticks:  1                         (animation -> hit delay, in ticks)
  Priority:         1
```

> Warning: Projectile and Animation triggers will not match until you fill in the correct
> **Projectile id** / **NPC anim id**. A blank or wrong id (the field starts at `-1`) never fires.
> Look up the real projectile/animation id in a live client first.

---

## Notes

- **Reacts to live events, not a timer.** The trigger reacts to the actual chat line (or
  projectile / animation); it does not guess a rotation by counting ticks.
- **Pre-empts the task order.** Prayer switches happen the moment the event is seen, regardless of
  which task is currently running and regardless of break-handler pauses.
- **No spamming.** A trigger won't re-flip a prayer that's already on; it only changes the
  overhead when the required protection actually changes.
- **No control-flow needed.** Olm Basics is purely reactive — you don't need **Loop**, variables,
  or **Stop Script** for the prayer rotation. (Those exist for sequential task scripts — see the
  Variables and Features docs.)
