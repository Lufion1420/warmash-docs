---
title: Units and heroes
nav_order: 1
parent: Differences from Warcraft III
---

# Units and heroes
{: .no_toc }

Everyday unit and hero natives work. Several `Blz*Field` natives support only some fields.

### Stats and fields

| Native | Status | What differs |
|---|---|---|
| `BlzGetUnitBaseDamage` | unverified | Weapon damage plus the hero's primary-attribute bonus; upgrades excluded and weapon index 0-based, as measured. |
| Attack dice upgrade (`BlzGetUnitDiceNumber`, `BlzGetUnitDiceSides`) | works | The upgrade raises the number of dice only, as measured. |
| `BlzGetUnitAttackCooldown`, `BlzIsUnitInvulnerable` | works | The cooldown is the base value without attack-speed bonuses. |
| `BlzGetUnitRealField` | partial | 17 fields (see Details); other fields return 0 with one log line. |
| `BlzGetUnitRealField(UNIT_RF_SELECTION_SCALE)` | bugfix | Returns the type's selection scale (footman 1.0, knight 1.4), as measured; was 0. |
| Hero armor from agility | unverified | Damage reduction uses the fractional armor; the display still rounds. |
| `BlzSetUnitBaseDamage`, `BlzSetUnitDiceNumber`, `BlzSetUnitDiceSides`, `BlzSetUnitAttackCooldown`, `BlzSetUnitArmor`, `BlzSetUnitMaxHP`, `BlzSetUnitMaxMana`, `BlzGetUnitMaxHP`, `BlzGetUnitMaxMana`, `BlzGetUnitDiceNumber`, `BlzGetUnitDiceSides`, `BlzGetUnitSkin` | works | Raising max HP or mana also raises the current value; lowering clamps it. |
| Getters after `BlzSetUnitArmor`, `BlzSetUnitMaxHP`, `BlzSetUnitMaxMana` while buffed | bugfix | Return exactly the value set, also while buffs are active. |
| `BlzSetUnitMaxHP` with 0 | bugfix | Kills the unit, as measured; `SetWidgetLife` does not revive it. A negative value also kills (unverified). |
| `BlzSetUnitMaxHP` with 0 on a unit with Reincarnation or Ankh | unverified | The replacement is skipped and the unit stays dead; during a Reincarnation delay the unit dies for real when it ends, without killer. |
| `RemoveUnit` on a unit in a Reincarnation delay | unverified | The unit stays removed: no revive, no death event, tech-tree count released. |
| `RemoveUnit`, then `ReviveHero`, `ReviveHeroLoc` or `ResurrectUnit` in the same tick | unverified | Returns false and revives nothing. |
| `RemoveUnit` on a hero that died normally | unverified | Releases its tech-tree count once (`GetPlayerTechCount`, hero training limit); before, the dead removed hero counted forever. |
| `RemoveUnit` on a unit that is already removed or whose corpse has decayed | unverified | Does nothing more: the tech-tree count is released once, and no second removal is logged. |
| `RemoveUnit` on a hero queued at an altar | unverified | The queue entry is cancelled and gold, lumber and food are refunded to the altar owner. |
| `BlzSetUnitMaxMana` with 0 or less | unverified | Max mana is clamped to at least 0. |
| `BlzSetUnitRealField` | partial | Sight radius, HP and mana regeneration, acquisition range and turn rate work; other fields return false. |
| `BlzGetUnitIntegerField` | partial | Only primary attribute and the four tint fields; other fields return 0. |
| `BlzSetUnitIntegerField` | partial | Only the classification field works; it reads back as written, and other fields return false. |
| `UNIT_IF_UNIT_CLASSIFICATION` writes | bugfix | Bit 128 is only Ancient and never blocks targeting; `IsUnitType(ANCIENT)` and ancient target lists read it, as measured. |
| `UNIT_IF_UNIT_CLASSIFICATION` write replaces the field | bugfix | Writing 128 or 0 clears `IsUnitType(MECHANICAL)`, and 0 clears the type's own Ancient, as measured. |
| `BlzGetUnitBooleanField` | partial | 12 fields, read from the unit type, not the unit; `BlzSetUnitBooleanField` is missing. |
| `SetUnitState` life and mana | bugfix | Clamped to the maximum; lowering max life does not lower current life. |
| `BlzGetUnitCollisionSize`, `GetDestructableTypeId`, `GetDestructableOccluderHeight` | works | Implemented. |
| `IsUnitInRange`, `IsUnitInRangeLoc`, `IsUnitInRangeXY` | bugfix | Real range checks from the first unit's edge; `IsUnitInRangeXY` was always false. |
| `IsUnitPaused`, `IsUnitInTransport`, `IsUnitLoaded`, `GetHeroSkillPoints`, `UnitInventorySize`, `IsPointBlighted` | works | `IsUnitPaused` is true after `PauseUnit`, false under `BlzPauseUnitEx`, as measured. |
| `IsUnitType` with `UNIT_TYPE_POISONED`, `UNIT_TYPE_PLAGUED` | gap | Always false: these effects do not exist in WarMash yet. |
| `IsUnitType(unit, UNIT_TYPE_POLYMORPHED)` | works | True while the unit is under Polymorph. |
| Null unit in `BlzGetUnitArmor`, `GetUnitDefense`, `GetUnitCastPoint`, `GetUnitCastBackswingPoint`, `UnitGetTemporaryDefenseBonus`; `GetSpellTargetUnit`, `GetSpellTargetLoc` outside a spell | bugfix | Return 0 or nil instead of a script error. |
| `IsUnitIllusion` | gap | Always false: WarMash cannot create illusions yet. |
| `SetWidgetLife`, `SetUnitState(UNIT_STATE_LIFE)` and heals on a killed unit | bugfix | Ignored: a killed unit stays at 0 life until a real revive (was a crash). |

Details:
- `BlzGetUnitRealField` fields: `uhpr`, `umpr`, `udfc`, `usir`, `uacq`, `ucpt`, `ucbs`, `ustp`, `uagp`, `uinp`, `uhpc`, `umpc`, `umvc`, `umvr`, `ufyh`, `usca`, `ussc`. Armor is unrounded; `usca` ignores `SetUnitScale`; sight radius excludes ability bonuses.
- `BlzGetUnitBooleanField` fields: `urai`, `udec`, `ubdg`, `ulos`, `unbm`, `uhhb`, `uhhm`, `uhhd`, `uhom`, `uscb`, `usew`, `ushr`.
- Weapon index of the `BlzSet/GetUnit*` damage natives is 0 or 1; out of range a setter does nothing and a getter returns 0.

### Heroes

| Native | Status | What differs |
|---|---|---|
| `BlzGetHeroPrimaryStat` (hidden native) | bugfix | Returns STR 0, AGI 1, INT 2 as in Reforged; 0 for a non-hero. |
| `GetHeroStr`, `GetHeroAgi`, `GetHeroInt` | works | Values match a Reforged run on one hero. |
| `SetHeroStr`, `SetHeroAgi`, `SetHeroInt` | bugfix | `GetHeroStr(false)` and the others return the value set; the non-permanent form does nothing. |
| `GetHeroLevel`, `GetUnitLevel`, `GetHeroXP` | works | Implemented. |
| `GetHeroProperName` | works | Returns nil for a non-hero. |
| `BlzSetHeroProperName` | works | The hero-bar tooltip and revive button show the new name only after their next rebuild. |
| Hero proper name at creation | gap | Stock heroes always get their first proper name, not a random one. |
| `ReviveHero`, `ReviveHeroLoc` | works | Fire no revive events; refused when the food cap would be exceeded. |
| Altar revive gold, lumber and time | bugfix | Capped by the `HeroMaxRevive*` gameplay constants. |
| `UnitModifySkillPoints` | unverified | Capped at what the hero can still learn; the real native's documented bugs are not copied. |
| `EVENT_UNIT_HERO_LEVEL`, `EVENT_PLAYER_HERO_LEVEL`, `GetLevelingUnit` | bugfix | One event pair per `SetHeroLevel`, `AddHeroXP` or `SetHeroXP` call with the final level, as in Reforged; unit list then player list; from a kill they run before the death events. |
| Hero level-up art | deliberate | One effect per gain call, not one per level gained. |
| Hero skill points | unverified | The "+" button shows only when the hero has points and something to learn. |

### Creation, pause, Locust and animation

| Native | Status | What differs |
|---|---|---|
| `CreateUnit`, `CreateUnitAtLoc`, `BlzCreateUnitWithSkin` in a timer or trigger callback | deliberate | The unit acts on the next tick; projectiles created in a callback still start one phase late. |
| `CreateUnit` family and `CreateCorpse`: start animation | deliberate | Script-created units start in Stand without Birth; engine-made units still play Birth. |
| `BlzPauseUnitEx` | gap | Own counter, separate from `PauseUnit`; command buttons stay visible but unusable. |
| `PauseUnit`, `BlzPauseUnitEx`: animation | unverified | The walk animation switches to stand; animations set by script are kept. |
| `SetUnitAnimationToWalk` (WarMash only) | deliberate | Counts as an engine walk, so a pause switches it to stand. |
| `PauseUnit`, `BlzPauseUnitEx`: end of pause | unverified | An idle unit looping a `SetUnitAnimationByIndex` animation returns to stand. |
| Locust (`UnitAddAbility` / `UnitRemoveAbility` with `'Aloc'`) | deliberate | Full Locust effects while present; removing it restores the unit at once (WC3 keeps them). |
| `SetUnitTimeScale` | unverified | Kept across animation changes. |
| `SetUnitAnimationByIndex` | unverified | A looping animation replays until any other animation request. |
| `BlzSetUnitSkin`, `BlzCreateUnitWithSkin` | works | Change model, portrait, tint and sounds; the animation and a script vertex colour reset. |
| `SetUnitVertexColor` | bugfix | Components clamp to 0..255, as measured. |
| `SetUnitScale` with a scale buff | deliberate | Drawn scale is script scale times the buff bonus; the script scale survives the buff. |
| `GetChangingUnitPrevOwner`, `SetUnitFlyHeight` | works | Both work now; see their rows in Players and Movement. |

Details:
- Locust: not selectable, targetable or enumerated, no bars, others walk through it; script damage still lands. Its own movement still collides (gap).

### Guard positions and creeps

Idle units walk home after a chase they started themselves. Player and script orders are never pulled back.

| Native | Status | What differs |
|---|---|---|
| Idle guard position and leash | gap | Returns home 5 s after passing 600 without damage; rules beyond the game constants are inferred. |
| Script move (`SetUnitX/Y`, `SetUnitPosition`) of more than 128 | deliberate | Moves the unit's guard home to the new spot. |
| `RemoveGuardPosition`, `RecycleGuardPosition`, `SetUnitCreepGuard`, `RemoveAllGuardPositions` | unverified | Implemented with inferred meanings. |
| Auto-acquire target choice | unverified | Highest `prio` first, then the nearest. |
| `GetUnitCurrentOrder` during auto-acquired attacks | unverified | Reads 0 (Hive, unmeasured), not attack. |
| `GetUnitCurrentOrder` after a stop, or after the running ability is removed or disabled mid-cast | bugfix | Reads 0 (Hive, unmeasured); was the old order. `SPELL_ENDCAST` handlers still read the cast's order. |
| `GetUnitCurrentOrder` for a move blocked by pathing or units | unverified | Keeps reading move while the unit still tries, reads 0 once it gives up (16 to 58 s); an arrived move reads 0. |
| Chase target turns invulnerable, hidden or Locust | deliberate | The guard walks home at once. |
| Re-acquire after a chase | works | Respects auto-attack-disabling buffs; a script order inside an autocast clears the leash. |
| Creep return (`MaxGuardDistance`, `GuardReturnTime`) | bugfix | Walks home 5 s after first passing 1000 from home, as measured. |
| `UnitAddSleep`, `UnitAddSleepPerm`, `UnitCanSleep`, `UnitCanSleepPerm`, `UnitWakeUp`, `UnitIgnoreAlarm` | gap | Not implemented: creeps never sleep or call for help. |

### Visibility and detection

| Native | Status | What differs |
|---|---|---|
| `IsUnitVisible`, `IsUnitFogged`, `IsUnitMasked`, `IsVisibleToPlayer`, `IsFoggedToPlayer`, `IsMaskedToPlayer`, `IsLocationVisibleToPlayer`, `IsLocationFoggedToPlayer`, `IsLocationMaskedToPlayer` | works | A point outside the map counts as masked. |
| `IsUnitDetected` | bugfix | True for any unit inside a detector of the player, as measured. |
| `IsUnitInvisible` | deliberate | True only for an invisible unit the player cannot see; plain fog gives false (1.29 gave true). |
