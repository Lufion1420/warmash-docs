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
| `BlzSetUnitDiceNumber`, `BlzSetUnitDiceSides`, `BlzSetUnitAttackCooldown`, `BlzSetUnitBaseDamage` | bugfix | Extreme values are clamped and non-finite cooldowns ignored; negative sides crashed and huge dice froze. |
| Attack dice upgrade (`BlzGetUnitDiceNumber`, `BlzGetUnitDiceSides`) | works | The upgrade raises the number of dice only, as measured. |
| `BlzGetUnitAttackCooldown`, `BlzIsUnitInvulnerable` | works | The cooldown is the base value without attack-speed bonuses. |
| `BlzGetUnitRealField` | partial | 32 fields (see Details); other fields return 0 with one log line. |
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
| Death of a unit type with bounty dice but sides 0 or less | bugfix | No longer raises an error. |
| Timed buffs on a unit revived by a death-replacement ability | bugfix | Slow, Bloodlust, stun and other timed buffs are cleared at death, as measured; timed life stays. |
| Summoned unit with Reincarnation | gap | It loses its timed life on reincarnating and never expires; Warcraft III behavior not confirmed. |
| `RemoveUnit` on a unit in a Reincarnation delay | deliberate | The unit stays removed with no revive or death event; Reforged keeps its typed unit count, WarMash releases it. |
| `GetUnitTypeId`, `GetUnitX`, `GetUnitY`, `GetUnitLoc`, `GetWidgetX`, `GetWidgetY` after `RemoveUnit` | bugfix | Read 0 after the call, as measured; events fired by the removal (`SPELL_ENDCAST`) still read real values. |
| `GetWidgetLife`, `GetUnitState` life and max life, `BlzGetUnitMaxHP` after `RemoveUnit` | unverified | Read 0; Reforged was measured only on a hero in a Reincarnation false death. |
| `UnitAlive`, `IsUnitType` dead, owner, name, mana, facing, range natives, `GetUnitCurrentOrder` after `RemoveUnit` | gap | A unit removed mid-action still reads its old state; `UnitAlive` stays true. |
| A unit's own actions after `RemoveUnit` | unverified | A removed unit no longer acts in that tick: no auto-acquire or order queue step. |
| `GetWidgetX`, `GetWidgetY` with nil | deliberate | Return 0 instead of raising a script error. |
| `RemoveUnit` called from a spell `EFFECT` condition | unverified | A Java-base spell's effect still runs; Reforged not measured. |
| `RemoveUnit` on a dead hero, then `ReviveHero` in the same tick | deliberate | Reforged revives it, as measured; WarMash returns false and revives nothing. |
| `RemoveUnit` on a hero that died normally | unverified | Releases its tech-tree count once (`GetPlayerTechCount`, hero training limit); before, the dead removed hero counted forever. |
| `RemoveUnit` on a unit that is already removed or whose corpse has decayed | unverified | Does nothing more: the tech-tree count is released once, and no second removal is logged. |
| `RemoveUnit` on a live haunted or entangled gold mine | bugfix | The parent mine is restored at the next unit-add step, so `IsUnitHidden(parent)` is still true inside the same trigger; Reforged not measured. |
| `RemoveUnit` on a hero queued at an altar | unverified | The queue entry is cancelled and gold, lumber and food are refunded to the altar owner. |
| `BlzSetUnitMaxMana` with 0 or less | unverified | Max mana is clamped to at least 0. |
| `BlzSetUnitRealField` | partial | Sight, regeneration, acquisition, turn rate, speed, cast times, armor, growth and some unit values work. |
| `BlzSetUnitRealField(UNIT_RF_CAST_POINT)`, `UNIT_RF_CAST_BACK_SWING` | works | Set per unit and used by casts; negative values become 0; kept across a morph (not confirmed in Reforged). |
| `GetUnitCastPoint`, `GetUnitCastBackswingPoint` | bugfix | Return the per-unit value, not only the unit type's. |
| `BlzGetUnitIntegerField` | partial | Hero, defense, armor, tint, bounty, level, targeting, regeneration, movement and some type fields work. |
| `BlzSetUnitIntegerField` | partial | Classification, hero base stats, attribute, defense, armor, bounty, level and targeted-as work. |
| `BlzSetUnitIntegerField` with `UNIT_IF_MOVE_TYPE` | gap | Returns false and changes nothing; Reforged 2.0.3 applies it, as documented. |
| `BlzSetUnitIntegerField` with `UNIT_IF_PRIMARY_ATTRIBUTE` | bugfix | Now applies, as Reforged 2.0.3 documents; see the hero stat fields below. |
| `UNIT_IF_UNIT_CLASSIFICATION` writes | bugfix | Bit 128 is only Ancient and never blocks targeting; `IsUnitType(ANCIENT)` and ancient target lists read it, as measured. |
| `UNIT_IF_UNIT_CLASSIFICATION` write replaces the field | bugfix | Writing 128 or 0 clears `IsUnitType(MECHANICAL)`, and 0 clears the type's own Ancient, as measured. |
| `UnitAddType`, `UnitRemoveType` after a classification write | bugfix | Set and clear the mechanical, undead and ancient bits (128 plus mechanical reads 136), as measured. |
| `BlzGetUnitBooleanField` | bugfix | Reads the unit's own value, not only the type's; `uatg` and `ufhp` return what was written. |
| `BlzSetUnitBooleanField` | partial | `urai`, `udec`, `uhhb`, `unbm` apply per unit until a type change; `ubdg` returns false. |
| `BlzSetUnitBooleanField`, other fields | stored-only | Stored per unit and read back until a type change; no effect. |
| `BlzSetUnitBooleanField(hero, UNIT_BF_RAISABLE, true)` | deliberate | Returns false and writes nothing unless the hero type is already raisable; Warcraft III behavior not confirmed. |
| `BlzGetUnitStringField`, `BlzSetUnitStringField` | partial | `unam` and `upro` work per unit; other unsupported fields read empty and set false. |
| `BlzSetUnitStringField` with `uubs`, `ushu` | stored-only | Stored per unit and read back; nothing visible changes. |
| `BlzSetUnitName` | unverified | Per-unit name for `GetUnitName` and the HUD; nil or empty is ignored; lost on type change. |
| `SetUnitState` life and mana | bugfix | Clamped to the maximum; lowering max life does not lower current life. |
| `BlzGetUnitCollisionSize`, `GetDestructableTypeId`, `GetDestructableOccluderHeight` | works | Implemented. |
| `IsUnitInRange`, `IsUnitInRangeLoc`, `IsUnitInRangeXY` | bugfix | Real range checks from the first unit's edge; `IsUnitInRangeXY` was always false. |
| `IsUnitPaused`, `IsUnitInTransport`, `IsUnitLoaded`, `GetHeroSkillPoints`, `UnitInventorySize`, `IsPointBlighted` | works | `IsUnitPaused` is true after `PauseUnit`, false under `BlzPauseUnitEx`, as measured. |
| `IsUnitType` with `UNIT_TYPE_POISONED`, `UNIT_TYPE_PLAGUED` | gap | Always false: these effects do not exist in WarMash yet. |
| `IsUnitType(unit, UNIT_TYPE_POLYMORPHED)` | works | True while the unit is under Polymorph. |
| Null unit in `BlzGetUnitArmor`, `GetUnitDefense`, `GetUnitCastPoint`, `GetUnitCastBackswingPoint`, `UnitGetTemporaryDefenseBonus`; `GetSpellTargetUnit`, `GetSpellTargetLoc` outside a spell | bugfix | Return 0 or nil instead of a script error. |
| `SetWidgetLife`, `SetUnitState(UNIT_STATE_LIFE)` and heals on a killed unit | bugfix | Ignored: a killed unit stays at 0 life until a real revive (was a crash). |

Details:
- `BlzGetUnitRealField` fields: `uhpr`, `umpr`, `udfc`, `usir`, `uacq`, `upri`, `uamn`, `udtm`, `uerd`, `ufrd`, `umxp`, `umxr`, `urun`, `uwal`, `uslz`, `ushh`, `ushw`, `ushx`, `ushy`, `uocc`, `ucpt`, `ucbs`, `ustp`, `uagp`, `uinp`, `uhpc`, `umpc`, `umvc`, `umvr`, `ufyh`, `usca`, `ussc`. Armor is unrounded; `usca` ignores `SetUnitScale`; sight radius excludes ability bonuses.
- Boolean fields: `urai`, `udec`, `ubdg`, `ulos`, `unbm`, `uhhb`, `uhhm`, `uhhd`, `uhom`, `uscb`, `usew`, `ushr`, `uatg`, `ufhp`; others read false.
- Stored-only boolean fields: `ulos`, `uhhm`, `uhhd`, `uhom`, `uscb`, `usew`, `ushr`, `uatg`, `ufhp`.
- Weapon index of the `BlzSet/GetUnit*` damage natives is 0 or 1; out of range a setter does nothing and a getter returns 0.

### Hero stat and defense fields

| Native | Status | What differs |
|---|---|---|
| `ustc`, `uagc`, `uinc` (integer) | unverified | Read and write hero stats without bonuses; writes clamp to 0..1000000. |
| `ustb`, `uagb`, `uinb` (integer) | unverified | Read hero stats with bonuses; setting them returns false. |
| `ustm`, `uagm`, `uinm` (integer) | gap | Read 0, and setting returns false. |
| `ustp`, `uagp`, `uinp` (real) | unverified | Read and write the hero's own per-level growth, 0..1000, kept across a morph. |
| Hero stat and growth fields on a non-hero | unverified | Stats read 0, growth reads the type's value, and setters return false. |
| `upra` write | unverified | Takes 1 STR, 2 INT, 3 AGI on a hero; reset on unit type change. |
| `upra` read on a non-hero | deliberate | Reads 0, even when the unit type has a primary attribute. |
| `udty` (integer) | unverified | Sets the unit's defense type, 0..7; reset on unit type change. |
| `udty` with Polymorph or Root | unverified | Polymorph ending, rooting or uprooting can overwrite a scripted value. |
| `uarm` (integer) | unverified | Sets hit-sound armor type 0..5; 0 has no sound and also replaces custom armor text; reset on type change. |
| `BlzSetUnitRealField(UNIT_RF_DEFENSE)` | unverified | Sets base armor exactly, buffs still add; the last write wins against `BlzSetUnitArmor`. |
| NaN or infinity in `ustp`, `uagp`, `uinp`, `udfc` | unverified | The setter returns false. |
| Hero stat and defense fields on a removed unit | unverified | Writes return false and reads give 0. |

### Bounty, level and other unit fields

| Native | Status | What differs |
|---|---|---|
| `ubba`, `ubdi`, `ubsi`, `ulba`, `ulbd`, `ulbs` (integer) | unverified | Read and write this unit's gold and lumber bounty base, dice and sides. |
| `ulev` (integer) | unverified | Writes a non-hero's level; on a hero it reads the hero level and returns false. |
| `GetUnitLevel` after a `ulev` write | unverified | Returns the written level on non-heroes. |
| `utar` (integer) | unverified | Replaces which target types this unit counts as. |
| `upri`, `uamn`, `udtm` (real) | unverified | Read and write this unit's priority, minimum attack range and death time. |
| Long `udtm` on a hero | unverified | Also delays its death event, decay and altar revival. |
| `uhpc`, `umpc` write (real) | unverified | Sets current life or mana; an unchanged value triggers no state events. |
| `uhpc` write above 0 on a dead unit | unverified | Returns false and does not revive it. |
| `ufyh` write (real) | unverified | Changes the fly height at once. |
| `uhrt` write (integer) | stub | Returns true but the regeneration type does not change; reading works. |
| `umvt`, `ufor`, `uori`, `uept`, `ulfi`, `ulfo` (integer) | unverified | Read only; setting returns false. |
| `uerd`, `ufrd`, `umxp`, `umxr`, `urun`, `uwal`, `uslz`, `ushh`, `ushw`, `ushx`, `ushy`, `uocc` (real) | unverified | Read the unit type's value; setting returns false. |
| Setting `usca`, `ussc`, `uclr`, `uclg`, `uclb`, `ucal`, `upar`, `ufmh` | gap | Returns false and changes nothing. |
| Written bounty, `ulev`, `utar`, `upri`, `uamn`, `udtm` | unverified | Lost on unit type change. |
| Bounty, level, life, mana and other fields above on a removed unit | unverified | Writes return false and reads give 0. |

### Weapon fields

| Native | Status | What differs |
|---|---|---|
| `BlzGetUnitWeapon*Field`, `BlzSetUnitWeapon*Field` (Integer, Real, Boolean, String) | partial | Per-unit attack values at index 0 or 1; writes are lost on unit type change. |
| Weapon fields on a dead unit | deliberate | Read and write like a living unit's; Warcraft III behavior not confirmed. |
| Weapon fields on a removed unit, bad index or missing attack | unverified | Setters return false; getters read 0, false or empty. |
| Setting `udl1`, `usd1`, `usr1`, `umh1` | stub | Returns true and changes nothing. |
| Fields the attack's weapon type does not hold | stub | Stored on the attack and read back; no effect. |
| `uwu1` (show attack in UI) | partial | Stored and read back, but the info panel still shows attack 1. |
| `ua1t`, `ucs1` integers | unverified | Follow `ConvertAttackType` and `ConvertWeaponType` order; out-of-range values return false. |
| Weapon field value limits | unverified | Values outside the limits are clamped; NaN and infinity return false. |

### Weapon fields: attacks enabled (`uaen`)

| Native | Status | What differs |
|---|---|---|
| `uaen` write | unverified | Per-attack script value that survives silence, disarm and root; morph or upgrade resets it. |
| `uaen` on a unit with Root | unverified | A script enable never exceeds what the current root state allows. |
| `uaen` first enable without an Attack ability | unverified | Adds the Attack ability, unless a script removed it. |
| `uaen` read | unverified | Returns the written value, else the type's setting, even while the attack is unusable. |
| `uaen` off, then on, in one callback | unverified | The running or stun-parked attack continues. |
| `uaen` off during a stun | unverified | If still off when the stun ends, the parked attack ends without damage. |

### Weapon fields: targets (`ua1g`, `ua1p`)

| Native | Status | What differs |
|---|---|---|
| `ua1g`, `ua1p` values | unverified | Documented target bits; undefined bits are ignored; flags without a bit, such as self refusal, stay. |
| `ua1g`, `ua1p` reads | unverified | Ancient makes the integer negative; "friend" reads as `0x6000`. |
| Written mask without any target bit (0, "none") | unverified | The attack targets nothing. |
| Writing back the value just read | unverified | Changes nothing, so `ua1p = 0` on an empty type set still hits everything. |
| Restoring a read 0 after another write, or copying it | unverified | The 0 then means "targets nothing". |
| Mask with only alliance or class bits | unverified | Without ground, air or structure bits, WarMash applies no type limit. |
| `ua1p` targeting nothing on a splash attack | unverified | The primary target is still hit. |
| `ua1p` written on a bounce attack | unverified | The bounce never hits its attacker. |
| `ua1p` on an attack without splash or bounce | stub | Stored and read back; no effect. |

Details:
- Fields: `ua1d`, `ua1b`, `ua1s`, `utc1`, `ua1t`, `ucs1`, `ubs1`, `udp1`, `ua1c`, `udl1`, `uhd1`, `uqd1`, `usd1`, `usr1`, `ua1z`, `uma1`, `ua1f`, `ua1h`, `ua1q`, `ua1r`, `uwu1`, `umh1`, `ua1m`, `uaen`, `ua1g`, `ua1p`.
- Limits: dice 0-10000, sides 0-100000, cooldown, damage point, backswing 0-3600, areas and speed 0-100000, range 16-100000, targets 1-10000, `uhd1`/`uqd1` 0-1000, `uma1` -10 to 10.

### Heroes

| Native | Status | What differs |
|---|---|---|
| `BlzGetHeroPrimaryStat` (hidden native) | bugfix | Returns STR 0, AGI 1, INT 2 as in Reforged, following `upra` writes; 0 for a non-hero. |
| `GetHeroStr`, `GetHeroAgi`, `GetHeroInt` | works | Values match a Reforged run on one hero. |
| `SetHeroStr`, `SetHeroAgi`, `SetHeroInt` | bugfix | Getters return the value set and a nil unit is ignored; the non-permanent form does nothing. |
| Hero stat changes inside a life state-event handler | bugfix | No longer overwritten with stale armor or regeneration; state events fire once, after the recalculation. |
| `GetHeroLevel`, `GetUnitLevel`, `GetHeroXP` | works | Implemented. |
| `GetHeroProperName` | works | Returns nil for a non-hero. |
| `BlzSetHeroProperName` | bugfix | Resolves `TRIGSTR_n` and refreshes cargo and tooltips; unresolvable text stays literal, also in unit name and texture setters. |
| Hero proper name at creation | gap | Stock heroes always get their first proper name, not a random one. |
| `ResurrectUnit`, Resurrection, Reincarnation end | bugfix | Food and tech count released by death come back once by the stored amounts; a unit killed again during the revive is skipped. |
| `ResurrectUnit` on a dead hero in an altar revive queue | unverified | The hero leaves the queue and the altar's food and gold are refunded; Reforged behavior not confirmed. |
| `ResurrectUnit` on a dissipated hero corpse | bugfix | The corpse is unhidden. |
| `GetUnitCurrentOrder` after `ReviveHero`, altar revive, Resurrection or Reincarnation | unverified | Reads 0; before, the order of the cast the unit died in. |
| `ReviveHero` on a hero still in a stun state | unverified | Stunned at once (permanent stun buff or `UNIT_TYPE_STUNNED`); before, unstunned until the next stun change. |
| `ReviveHero`, `ReviveHeroLoc` | works | Fire no revive events; refused when the food cap would be exceeded. |
| `SetWidgetLife`, `SetUnitState(UNIT_STATE_LIFE)` in a life state-event handler during a revive | bugfix | The unit stays dead if the handler killed it inside the revive, as in Warcraft III; before, it was revived. |
| `ReviveHero` or altar revive when a handler during the revive kills or removes the hero again | bugfix | `ReviveHero` returns false, no finish event, no rally order; the dead hero keeps no food. Reforged not measured. |
| `ReviveHero` called by a handler inside a running revive (altar refund, enter-region or state handler) | bugfix | It does the only revive: one START, one FINISH, no second revive art, no altar rally order. Not confirmed against Reforged. |
| A handler kills then revives the hero during a revive (instant respawn) | bugfix | The revive that ran last decides; a `ReviveHero` after a new death fires its own START. Not confirmed against Reforged. |
| `ReviveHero` from a gold-refund handler near the food ceiling | bugfix | No longer refused; altar-paid food is not counted twice. |
| Altar-queued hero whose Resurrection raise ends in a Reincarnation false death | bugfix | The altar entry is cancelled and refunded. |
| Altar revive cancel of a paid entry | unverified | The refunded food goes straight to the hero; no waiting train item takes it. |
| Altar revive gold, lumber and time | bugfix | Capped by the `HeroMaxRevive*` gameplay constants. |
| Altar revive lumber cost | unverified | Checks and charges the hero type's lumber cost; before, the gold cost was the base. |
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
| Model change (skin, morph) with buff art, held units, rally flag or lightning | unverified | Each moves to the matching point of the new model, else origin. |
| Model change of a dead unit | unverified | The death or decay stage continues on the new model without repeating sounds or splats. |
| Resurrection of a unit with a changed skin | bugfix | The unit keeps its skin. |
| `SetUnitVertexColor` | bugfix | Components clamp to 0..255, as measured. |
| `SetUnitScale` with a scale buff | deliberate | Drawn scale is script scale times the buff bonus; the script scale survives the buff. |
| `GetChangingUnitPrevOwner`, `SetUnitFlyHeight` | works | Both work now; see their rows in Players and Movement. |

Details:
- Locust: not selectable, targetable or enumerated, no bars, others walk through it; script damage still lands. Its own movement still collides (gap).

### Illusions

Only [`WmsCreateIllusion`](../warmash-extensions.html) makes illusions.

| Native | Status | What differs |
|---|---|---|
| `IsUnitIllusion` | works | True for units made by `WmsCreateIllusion`; was always false. |
| Mirror Image, Wand of Illusion, item illusions | gap | These abilities do not create illusions yet. |
| `UnitAddAbility` on an illusion | deliberate | Returns false for any ability an illusion cannot keep. |
| `UnitAddItem`, `UnitAddItemById`, `UnitAddItemToSlotById` on an illusion | deliberate | Refused: false or nil. Warcraft III behavior not confirmed. |
| Hex on an illusion | gap | Does not kill it; a Reforged player report says Warcraft III does. |
| Illusion tint for allies | unverified | Seen when the viewer treats the owner as ally (`IsUnitAlly`), also one-sided; Warcraft III unmeasured. |

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
| `GetUnitCurrentOrder` after Ensnare, Polymorph, Charm or `Amov` removal stops the unit | unverified | Reads 0; was 851972. `ISSUED_ORDER` events are unchanged. |
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
