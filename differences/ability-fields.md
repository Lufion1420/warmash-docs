---
title: Ability fields and levels
nav_order: 12
parent: Differences from Warcraft III
---

# Ability fields and levels
{: .no_toc }

Field writes change one ability instance, and most act only at the next level refresh. Many UI and type fields are stubs: WarMash stores the value and nothing uses it.

```lua
local id = FourCC("A000")
local abil = BlzGetUnitAbility(hero, id)
BlzSetAbilityRealLevelField(abil, ABILITY_RLF_COOLDOWN, 0, 5.0) -- 0 is level 1
SetUnitAbilityLevel(hero, id, GetUnitAbilityLevel(hero, id))    -- refresh
```

### Field natives

| Native | Status | What differs |
|---|---|---|
| `BlzGetAbility*Field`, `BlzSetAbility*Field` (Boolean, Integer, Real, String) | partial | Per ability instance; getters return your value first, then object data. Some setters are stubs. |
| `BlzGetAbilityBooleanLevelField`, `BlzSetAbilityBooleanLevelField` | unverified | Per instance, 0-based level; safe to call, while Reforged crashed on a built Spell Book, as measured. |
| `BlzSetAbilityStringLevelField` | partial | Per instance, 0-based level; tooltip and art fields are stubs. |
| `BlzGet`, `BlzSet`, `BlzAdd`, `BlzRemoveAbility*LevelArrayField` | unverified | 0-based index; Add appends duplicates, Remove takes the first match, Set cannot grow the list. |
| New level and array field writes | unverified | Act at the next level change or `Inc`, `Dec`, `SetUnitAbilityLevel` refresh. |
| Non-level writes the engine reads (`'amsp'`, `'arlv'`) | unverified | Also wait for the next level refresh. |
| New field natives: foreign or unknown field, bad level | unverified | Refused with one log line: getters return 0, false or "", setters false. |
| New field natives: wrong field shape | unverified | Per-level field through a non-level native, or single-value field through an array native, is refused. |
| Setters for `ABILITY_BF_HERO_ABILITY`, `ABILITY_SF_NAME` | works | Return false; `common.j` marks both as get only. |
| Field setters on a removed ability's handle | unverified | Return true; nothing changes in game. |

Details:
- "New field natives" are all ability field natives except the five older ones in the last table.
- Array index out of range: getters return 0, false or "", Set returns false.

### Stub fields

| Native | Status | What differs |
|---|---|---|
| Setters for button positions, icons, sounds, missile arc, effect art and tooltip fields | stub | The value is stored and read back; nothing in game uses it. |
| Setters for `'alev'`, `'apri'`, `'alsk'`, `'aite'`, `'achd'`, `'atac'`, `'acac'` | stub | The value is stored and read back; nothing in game uses it. |
| `BlzSetAbilityIcon` | works | Sets the normal icon on all levels; the disabled icon is unchanged. |

### Older level field natives

| Native | Status | What differs |
|---|---|---|
| `BlzGetAbilityRealLevelField`, `BlzGetAbilityIntegerLevelField` | works | Level is 0-based; a script-set value is returned first. |
| `BlzSetAbilityIntegerLevelField`, `BlzSetAbilityRealLevelField` | partial | Take effect on the next level change (Channel at once); some fields stay stored-only. |
| `BlzSetAbilityIntegerLevelField`, `BlzSetAbilityRealLevelField`: huge levels | bugfix | On list fields, list index 10000 or higher, past the type's list, reads back but is not applied. |
| `BlzGetAbility*LevelField`, `BlzSetAbility*LevelField` for Integer and Real | unverified | Accept fields of other ability classes and any level from 0, unlike the newer natives. |
| `BlzGetAbilityStringLevelField` | partial | Script-set value first, then object data for art, tooltip and the ability's own string fields. |
| `BlzGetAbilityStringLevelField`: other classes' fields | unverified | Return "" for any level. |
| Tooltip text from `BlzGetAbilityStringLevelField` | unverified | Returned raw, placeholders unresolved. |
| `BlzSetAbilityRealLevelField(ABILITY_RLF_CASTING_TIME)` on ability-builder spells | bugfix | Read on every cast; no level refresh needed. |

Details:
- `BlzSetAbility*LevelField` stays stored-only for item life and mana bonuses (`AIml`, `AImm`; deliberate), abilities without an engine class, and abilities that read a value only when added.

### Level changes

| Native | Status | What differs |
|---|---|---|
| `IncUnitAbilityLevel`, `DecUnitAbilityLevel`, `SetUnitAbilityLevel` at an unchanged level | unverified | Re-apply ability data only after a script field write; stub writes by newer natives do not count. |
| `IncUnitAbilityLevel`, `DecUnitAbilityLevel`, `SetUnitAbilityLevel` inside that ability's own level change | unverified | Applied right after the running change; level getters already read the requested level. |
| `DecUnitAbilityLevel` | works | Lowers by 1, not below 1; returns 0 without the ability. |
| `IncUnitAbilityLevel`, `SetUnitAbilityLevel`, `SetAbilityLevel` out of range | bugfix | Clamped to the ability's levels instead of a script error. |
| `GetUnitAbilityLevel` | bugfix | Returns the real level, 0 without the ability (was 1). |
| Level change inside the ability's own `SPELL_EFFECT` handler | bugfix | The cooldown just started takes the new level's value. |

Details:
- Level change inside a level change: at most 4 follow-up passes; dropped if the ability or unit is removed; still applied on a dead unit.
