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
| `BlzSetUnitBaseDamage`, `BlzSetUnitDiceNumber`, `BlzSetUnitDiceSides`, `BlzSetUnitAttackCooldown`, `BlzSetUnitArmor`, `BlzSetUnitMaxHP`, `BlzSetUnitMaxMana`, `BlzGetUnitMaxHP`, `BlzGetUnitMaxMana`, `BlzGetUnitDiceNumber`, `BlzGetUnitDiceSides`, `BlzGetUnitSkin` | works | Raising max mana or max HP of a living unit leaves the current value unchanged, as measured; lowering clamps. |
| Getters after `BlzSetUnitArmor`, `BlzSetUnitMaxHP`, `BlzSetUnitMaxMana` while buffed | bugfix | Return exactly the value set, also while buffs are active. |
| Raising max HP of a dead unit (`BlzSetUnitMaxHP`, `SetUnitState` max life, hero stats, life items, HP upgrades) | bugfix | The unit stays dead and no second death event fires, as measured; only revive paths bring it back. |
| Life of a dead unit after a max-life raise | deliberate | Reforged reads a nonzero corpse life (still dead), as measured; WarMash reads 0. |
| `SetUnitState(u, UNIT_STATE_MAX_LIFE, x)` | bugfix | Ignored, as measured in Reforged; use `BlzSetUnitMaxHP`. `UNIT_STATE_MAX_MANA` is ignored too. |
| Max life changes from `SetHeroStr`, strength gain, `AIlf`/`AIl2`, HP upgrades, Avatar | bugfix | Strength changes move life by the change; `AIlf`/`AIl2` (Periapt) keep the life ratio; Avatar end only clamps life to the new max, as measured; upgrades inferred. |
| `UNIT_STATE_MAX_MANA` via `SetUnitState` | deliberate | Ignored, as in Reforged. |
| Mana change from `SetHeroInt` or intelligence gain | bugfix | Mana moves by the max-mana change, never below 0, as measured. |
| Max life drops from item drop, Avatar end, strength loss | unverified | Life drops by the change, at least 1 for a living unit; not measured. |
| Life bonuses while max-HP buffs or a `BlzSetUnitMaxHP` value are active | bugfix | No longer counted twice. |
| Max life or mana change from an old maximum of 0 | bugfix | Keeps the current value clamped to 0 and the new maximum instead of becoming NaN. |
| `BlzSetUnitMaxHP` with 0 | bugfix | Kills the unit, as measured; `SetWidgetLife` does not revive it. A negative value also kills (unverified). |
| `BlzSetUnitMaxHP` with 0 on a unit with Reincarnation | deliberate | Reforged triggers the Reincarnation, as measured; WarMash skips it and the unit stays dead. Ankh behaves the same in both. |
| Reincarnation delay ending with max life 0 | deliberate | Reforged leaves a zombie alive at life 0 without events, as measured; WarMash turns it into a real death. |
| Reincarnation false death | works | Fires no death event, bounty or XP, as measured. |
| Timed buffs on a unit revived by a death-replacement ability | bugfix | Slow, Bloodlust, stun and other timed buffs are cleared at death, as measured; timed life stays. |
| Summoned unit with Reincarnation | gap | It loses its timed life on reincarnating and never expires; Warcraft III behavior not confirmed. |
| `RemoveUnit` on a unit in a Reincarnation delay | deliberate | The unit stays removed with no revive or death event; Reforged keeps its typed unit count, WarMash releases it. |
| Unit read after `RemoveUnit` | gap | It still reads its old type and position; Reforged reads type 0 and position 0,0, as measured. |
| `RemoveUnit` called from a spell `EFFECT` condition | unverified | A Java-base spell's effect still runs; Reforged not measured. |
| `RemoveUnit` on a dead hero, then `ReviveHero` in the same tick | deliberate | Reforged revives it, as measured; WarMash returns false and revives nothing. |
| `RemoveUnit` on a hero that died normally | unverified | Releases its tech-tree count once (`GetPlayerTechCount`, hero training limit); before, the dead removed hero counted forever. |
| `RemoveUnit` on a unit that is already removed or whose corpse has decayed | unverified | Does nothing more: the tech-tree count is released once, and no second removal is logged. |
| `RemoveUnit` on a live haunted or entangled gold mine | bugfix | The parent mine is restored at the next unit-add step, so `IsUnitHidden(parent)` is still true inside the same trigger; Reforged not measured. |
| `RemoveUnit` on a hero queued at an altar | unverified | The queue entry is cancelled and gold, lumber and food are refunded to the altar owner. |
| `BlzSetUnitMaxMana` with 0 or less | unverified | Max mana is clamped to at least 0. |
| `BlzSetUnitRealField` | partial | Sight radius, HP and mana regeneration, acquisition range, turn rate, cast point and cast backswing work; other fields return false. |
| `BlzSetUnitRealField(UNIT_RF_CAST_POINT)`, `UNIT_RF_CAST_BACK_SWING` | works | Set per unit and used by casts; negative values become 0; kept across a morph (not confirmed in Reforged). |
| `GetUnitCastPoint`, `GetUnitCastBackswingPoint` | bugfix | Return the per-unit value, not only the unit type's. |
| `BlzGetUnitIntegerField` | partial | Only primary attribute and the four tint fields; other fields return 0. |
| `BlzSetUnitIntegerField` | partial | Only the classification field works; it reads back as written, and other fields return false. |
| `UNIT_IF_UNIT_CLASSIFICATION` writes | bugfix | Bit 128 is only Ancient and never blocks targeting; `IsUnitType(ANCIENT)` and ancient target lists read it, as measured. |
| `UNIT_IF_UNIT_CLASSIFICATION` write replaces the field | bugfix | Writing 128 or 0 clears `IsUnitType(MECHANICAL)`, and 0 clears the type's own Ancient, as measured. |
| `UnitAddType`, `UnitRemoveType` after a classification write | bugfix | Set and clear the mechanical, undead and ancient bits (128 plus mechanical reads 136), as measured. |
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
| `GetUnitCurrentOrder` after `ReviveHero`, altar revive, Resurrection or Reincarnation | unverified | Reads 0; before, the order of the cast the unit died in. |
| `ReviveHero` on a hero still in a stun state | unverified | Stunned at once (permanent stun buff or `UNIT_TYPE_STUNNED`); before, unstunned until the next stun change. |
| `ReviveHero`, `ReviveHeroLoc` | works | Fire no revive events; refused when the food cap would be exceeded. |
| Altar revive cancel of a paid entry | unverified | The refunded food goes straight to the hero; no waiting train item takes it. |
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
| `BlzSetUnitSkin`, `BlzCreateUnitWithSkin` | works | Change model, portrait, tint and sounds; the animation, a script vertex colour (as measured) and the script scale reset; `SetUnitScale` afterwards is absolute. |
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
| `UnitSetUpgradeProgress`, `UnitSetConstructionProgress` | works | Percent is clamped 0-100; each acts only on its own case, and training or researching units are ignored, as measured. |
