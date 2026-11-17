---
title: Differences from Warcraft III
nav_order: 2
---

# Differences from Warcraft III
{: .no_toc }

What a map script observes in WarMash that differs from real Warcraft III, native by native. Compared with Reforged 3.0.1 for Lua maps and 1.29 for JASS maps.

A native that is not listed is either believed to match or is missing; see [Native coverage](#native-coverage-and-constants). The status words are explained on the [home page](./).

**Topics:** [Units and heroes](#units-and-heroes) · [Abilities and spells](#abilities-and-spells) · [Orders](#orders) · [Damage events](#damage-and-attack-events) · [Buffs](#buffs) · [Movement and terrain](#movement-pathing-and-terrain) · [Regions and groups](#regions-and-unit-groups) · [Players and game state](#players-and-game-state) · [Timers and triggers](#timers-triggers-and-hashtables) · [Selection and input](#selection-and-input) · [Items and shops](#items-and-shops) · [UI frames](#ui-frames-and-command-card) · [Camera](#camera) · [Effects and sound](#special-effects-and-sound) · [Lua runtime](#lua-runtime) · [JASS runtime](#jass-runtime) · [Map loading](#map-loading-and-data) · [Native coverage](#native-coverage-and-constants)

"Measured" means checked by a scripted run in real Reforged 3.0.1.

## Units and heroes

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
| `BlzSetUnitMaxHP`, `BlzSetUnitMaxMana` with 0 or less | unverified | Max HP is clamped to at least 1, max mana to at least 0. |
| `BlzSetUnitRealField` | partial | Sight radius, HP and mana regeneration, acquisition range and turn rate work; other fields return false. |
| `BlzGetUnitIntegerField` | partial | Only primary attribute and the four tint fields; other fields return 0. |
| `BlzSetUnitIntegerField` | stored-only | Only the classification field is kept, and it does not change targeting; other fields return false. |
| `IsUnitType(unit, UNIT_TYPE_ANCIENT)` | partial | Reads bit 128 of the classification set by `BlzSetUnitIntegerField`. |
| `BlzGetUnitBooleanField` | partial | 12 fields, read from the unit type, not the unit; `BlzSetUnitBooleanField` is missing. |
| `SetUnitState` life and mana | bugfix | Clamped to the maximum; lowering max life does not lower current life. |
| `BlzGetUnitCollisionSize`, `GetDestructableTypeId`, `GetDestructableOccluderHeight` | works | Implemented. |
| `IsUnitInRange`, `IsUnitInRangeLoc`, `IsUnitInRangeXY` | bugfix | Real range checks from the first unit's edge; `IsUnitInRangeXY` was always false. |
| `IsUnitPaused`, `IsUnitInTransport`, `IsUnitLoaded`, `GetHeroSkillPoints`, `UnitInventorySize`, `IsPointBlighted` | works | `IsUnitPaused` counts `PauseUnit` only, not `BlzPauseUnitEx`. |
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
| `GetUnitCurrentOrder` during auto-acquired attacks | unverified | Reads 0 or the last real order, not attack. |
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

## Abilities and spells

Channel-based (`ANcl`) spells have mana, cooldown, the five spell events and settable level fields. Disable and hide differ from Reforged on purpose:

```lua
BlzUnitDisableAbility(hero, FourCC("A000"), true, true)
BlzUnitDisableAbility(hero, FourCC("A000"), false, true)
-- WarMash: usable again. Reforged: still refused until (false, false).

BlzUnitHideAbility(hero, FourCC("A000"), true)
BlzUnitHideAbility(hero, FourCC("A000"), true)
BlzUnitHideAbility(hero, FourCC("A000"), false)
-- WarMash: shown again. Reforged: still hidden, one more false call needed.
```

### Disable, hide, levels and fields

| Native | Status | What differs |
|---|---|---|
| `BlzUnitDisableAbility`: counting | bugfix | A counter per ability as in Reforged: N disables need N enables, as measured. |
| `BlzUnitDisableAbility(..., false, hideUI)`: which disable an enable closes | deliberate | Closes one with the same `hideUI` first, else any other; Reforged needs `(false, false)`. |
| `BlzUnitHideAbility`: counting | deliberate | An on/off switch: one `false` undoes any number of `true` calls; Reforged counts. |
| Ability hidden by `BlzUnitHideAbility`: orders | bugfix | Refuses orders like a disabled ability, as measured; a running cast finishes. |
| `ConvertAbilityDisableType(7)` | deliberate | Valid in WarMash (disable type POLYMORPH); out of range in WC3. |
| `SetAbilityDisabled`, `SetAbilityIconShowing` (WarMash only) | deliberate | Script disable and icon switches; `SetAbilityIconShowing` does not refresh the command card by itself. |
| `BlzGetUnitAbilityByIndex` | unverified | Index 0 is the newest ability; abilities the engine adds itself are skipped. |
| `BlzGetAbilityStringLevelField` | partial | 14 string fields, 0-based level; other fields return "". |
| Tooltip text from `BlzGetAbilityStringLevelField` | unverified | Returned raw, placeholders unresolved. |
| `BlzSetAbilityIcon` | works | Sets the normal icon on all levels; the disabled icon is unchanged. |
| `BlzGetAbilityRealLevelField`, `BlzGetAbilityIntegerLevelField` | works | Every field is known; level is 0-based; a script-set value is returned first. |
| `BlzSetAbilityIntegerLevelField`, `BlzSetAbilityRealLevelField` | partial | Take effect on the next level change (Channel at once); some fields stay stored-only. |
| `IncUnitAbilityLevel`, `DecUnitAbilityLevel`, `SetUnitAbilityLevel` with script-set fields | unverified | At an unchanged level they re-apply the data only when a script set fields. |
| `DecUnitAbilityLevel` | works | Lowers by 1, not below 1; returns 0 without the ability. |
| `IncUnitAbilityLevel`, `SetUnitAbilityLevel`, `SetAbilityLevel` out of range | bugfix | Clamped to the ability's levels instead of a script error. |
| `GetUnitAbilityLevel` | bugfix | Returns the real level, 0 without the ability (was 1). |
| Level change inside the ability's own `SPELL_EFFECT` handler | bugfix | The cooldown just started takes the new level's value. |
| `UnitAddAbility` with a null unit or a duplicate | bugfix | Returns false without an error. |
| `GetSpellAbility`, `GetSpellTargetDestructable`, `GetSpellTargetItem`, `BlzGetUnitAbilityCooldown`, `BlzGetUnitAbilityManaCost`, `BlzGetAbilityCooldown`, `UnitResetCooldown`, `GetWidgetLife`, `SetWidgetLife` | works | `UnitResetCooldown` also ends item cooldowns; `SetWidgetLife` is capped at max life. |
| `BlzSetUnitAbilityCooldown` | partial | Works for Channel and engine spells; stored-only for other ability kinds. |
| `BlzEndUnitAbilityCooldown` | bugfix | Also ends Channel cooldowns. |

Details:
- `BlzSetAbility*LevelField` stays stored-only for item life and mana bonuses (`AIml`, `AImm`; deliberate), abilities without an engine class, and abilities that read a value only when added.

### Channel and spell events

| Native | Status | What differs |
|---|---|---|
| Channel (`ANcl`) | works | Per-level fields `Ncl1` to `Ncl6`, mana, cooldown and all five spell events. |
| Channel with follow-through time 0 | unverified | Treated as an instant cast (0.001 s). |
| Channel: `SPELL_FINISH` / `SPELL_ENDCAST` timing | bugfix | Fire in the tick the follow-through ends, after that tick's timers, as measured. |
| Ability-builder (JSON) channels: end events | deliberate | Same order as Channel by analogy; a cut channel fires ENDCAST only. |
| `BlzPauseUnitEx` during a Channel cast | bugfix | Ends the cast and re-issues the order at the unpause, as measured. |
| `BlzPauseUnitEx` inside the EFFECT of an instant Channel | deliberate | Not re-issued, so it casts once as in Reforged; FINISH is not fired. |
| Channel with blank `Animnames` | deliberate | No cast animation plays. |
| Channel: stun before EFFECT | unverified | The cast ends; WC3 reportedly casts again after the stun. |
| Channel: order given before EFFECT | unverified | Interrupts the cast; not queued. |
| Channel: `Ncl5` (disable other abilities) | unverified | Greys only spell and item abilities. |
| Channel: target art, `RemoveUnit` | gap | No target art; removing the caster fires ENDCAST. |
| `GetTriggerPlayer` in spell events | gap | Returns nil. |
| `GetSpellTargetX`, `GetSpellTargetY`, `GetSpellTargetLoc` | unverified | The target point, else the target's position, else 0. |
| Some engine spells and the item heal and mana abilities | gap | Fire no spell events. |
| `BlzSetUnitBooleanField` | gap | Not implemented; it is the last missing native that the reference map's spell code calls. |

Details:
- Channel event order: CHANNEL, CAST after the casting time, EFFECT (mana and cooldown taken), then FINISH and ENDCAST. An interrupted cast fires ENDCAST only.
- `GetUnitCurrentOrder` reads 851973 between a `BlzPauseUnitEx` pause and the re-issue.

### Stock ability classes

| Native | Status | What differs |
|---|---|---|
| Storm Bolt (`AHtb`), Attribute Bonus (`Aamk`) | works | A missile with speed 0 or no art arrives at once. |
| Cripple (`Acri`), Drunken Haze (`ANdh`) | works | Implemented; Drunken Haze `Nsi1` is a bit mask (1 melee, 2 ranged, 4 special, 8 spells). |
| Drunken Haze overlapping | gap | Two from the same caster: the disarm ends when the first expires. |
| Cleave (`ANca`) | unverified | Melee only; each unit in the area takes the full factor and no attack event fires. |
| Item sight bonus (`AIsi`) | unverified | Raises sight while carried; the sight field reads the value without it. |
| Spell damage reduction (`AIsr`) | unverified | Reduces `ATTACK_TYPE_NORMAL` and `ATTACK_TYPE_MAGIC` damage. |
| `Aclf`, `Aspb`, `Aprg`, `AEer`, `Assk` | gap | Do nothing: no engine class yet. |
| Aerial Shackles (`Amls`) | bugfix | The channel ends after its duration and fires ENDCAST. |
| Ensnare (`Aens`) | partial | Target cannot move but may attack and cast; hits magic-immune, refuses ethereal, as measured. |
| Bloodlust (`Ablo`) | works | Attack rate, speed and scale bonus with autocast; speed reads 337.5 from 270, as measured. |
| Unholy Frenzy (`Auhf`) | bugfix | The drain is a non-attack damage event from the caster every second, as measured. |
| Polymorph (`Aply`) | partial | The sheep cannot attack or cast; type id and current life are kept, as measured. |
| Ability added to a polymorphed unit; spell freed while disarmed | unverified | The added ability is disabled at once; the freed spell is usable at once. |
| Idle autocaster that cannot cast | bugfix | Auto-attacks instead of standing still. |

Details:
- Ensnare: flyers sink and are targeted as ground units; the net is negative physical and not dispellable (measured). Duration and cooldown come from the map's data. A silenced caster can cast it (unverified).
- Unholy Frenzy: DataB life per second, no armor; attack type reads chaos and damage type universal (unverified). It can kill, and wakes or provokes the target.
- Polymorph: heroes and summons refused; harvest, build, repair, load and items off. Known gaps: turn rate and collision unchanged.

### Targeting rules

| Native | Status | What differs |
|---|---|---|
| Ability target flags (all abilities, `IssueTargetOrder`) | unverified | Flags of one kind are alternatives; "enemy,neutral" also accepts Neutral Passive units. |
| Target lists with "self" | bugfix | "self" lets the caster be targeted; a self-only list accepts only the caster. |
| Physical spells on ethereal targets | bugfix | Refused with `Notethereal`. |
| Ability-builder autocast with "enemies" | deliberate | Never picks its own caster. |
| Script-added ability without its `Requires=` research | unverified | Reforged refuses the cast (measured); WarMash enforcement not checked. |

## Orders

Attack orders end when the target can no longer be attacked. A paused unit still accepts orders.

| Native | Status | What differs |
|---|---|---|
| `IssuePointOrder` | unverified | Same as `IssuePointOrderById`; a nil order string returns false. |
| `IssueImmediateOrder`, `IssueImmediateOrderById` with "stop" | bugfix | Returns true. |
| Orders to a paused unit | gap | A paused unit still accepts orders. |
| Order issued inside an issued-target-order trigger condition | bugfix | The outer order keeps its target (was replaced or crashed). |
| `IssueBuildOrder` | works | Resolves the unit name; an unknown name returns false. |
| Build placement off the pathing grid | bugfix | Refused with the Cantplace error. |
| Channel base order strings that are not WC3 order strings | unverified | Get WarMash-only order ids (0xD8000 to 0xE7FFF), not the real game's. |
| Targeted order on an unseen, immune or ethereal unit | bugfix | The visibility error comes first, so nothing about the unit is revealed. |
| Click on a unit the active spell cannot target | deliberate | Shows the target error and casts nothing; targeting stays active. |
| Attack target turns invulnerable or invisible | unverified | The attacker drops the target and ends the attack order. |
| Attack target lost mid-chase or mid-backswing | deliberate | The attack ends at once; a swing before its damage point deals nothing. |

## Damage and attack events

DAMAGING fires before armor, DAMAGED after. Events fire only for units, not for misses or dead or invulnerable targets.

```lua
-- Halve all damage that is not an attack
local t = CreateTrigger()
for i = 0, bj_MAX_PLAYER_SLOTS - 1 do
    TriggerRegisterPlayerUnitEvent(t, Player(i), EVENT_PLAYER_UNIT_DAMAGED, nil)
end
TriggerAddAction(t, function()
    if not BlzGetEventIsAttack() then
        BlzSetEventDamage(GetEventDamage() * 0.5)
    end
end)
```

| Native | Status | What differs |
|---|---|---|
| `GetEventDamage`, `GetEventDamageSource`, `BlzGetEventDamageTarget`, `BlzSetEventDamage` | works | Outside a damage event the getters return 0 or nil and the setter does nothing. |
| The same getters outside an event | unverified | 0, nil or false, with one log line. |
| `BlzGetEventAttackType`, `BlzGetEventDamageType`, `BlzGetEventWeaponType`, `BlzGetEventIsAttack` and the type setters | works | Types set in DAMAGING feed the armor calculation; setters return false outside an event. |
| `EVENT_UNIT_DAMAGING`, `EVENT_PLAYER_UNIT_DAMAGING` | works | Before armor; setting 0 blocks the hit, a negative amount heals. |
| `EVENT_UNIT_DAMAGED`, `EVENT_PLAYER_UNIT_DAMAGED` | works | After armor; the amount set is the final life loss. |
| When DAMAGED is skipped | unverified | Only when DAMAGING set the amount to 0 or less (behaviour of 1.31.1 and later). |
| Damage dealt inside a damage handler | deliberate | Fires the events again, up to 32 levels deep. |
| Handler makes the target invulnerable or kills it | unverified | Invulnerable: the hit deals nothing; `KillUnit` in DAMAGING still fires DAMAGED. |
| Damage set to 0 or less, seen by lifesteal and thorns | unverified | Counts as 0 damage. |
| Hit by a unit without thorns | bugfix | Exactly one DAMAGING and one DAMAGED per hit. |
| `EVENT_UNIT_ATTACKED`, `EVENT_PLAYER_UNIT_ATTACKED` | works | Fires once per swing, before the damage point. |
| When trigger actions run | deliberate | Damage triggers run actions inside the event; other events queue code actions until the tick ends. |
| `GetHandleId(attacktype)` | bugfix | Returns the `common.j` index, 0 to 6. |
| `UnitDamageTarget` hit sound | bugfix | Plays the weapon type's sound; `WEAPON_TYPE_WHOKNOWS` is silent. |
| `UnitDamagePoint` | works | Honours the delay and collision size, hits the source and allies, as measured. |

## Buffs

The buff filter natives use a mostly inferred classification table: check the buffs your map relies on.

| Native | Status | What differs |
|---|---|---|
| `UnitCountBuffsEx`, `UnitHasBuffsEx`, `UnitRemoveBuffs`, `UnitRemoveBuffsEx` | works | Filter by polarity, magic or physical, aura and timed life on an inferred table. |
| Non-stacking aura buffs from several sources | bugfix | The buff stays listed while another source still applies. |
| Spell buffs with the same buff id | deliberate | A timed spell buff replaces the older one; auras and passives still stack. |
| `UnitRemoveAbility(unit, buffId)` | bugfix | Removes every instance, not only the first. |
| Slow (`Aslo`) on re-level of a disabled ability | bugfix | No longer errors. |
| `UnitApplyTimedLife` | works | A later call replaces the timer without killing the unit. |
| `UnitApplyTimedLife` with 0, negative or huge durations | bugfix | 0 or negative expires on the next tick; huge values no longer overflow. |
| `UnitApplyTimedLife` target art | bugfix | No target art is shown. |

Details:
- Timed-life buff `BTLF` is classed neither magic nor physical, so filters with both false never count it. Removing it kills the summon.
- Polymorph `Bply` counts with `(false, true, false, false, false, false, false)`. Ensnare's net is negative physical and not dispellable (measured).

## Movement, pathing and terrain

```lua
-- Speed reads are not rounded (base 270 with Bloodlust +25%)
print(GetUnitMoveSpeed(u))                     --> 337.5
print(BlzGetUnitRealField(u, UNIT_RF_SPEED))   --> 337.5
```

| Native | Status | What differs |
|---|---|---|
| `SetUnitX`, `SetUnitY` and other script moves | bugfix | Drawn at the new position within one tick. |
| `SetUnitFacing`, `SetUnitFacingTimed` | bugfix | Turn gradually at the turn rate, at most about 382 degrees per second (cited formula). |
| `BlzSetUnitFacingEx` | deliberate | Instant; cancels a pending turn. |
| `SetUnitFlyHeight` | bugfix | The rate is honoured; 0 or NaN is instant. |
| `SetUnitFlyHeight` below 0 | unverified | Ground unit types are clamped at 0. |
| Snared units (`UNIT_TYPE_SNARED`) | bugfix | Do not walk and read move speed 0, as measured. |
| `GetUnitMoveSpeed`, `BlzGetUnitRealField(UNIT_RF_SPEED)` with a speed buff | bugfix | Unrounded value, as measured; movement itself still uses whole numbers. |
| `GetUnitDefaultMoveSpeed` | unverified | The unit type's speed from object data; 0 for a null unit. |
| Slow or speed upgrade while a speed buff is active | bugfix | Change the base speed without baking in the buff; WC3's combination rule is unverified. |
| `IsTerrainPathable` | partial | Works for walk, fly, build and float pathing; other types return false. |
| `SetTerrainPathable` | unverified | `true` clears the cell's blocking bit, or blight with `PATHING_TYPE_BLIGHTPATHING`. |
| `SetTerrainPathable` with peon-harvest, amphibious or any pathing | gap | Ignored with one log line. |
| `GetTerrainType` | unverified | The tile of the nearest terrain corner. |
| `SetTerrainType` | stub | The ground does not change. |

## Regions and unit groups

```lua
-- Spatial enumerations return corpses, as in WC3: filter them
GroupEnumUnitsInRange(g, x, y, 600.0, Filter(function()
    return not IsUnitType(GetFilterUnit(), UNIT_TYPE_DEAD)
end))
```

| Native | Status | What differs |
|---|---|---|
| `RegionAddRect`, `RegionAddCell`, `RegionAddCellAtLoc` over a standing unit | bugfix | The unit is inside at once but gets no enter event. |
| `GroupEnumUnitsOfType`, `...OfPlayer`, `...InRect`, `...InRange`, `...InRangeOfLoc`, `...Selected` and their Counted forms | bugfix | Clear the group first; the spatial forms skip Locust units. |
| `GroupEnumUnitsInRect`, `GroupEnumUnitsInRange`, `GroupEnumUnitsInRangeOfLoc` | bugfix | Return dead units until the corpse is gone, as in WC3. |
| `GroupEnumUnitsInRangeOfUnit` and its Counted form (WarMash only) | bugfix | Clear first, skip Locust units, accept a null unit. |

## Players and game state

In a single-player start of a non-melee map only the local player's slot is playing.

| Native | Status | What differs |
|---|---|---|
| `IsPlayerAlly`, `IsPlayerEnemy` | unverified | Ally means a passive alliance toward the other; a player is its own ally. |
| `SetPlayerState` | bugfix | Config phase keeps only `ALLIED_VICTORY` and `GIVES_BOUNTY`; in-game states fixed. |
| Slot states in a single-player non-melee start | unverified | Other user slots are empty, not playing. |
| `IsUnitOwnedByPlayer` | works | False when an argument is nil. |
| `GetChangingUnit`, `GetChangingUnitPrevOwner`, change-owner events | bugfix | Fire for `SetUnitOwner` and Charm; nesting stops at 16 levels. |
| `GetPlayerUnitCount` | unverified | Living units including structures; units in a training queue never count. |
| `GetPlayerStructureCount` | bugfix | Counts living structures only. |
| `GetPlayerTypedUnitCount` | bugfix | `includeUpgrades` counts upgraded tiers; dead units are excluded. |
| Cancel of a supply-blocked revive or train | bugfix | Returns no food that was never paid. |
| `EndGame` | gap | Leaves to the menu, no score screen. |
| `RestartGame`, `ChangeLevel` | deliberate | Leave to the menu. |
| `DisplayTimedTextFromPlayer`, `GetDefaultDifficulty`, `UnitSuspendDecay` | works | Implemented. |
| `GetSellingUnit`, `GetRescuer`, `GetSoldItem` | gap | Always nil: no sell or rescue events. |
| `PauseGame`, `DisplayLoadDialog`, `PingMinimap`, `UnitAddIndicator`, `VolumeGroupSetVolume`, `AddItemToStock`, `RemoveItemFromStock`, `DestroyUbersplat`, `RemoveWeatherEffect` | stub | One log line, no effect. |

## Timers, triggers and hashtables

Timers fire at their real period, also below the 0.05 s tick.

| Native | Status | What differs |
|---|---|---|
| Repeating timer under 0.05 s | bugfix | Fires at its real period, up to 50 times per tick. |
| Timeouts just under a whole tick (0.65 s) | bugfix | Fire on the exact tick, not one early. |
| `TimerStart` on a running timer | bugfix | Restarts it. |
| `PauseTimer`, `TimerGetRemaining` | bugfix | The remaining time stays frozen; a second pause does nothing. |
| `PauseTimer` then `ResumeTimer` in the same tick | bugfix | The last call wins. |
| Timer destroyed by another timer due in the same tick | gap | Its callback for that tick still runs once. |
| Repeating timer under 1 ms | deliberate | Fires every 1 ms, at most 50 times per tick. |
| `TimerStart` with NaN or negative infinity | deliberate | Acts like 0; `TimerGetTimeout` returns 0. |
| `TimerStart` with +Infinity | deliberate | Never fires. |
| `PauseTimer`, `ResumeTimer`, `DestroyTimer` with nil | bugfix | Do nothing. |
| `TriggerEvaluate` and trigger conditions | bugfix | Every condition runs and the results are combined with AND. |
| `Not(boolexpr)` | bugfix | Negates (returned the input unchanged). |
| `TriggerRemoveAction`, `TriggerClearActions` | unverified | An action removed during the same firing still runs once. |
| Hashtable natives | works | One slot per value type; `RemoveSaved*`, `FlushParentHashtable`, `FlushChildHashtable` work. |

## Selection and input

Selection works for the local player only. Mouse and key events are not synced: orders issued from them would desync a multiplayer game.

| Native | Status | What differs |
|---|---|---|
| `SelectUnit`, `ClearSelection`, `GroupEnumUnitsSelected`, `IsUnitSelected` | works | No 12-unit cap; `GroupEnumUnitsSelected` sees changes only on a later turn, as in Reforged. |
| Selection events (`EVENT_PLAYER_UNIT_SELECTED`, `DESELECTED` and the unit forms) | bugfix | Fire on every path, once per frame; `RemoveUnit` fires no deselect, as in Reforged. |
| `SyncSelections` | stub | Does nothing (single client). |
| `EnableSelect`, `EnablePreSelect`, `EnableDragSelect`, `BlzEnableSelections` | stored-only | Selection keeps working normally. |
| `BlzIsSelectionEnabled`, `BlzIsSelectionCircleEnabled` | stored-only | Return the stored values. |
| `BlzIsUnitSelectable` | bugfix | False for Locust units. |
| Clicking an item or destructable | bugfix | Empties the unit selection. |
| Mouse events and `BlzGetTriggerPlayerMouse*` | partial | Local player only; move fires at most once per tick. |
| Portrait click with a unit-target command armed | works | Targets the selected unit. |
| `BlzTriggerRegisterPlayerKeyEvent` and key getters | partial | One fire per press and release, no repeat, no Windows key. |
| F1 to F3 | unverified | Select the nth hero; a second press centres the camera without a pan. |
| F4 to F7 | deliberate | Also select heroes 4 to 7 (stock WC3 has F1 to F3). |

## Items and shops

Shop purchases by script work; the stock rules are unverified and there are no sell events.

| Native | Status | What differs |
|---|---|---|
| `BlzGetItemIntegerField` with `'iarm'` and tint fields | gap | Return 0 with one log line. |
| `BlzSetItemName`, `BlzSetItemDescription`, `BlzSetItemTooltip`, `BlzSetItemExtendedTooltip`, `BlzSetItemIconPath`, `BlzGetItemTooltip`, `BlzGetItemExtendedTooltip` | works | Per-item overrides; a script icon also replaces the disabled icon. |
| `BlzItemRemoveAbility`, `UnitDropItemPoint` | works | Drops at once within 32 units, otherwise orders a drop. |
| `BlzSetItemBooleanField` | partial | Four fields; others return false. |
| `UnitAddItem`, `UnitHasItem` | bugfix | `UnitAddItem` returns a boolean. |
| `UnitUseItem`, `UnitUseItemTarget` | deliberate | False when the order is refused (a potion at full life); Reforged returns true. |
| `UnitUseItemPoint` | deliberate | True when the order is accepted; WC3 always returns false. |
| `IssueNeutralImmediateOrderById` | partial | Buys items and units; no sell event, rally point or tavern revive. |
| `IssueNeutralPointOrder`, `IssueNeutralPointOrderById`, `IssueNeutralTargetOrder`, `IssueNeutralTargetOrderById` | unverified | `neutralinteract` sets the shop selection for the ordering player only. |
| `IssueNeutralImmediateOrder` (string form) | gap | Not implemented on purpose. |

## UI frames and command card

The Frame API works for script frames, and the stock parts that custom-UI maps move (minimap, portrait, hero buttons, buff bar, tooltips, world frame) are real. Other origin frames are empty placeholders.

### Frame layout and events

| Native | Status | What differs |
|---|---|---|
| `BlzFrameSetScale` | unverified | The frame's own scale, else the parent's; point offsets scale with the frame (not for TEXT). |
| `BlzFrameGetWidth`, `BlzFrameGetHeight` | unverified | Lay out first and return the scaled size. |
| `BlzFrameIsVisible` | bugfix | False when any ancestor is hidden, as measured. |
| `BlzFrameSetParent` | deliberate | Copies the parent's level; the frame's own alpha resets to 1 (same look). |
| `BlzFrameClearAllPoints` without a new point | unverified | The frame is not drawn or clickable until it gets a point; root frames are exempt. |
| Draw order of overlapping frames | unverified | Siblings by level, then creation; children above their parent. |
| `BlzFrameGetChild`, `BlzFrameGetChildrenCount` on the game UI root | gap | Engine frames `SmashFloatingBars` (0) and `SmashWorldFrame` (1) come first, so indices differ from WC3. |
| `BlzFrameGetChild` on buttons | unverified | Documented order; out of range returns nil. |
| `BlzFrameSetText` on a template-less GLUETEXTBUTTON | bugfix | Shows the text. |
| `BlzCreateFrameByType` EDITBOX or SLIDER without a template | bugfix | Works. |
| `BlzGetOriginFrame`, `BlzTriggerRegisterFrameEvent` with an out-of-range index | unverified | Return nil with one log line. |
| `BlzFrameClick` on a disabled button | bugfix | Fires no click event. |
| `BlzEnableUIAutoPosition`, `BlzFrameSetFocus`, `BlzFrameCageMouse`, `BlzFrameSetTextSizeLimit`, `BlzTextAreaFrameSetAutoScroll` | stored-only | Value stored, one log line, no effect. |
| Frame events: checkbox, edit box, slider, popup menu, mouse enter, leave, up and wheel | works | Fire; disabled frames fire none. |
| Frame events MOUSE_DOWN, MOUSE_DOUBLECLICK, SPRITE_ANIM_UPDATE | deliberate | Never fire, as observed in WC3. |
| Frame events DIALOG_ACCEPT, DIALOG_CANCEL | gap | Never fire: no dialog frame type. |
| Keyboard focus after a click | gap | WC3's focus quirk is not modelled. |

Details:
- Slider events fire only on a value change; popup item index is 0-based; wheel delta is +120 or -120 (sign unverified).

### Text and fonts

| Native | Status | What differs |
|---|---|---|
| Template-less TEXT frame: size | unverified | Sized from its text with one point, else 0.012 high; font height 0.0105. |
| `BlzFrameSetAbsPoint` on a scaled TEXT frame | bugfix | The offset is not scaled; only the font is. |
| Template-less TEXT frame: line pitch | deliberate | 0.0095 at scale 1, scaled with the frame. |
| TEXT frame placed by two points | deliberate | Wraps at the frame width, between words. |
| Colour codes (`&#124;c`, `&#124;r`) | bugfix | Coloured runs line up with the preceding text. |
| Text that overflows a fixed-height TEXT frame | bugfix | Whole lines outside the frame are not drawn. |
| Multi-line CENTER text | unverified | Each line is centred; right-aligned lines still align left. |
| `BlzFrameSetEnable` on TEXT | stub | No visual effect. |
| `BlzFrameSetTextAlignment` CENTER, MIDDLE | unverified | Text is centred in the frame. |
| `BlzFrameSetFont` | partial | Only the height is applied. |
| Script TEXT under a later-sibling BACKDROP | bugfix | Text draws above the backdrop, as seen in WC3. |
| Text at an effective scale below 0.05 | deliberate | Not drawn. |

### Textures and sprite frames

| Native | Status | What differs |
|---|---|---|
| `BlzFrameSetTexture` with a texture that fails to load | unverified | Keeps the old texture; the failure is logged once. |
| `BlzFrameSetTexture` with "" or `NONE` | unverified | Clears the texture. |
| `.png` texture paths | gap | Do not load. |
| Texture by bare file name | deliberate | Exact path only, plus an extension swap. |
| `[FrameDef]` in war3mapSkin.txt | gap | Not applied. |
| `BuildTimeIndicator` skin key | gap | Ignored. |
| Tiled BACKDROP | bugfix | Tiles stay inside the inner rect. |
| SPRITE frame given an image path | unverified | Draws nothing, one log line. |
| `BlzFrameSetSpriteAnimate` | partial | Plays the animation at once; queue flags are ignored. |
| `BlzFrameSetModel` with a camera index the model lacks | unverified | Drawn flat at the frame's bottom left, playing sequence 0. |
| Such screen-space models: draw layer | gap | Drawn above all UI, tooltips included; parent alpha ignored. |
| Such models: visibility and lifetime | unverified | Follow the frame's visibility; freed on `BlzDestroyFrame`. |
| `BlzFrameSetModel` on a hidden frame | unverified | The model loads when the frame is first shown. |
| `BlzFrameSetModel` with a blank path | unverified | Clears the model. |

### Stock HUD and origin frames

| Native | Status | What differs |
|---|---|---|
| `BlzHideOriginFrames` | unverified | Hides the stock HUD; `false` restores it; script children stay visible. |
| `BlzGetOriginFrame` for message frames, `ConsoleUIBackdrop`, `CommandBarFrame`, `SimpleHeroLevelBar`, `Leaderboard*` | gap | Empty placeholder frames. |
| Minimap (`MiniMapFrame`, `ORIGIN_FRAME_MINIMAP`) | works | The real minimap; frame natives take effect. |
| Minimap draw order | unverified | Drawn below all root-level script frames. |
| Minimap when `ConsoleUI` moves | gap | Does not follow. |
| `ORIGIN_FRAME_WORLD_FRAME` | bugfix | A real frame: once fetched, its rect sets the 3D view. |
| Frames parented to the world frame | works | Draw below the stock HUD. |
| `ORIGIN_FRAME_UNIT_PANEL_BUFF_BAR` and its label | bugfix | Real frames. |
| `ORIGIN_FRAME_HERO_BAR`, `HERO_BUTTON`, `HERO_HP_BAR`, `HERO_MANA_BAR`, `HERO_BUTTON_INDICATOR` | unverified | Real clickable hero icons; not hidden by `BlzHideOriginFrames`. |
| `ORIGIN_FRAME_PORTRAIT` | deliberate | Drawn exactly in the frame rect; Reforged's offset quirk is not copied. |
| Inventory parts when `ConsoleUI` moves | bugfix | Follow its position, not its visibility, alpha or scale. |
| `BlzGetFrameByName` for `InfoPanelIcon*` stat frames | bugfix | Contexts 0, 1, 2 and 6 work; other contexts return nil. |
| Frames that follow a moved `SimpleNameValue` | bugfix | Class value, level bar and progress indicator move with it. |
| Hero XP bar at max level | unverified | Shown full. |
| Minimap camera box | unverified | A trapezoid outline; Reforged shows a larger rectangle. |
| Top menu buttons on Lua maps | deliberate | Reforged labels; the buttons open nothing. |
| `ORIGIN_FRAME_UNIT_MSG` | gap | Placeholder: moving it has no effect. |
| Status bar fill | bugfix | Clamped to 0..1. |
| FPS text | deliberate | Top right, smaller. |

Details:
- Portrait: frame alpha and scale are not applied; a size-0 frame shows no portrait.
- Hero buttons: a dead hero's icon is darkened; F1 to F3 select heroes; stock layout unverified.

### Tooltips

| Native | Status | What differs |
|---|---|---|
| `ORIGIN_FRAME_UBERTOOLTIP` placement | bugfix | The box follows the script-placed text and stays on screen. |
| `BlzGetOriginFrame(ORIGIN_FRAME_TOOLTIP)` | deliberate | The one-line hero-button tip, not the command tooltip box. |
| `BlzFrameIsVisible`, `BlzFrameSetVisible` on that tip | deliberate | Return the script's flag; the engine shows the tip without changing it. |
| Placement of that tip | deliberate | Above the hovered hero icon until a script sets a point. |

### Floating health bars

| Native | Status | What differs |
|---|---|---|
| HP bars on Frame API maps | works | Drawn under all UI and script frames. |
| "Always show Health Bars" option | deliberate | On by default; holding Alt hides the bars. |
| Bar look and height | unverified | Heroes get a box with level, HP and mana; height from the Stand animation. |

### Command card and cursor

| Native | Status | What differs |
|---|---|---|
| Command button hotkey letters | deliberate | Lua maps show each button's hotkey, as in Reforged; JASS maps show none. |
| Passive item-based abilities | deliberate | Get no command button. |
| Disabled button without disabled art | bugfix | Normal icon at 45 percent brightness. |
| Cursor tint | deliberate | Red over enemies, green over own or allied units, yellow over neutral units and objects. |

### Chat and text messages

| Native | Status | What differs |
|---|---|---|
| `ORIGIN_FRAME_CHAT_MSG` | bugfix | The real chat area; lines fade after 10 s; local only. |
| `BlzDisplayChatMessage` | gap | Not implemented. |
| `TriggerRegisterPlayerChatEvent`, `GetEventPlayerChatString`, `GetEventPlayerChatStringMatched` | works | Fire from the local player's chat; chat is not sent over the network. |
| `DisplayTextToPlayer` duration | bugfix | `len*0.1 + 3` seconds, as in WC3. |

### Multiboards and quests

| Native | Status | What differs |
|---|---|---|
| `Multiboard*` (27 natives) | deliberate | Drawn at the top right, one board at a time; items are 0-based. |
| `MultiboardSetRowCount`, `MultiboardSetColumnCount` shrinking | unverified | Removed cells lose their values. |
| Multiboard item widths | deliberate | Applied on the next display, as documented. |
| Multiboard and `ShowInterface(false)` | unverified | Hidden; an unset title colour is gold. |
| `MultiboardReleaseItem` | deliberate | Does nothing; the cell stays. |
| Multiboard item with an invalid icon | gap | Draws nothing; WC3 draws green. |
| `MultiboardDisplay(board, false)` | gap | Leaves the shown board alone; WC3 closes it. |
| Multiboard drawing details | unverified | Long text is not clipped; the board is click-through. |
| Multiboard borders | bugfix | Sit on the boxes measured in Reforged. |
| `Quest*` (21 natives) | stored-only | Values are kept and read back; there is no quest dialog. |

### Cinematic filters and interface

| Native | Status | What differs |
|---|---|---|
| `SetCineFilter*`, `DisplayCineFilter`, `IsCineFilterDisplayed`, `CinematicFadeBJ` | unverified | A full-screen fade below all UI, interpolated in game time. |
| Cine filter blend modes | partial | BLEND and ADDITIVE exact; the others approximated. |
| `SetCineFilterTexMapFlags` | stored-only | Ignored. |
| `EnableUserUI` | stored-only | Input is not blocked. |

## Camera

| Native | Status | What differs |
|---|---|---|
| Camera setup field slots 7 to 13 | gap | Accepted and ignored. |
| `SetCameraField`, `AdjustCameraField` | unverified | Interpolated over the duration; the mouse wheel drops the script values. |
| `CAMERA_FIELD_ROLL` and the local pitch, yaw and roll fields | stored-only | The view does not change. |
| `GetCameraField` | unverified | Radians for angles. |
| `GetCameraEyePositionX/Y/Z/Loc`, `GetCameraTargetPositionX/Y/Z/Loc` | unverified | Implemented. |
| `SetCameraPosition` | bugfix | Cancels a running pan. |
| `CAMERA_FIELD_ZOFFSET` | deliberate | Replaces the preset height until the camera is reset. |
| `CameraSetSourceNoise`, `CameraSetTargetNoise` and their Ex forms | bugfix | A sinusoidal sway (shape inferred). |
| Camera bounds | unverified | The target is clamped every frame, as in a Reforged run. |
| `BlzCameraSetupApplyForceDurationSmooth`, `BlzCameraSetupGet/SetCameraType`, `BlzCameraSetupGet/SetLabel`, `BlzGet/SetCameraAllowsHotkeyTargetLock`, `CameraSetDepthOfFieldScale`, `CameraSetFocalDistance`, `CameraSetSmoothingFactor`, `Get/SetCameraFieldControlledByInput`, `SetCameraOrientController`, `SetCameraQuickPosition`, `SetCameraRotateMode`, `SetCinematicCamera`, `StopCamera` | gap | Not implemented (17 natives). |

## Special effects and sound

Effects attached to a unit ignore position, orientation and alpha. Images are not drawn.

### Special effects

| Native | Status | What differs |
|---|---|---|
| `BlzSetSpecialEffect*` position, scale, orientation, colour, alpha, time and `BlzPlaySpecialEffect` | works | Angles in radians, colour 0 to 255; a missing animation falls back to Stand. |
| Position, orientation or alpha natives on an attached effect | deliberate | Ignored without a crash (Reforged reportedly crashes). |
| Effects on bones of a faded-out geoset | bugfix | Drawn at their current spot, not frozen. |
| `BlzSpecialEffectAddSubAnimation`, `RemoveSubAnimation`, `ClearSubAnimations` | partial | `ROOTED`, `ENTANGLE` and `BERSERK` are ignored. |
| `AddSpecialEffectTarget` attachment names | unverified | Case-insensitive token match; an unknown name attaches to origin. |
| `DestroyEffect` | bugfix | Death plays once, at once; particles fade out. |
| Effect whose model has no Stand | unverified | Loops until `DestroyEffect`. |
| `DestroyEffect` at time scale 0 | deliberate | Death plays at speed 1 so the effect is removed. |
| `BlzSetSpecialEffectAlpha`, `BlzSetSpecialEffectColor` on particles | unverified | Do not reach particles and ribbons. |
| `BlzSetSpecialEffectPositionLoc`, `BlzResetSpecialEffectMatrix` | gap | Not implemented. |
| Drawn position of a moved effect | deliberate | Smoothed between ticks; getters return the script value. |
| Global sequences of a held effect | bugfix | Keep running, as seen in Reforged. |
| One-shot engine art off-screen | bugfix | Finishes and leaves. |
| `BlzPlaySpecialEffect` with a type other than STAND or WALK | deliberate | Plays once, then Stand. |
| `BlzSetSpecialEffectTimeScale(e, 0)` with `BlzSetSpecialEffectTime` | bugfix | The set frame is held. |
| `AddSpecialEffect` with an empty model path | bugfix | Returns an invisible handle instead of nil. |
| `CreateImage` family | stored-only | State is kept; nothing is drawn. |

### Models and textures

| Native | Status | What differs |
|---|---|---|
| MDX texture wrap flags | bugfix | Applied per model; one model no longer changes another. |
| MDX texture animation id out of range | unverified | Ignored instead of failing the model. |
| `Preload` with a model path | bugfix | Loads the model for `AddSpecialEffect`. |
| Effect model preload at map load | deliberate | Model paths in the script are loaded during the loading screen; `-nopreloadfx` turns it off. |
| Ribbon with no visibility key in the animation | unverified | Does not emit. |
| Splats and ribbons under distance fog | bugfix | Each uses its own fog setting; no flicker. |
| Models spawned by model events (gibs, blood) | unverified | Leave within 5 s of their end. |
| TGA types 3, 10 and 11, other origins | bugfix | Load. |
| 32-bit TGA with 0 alpha bits | unverified | The 4th byte is alpha if any pixel uses it. |
| TGA above 8192x8192 | unverified | Rejected. |
| JPEG BLP with an implausible size | deliberate | Decoded the old way; no visible difference. |

### Sound

| Native | Status | What differs |
|---|---|---|
| `StopSound` | gap | Stops at once; `fadeOut` is ignored. |
| `KillSoundWhenDone`, `SetSoundVolume`, `ClearMapMusic` | partial | Work on `CreateSound` handles only, not label or MIDI sounds. |
| `StartSound` on a playing looping sound | unverified | Does nothing; a refused start is retried by the next call. |
| Decoded sound data | deliberate | Shared per file; only memory and start time differ. |
| `SetSoundPosition` | partial | Used at the next start; `SetSoundDistances` and `AttachSoundToUnit` are missing. |
| Sounds never killed | bugfix | Released when the map closes. |
| 3D sound distance | unverified | Measured from the camera target. |

## Lua runtime

Lua 5.3 with 64-bit integers. A declared but missing native returns nil and logs one line.

| Native | Status | What differs |
|---|---|---|
| Lua integers | gap | 64-bit, not 32-bit. |
| `require`, `package.preload`, `package.loaded` | deliberate | Available (WC3 has none). |
| `TypeDefine` | unverified | Does nothing. |
| Strings passed to natives | gap | Invalid UTF-8 becomes U+FFFD. |
| Null string results | unverified | Appear as nil. |
| `ExecuteFunc` | unverified | Queued as a new thread; an unknown name is logged once. |
| `TriggerSleepAction` outside a trigger action | unverified | The callback ends with one logged traceback. |
| Handles as weak-table keys | unverified | The key can vanish while the handle lives. |
| `GetLocalPlayer()` stored at load time | unverified | Should work in WarMash; in Reforged call it at use time. |
| `CreateTimer`, `CreateGroup` in `config()` | works | Implemented. |

## JASS runtime

JASS maps read `common.j` and `Blizzard.j` from the configured install (1.29 today).

| Native | Status | What differs |
|---|---|---|
| `GetTriggerUnit`, `GetFilterUnit`, `GetEnumUnit` from `main` | gap | Throw an error. |
| Integer literal with a leading zero (`007`) | gap | The script fails to parse. |
| Frame API natives in JASS maps | gap | Only if the install's `common.j` declares them. |
| One-character rawcode (`'a'`) | unverified | Read as 0x61000000, not 97. |

## Map loading and data

A map with `war3map.lua` is a Lua map. Data a non-melee map does not override comes from the 1.29 files.

| Native | Status | What differs |
|---|---|---|
| Lua or JASS | unverified | Lua when the archive contains `war3map.lua`. |
| `JASS_FILE_LIST` extras for Lua maps | gap | Ignored. |
| Lua maps' `common.j`, `blizzard.j` | deliberate | Bundled 3.0.1 copies, whatever the install. |
| Stock balance data for non-melee maps | deliberate | 1.29 values (Tavern sells 5 heroes, Ensnare 12 s), not Reforged's. |
| A map's `Units\CommandFunc.txt` | unverified | Replaces the stock file; missing sections use the built-in ones. |
| Map textures vs game textures | deliberate | A map file of the same base name wins, also across `.blp`, `.tga` and `.dds`. |

## Native coverage and constants

Missing natives are not listed one by one. Stored-only and stub natives are in their topics above.

| Native | Status | What differs |
|---|---|---|
| Unimplemented natives | gap | 594 declared natives missing in the Lua game VM (count of 2026-11-16); see Details for the list. |
| A declared native without implementation | gap | Returns nil and logs one line; no error is raised. |
| `MathRound` | unverified | Half away from zero. |
| `PLAYER_COLOR_BLACK`, `MAP_RELOADED` | gap | nil. |
| `Convert*` anim, subanim, fog style, key and mouse values | gap | Values exist, but no native uses the new ones yet. |

Details:
- Get the list: launch a Lua map and read the sorted `[lua] natives declared without implementation` list (the second one) in the newest `Logs/*.err.log` in the game's assets folder.
- Earlier counts: a stock JASS melee map 304 (2026-11-16); 865 of 1712 before later work.
