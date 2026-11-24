---
title: Abilities and spells
nav_order: 2
parent: Differences from Warcraft III
---

# Abilities and spells
{: .no_toc }

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
| `BlzUnitDisableAbility`, `SetPlayerAbilityAvailable` on `Aatk` | bugfix | No damage, also from a swing in progress and auto-acquire; Reforged accepts the attack order and the unit walks, WarMash refuses it. |
| `BlzUnitHideAbility` on `Aatk` | deliberate | The attack stays hidden and unusable; Reforged: hiding it has no effect. Visual-only hiding is planned as a `Wms` native. |
| `UnitRemoveAbility`, `GetUnitAbilityLevel`, `BlzGetUnitAbility` on `Aatk` (getters also `Amov`) | bugfix | Removal returns true and the unit has no attack; the level reads 1, then 0 after removal; the handle is not nil. |
| Removing `Aatk` during an attack | unverified | Ends the attack without a stop order and drops only queued attack orders; WC3 behavior not confirmed. |
| `UnitAddAbility(u, 'Aatk')` after a removal | deliberate | Restores the attack; a morph also restores a script-removed attack. |
| `BlzUnitDisableAbility`, `BlzUnitHideAbility` on `Aatk`, `Amov` | bugfix | Disables or hides the engine ability; a disabled attack stops, no auto-acquire, attack orders refused (rules inferred). |
| `BlzEndUnitAbilityCooldown` on a Spell Book child | unverified | Ends the book's cooldown only when that child started it; a script cooldown on the book stays. |
| `GetAbilityLevel(nil)` | bugfix | Returns 0 instead of raising an error. |
| Boolean, non-level and array ability field natives (`BlzGet/SetAbilityBooleanField` and similar) | gap | Not implemented. |
| Ability hidden by `BlzUnitHideAbility`: orders | bugfix | Refuses orders like a disabled ability, as measured; a running cast finishes. |
| `ConvertAbilityDisableType(7)` | deliberate | Valid in WarMash (disable type POLYMORPH); out of range in WC3. |
| `SetAbilityDisabled`, `SetAbilityIconShowing` (WarMash only) | deliberate | Script disable and icon switches; `SetAbilityIconShowing` does not refresh the command card by itself. |
| `BlzGetUnitAbilityByIndex` | unverified | Index 0 is the newest ability; abilities the engine adds itself are skipped. |
| `BlzGetAbilityStringLevelField` | partial | 14 string fields, 0-based level; other fields return "". |
| Tooltip text from `BlzGetAbilityStringLevelField` | unverified | Returned raw, placeholders unresolved. |
| `BlzSetAbilityIcon` | works | Sets the normal icon on all levels; the disabled icon is unchanged. |
| `BlzGetAbilityRealLevelField`, `BlzGetAbilityIntegerLevelField` | works | Every field is known; level is 0-based; a script-set value is returned first. |
| `BlzSetAbilityRealLevelField(ABILITY_RLF_CASTING_TIME)` on ability-builder spells | bugfix | Read on every cast; no level refresh needed. |
| Shadow Strike slow and hit | bugfix | No error on hit; the slow always ends (buff removal, `UnitRemoveBuffs` or expiry). |
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
| Recasting a map-built channel ability while it channels | unverified | The old channel ends first, so its end actions run before the new cast; an Aerial Shackles recast frees the first target. |
| Channel (`ANcl`) | works | Per-level fields `Ncl1` to `Ncl6`, mana, cooldown and all five spell events. |
| Channel with follow-through time 0 | unverified | Treated as an instant cast (0.001 s). |
| Channel cast after a walk into range | bugfix | The cast runs normally and FINISH fires at its end; before, it ended at once with ENDCAST only. Failing for lack of mana no longer leaves the unit stuck. |
| Channel: `SPELL_FINISH` / `SPELL_ENDCAST` timing | bugfix | Fire in the tick the follow-through ends, after that tick's timers, as measured. |
| Ability-builder (JSON) channels: end events | deliberate | Same order as Channel by analogy: FINISH then ENDCAST at the channel end; a cut channel fires ENDCAST only. |
| `BlzPauseUnitEx` during a Channel cast | bugfix | Ends the cast; at the unpause a cast cut before its EFFECT is re-issued, as measured. |
| `PauseUnit` during a cast | bugfix | Cuts the cast like `BlzPauseUnitEx`, as measured; Channel `ANcl` ignores the pause. A null unit is ignored. |
| `PauseUnit` on a unit the engine also pauses (loaded in a transport) | unverified | Stays an engine pause: the cast freezes instead of being cut, and a cut ended by an engine unpause is not re-issued. |
| Cast point 0: `PauseUnit`, `BlzPauseUnitEx` or an order from a `SPELL_CAST` or `SPELL_EFFECT` trigger action | gap | Cannot cut the cast; in Reforged it cuts (from EFFECT: no effect, mana spent, no re-cast), as measured. Pausing from the trigger condition cuts. |
| `BlzPauseUnitEx` unpause after the EFFECT of a Channel cast | deliberate | Reforged re-issues the order, so one order can cast several times; WarMash does not, and the unit takes its next order or goes idle. |
| `BlzPauseUnitEx` in the CAST or EFFECT of an instant Channel | bugfix | EFFECT, FINISH and ENDCAST fire in one tick and the order is not re-issued, as measured. |
| Paused instant Channel: target lost before EFFECT | unverified | A dead, hidden or untargetable target ends the cast with ENDCAST, no EFFECT, no mana. |
| Paused instant Channel: target lost after EFFECT | works | FINISH and ENDCAST still fire. |
| Paused instant Channel: stun during the pause | unverified | The cast ends with ENDCAST in the unit's next update. |
| Script error in a FINISH or ENDCAST action | bugfix | Other casts ending in the same tick still end normally. |
| Channel with blank `Animnames` | deliberate | No cast animation plays. |
| Channel: stun before EFFECT | unverified | The cast ends; WC3 reportedly casts again after the stun. |
| Channel: order given before EFFECT | unverified | Interrupts the cast; not queued. |
| Channel: `Ncl5` (disable other abilities) | unverified | Greys only spell and item abilities. |
| Channel: target art | gap | The target art is not spawned. |
| `GetTriggerPlayer` in spell events | gap | Returns nil. |
| `GetSpellTargetX`, `GetSpellTargetY`, `GetSpellTargetLoc` | unverified | The target point, else the target's position, else 0. |
| Spell events of the engine spells (Storm Bolt, Polymorph, Charm and similar) and the JSON ability-builder spells | works | CHANNEL and CAST at cast start, EFFECT at the cast point after mana and cooldown, FINISH and ENDCAST after the backswing; a cut gives ENDCAST only. |
| Engine spells: caster hold time | bugfix | The caster is held for the cast point plus backswing, as measured for Storm Bolt; before, only the backswing. |
| Engine spells: backswing delay, autocast, cut after EFFECT | unverified | Inferred from Channel behavior, not measured. |
| Engine spells: stun or sleep before EFFECT | unverified | ENDCAST fires, then the unit casts again after the stun, mana paid once. |
| Engine spells: stun after EFFECT | unverified | ENDCAST only; an order given from that ENDCAST runs after the stun. |
| `BlzPauseUnitEx` from an EFFECT action on Animate Dead or Neutral Charm style channels | bugfix | Stops the channel. |
| Item heal and mana potions | unverified | Fire all five spell events with the potion's ability id; charge spend and `EVENT_UNIT_USE_ITEM` follow after the script pass. |
| Ability-builder (JSON) spells: CHANNEL order | bugfix | CHANNEL and CAST fire at cast start, before EFFECT; before, CAST came first and CHANNEL only after EFFECT. |
| Ability-builder (JSON) spells: FINISH and ENDCAST | bugfix | Both fire at the end of the backswing or channel; before, ENDCAST fired before FINISH on every end. |
| Ability-builder (JSON) spells with begin-casting actions (other than the morphs below) | unverified | Mana and cooldown are paid at cast start, and the hand-off fires CHANNEL, CAST, EFFECT, FINISH, ENDCAST in one tick. |
| Spell events of morph abilities (Bear Form, Crow Form, Burrow, Stone Form, Metamorphosis and similar) | bugfix | CHANNEL, CAST, EFFECT fire in the order tick with the old type, mana and cooldown paid at EFFECT; FINISH, ENDCAST follow after the morph time with the new type, as measured. |
| Morph abilities: stop, second cast, death, stun | partial | A stop after the window does not cancel and reverting pays mana again, as measured; death mid-morph gives ENDCAST only and stun defers FINISH and ENDCAST (both unverified). |
| Morph abilities: order issued from the EFFECT handler | bugfix | `Issue*Order` cuts the morph: ENDCAST inside the call, no FINISH, old type, mana and cooldown stay spent, as measured. |
| Morph abilities: queued order across a morph; stop or timer order in the EFFECT tick | partial | Queued move, patrol, follow and hold-position orders run after the morph; queued attack orders are dropped. A same-tick timer order may also cut. |
| Morph abilities: re-ordering the morph's own ability from the EFFECT handler | unverified | Ends the first cast with one ENDCAST and casts again, so the direction follows the unit type; Reforged probably rejects it. |
| `RemoveUnit` with an open cast (Channel, engine spells, JSON spells, morphs) | bugfix | ENDCAST fires once inside the call, PLAYER event before UNIT, no FINISH, no EFFECT if before it; a removed morph does not complete, as measured. |
| Ability-builder (JSON) spells: stun before EFFECT | bugfix | The cast restarts after the stun and pays mana once. |
| Ability-builder (JSON) spells: stunned caster dies, is removed, gets a new order or loses the ability | bugfix | The parked cast is dropped: end-channel actions run once and nothing resumes after `ReviveHero`. |
| `BlzPauseUnitEx` from a spell handler on ability-builder spells | unverified | Cuts the cast with ENDCAST inside the pause call; the unpause re-issues the order if EFFECT had not fired. |
| `BlzPauseUnitEx` during a Java spell-base cast (Storm Bolt and similar) | bugfix | Before EFFECT: ENDCAST in the pause call, the unpause re-issues the order, mana once; after EFFECT: ENDCAST only, as measured. |
| Spell CHANNEL, CAST, EFFECT actions of engine and JSON spells | bugfix | They run inside the event after the order's `ISSUED_ORDER` handlers; a pause, stop, new order or target kill from those handlers cuts the cast before any event or payment, as measured. |
| Engine and JSON spells: EFFECT action that pauses, stops or removes the caster | bugfix | The effect is skipped; mana and cooldown stay spent, no FINISH, no re-cast, as measured. |
| Engine and JSON spells: CAST or EFFECT action that kills or removes the target | unverified | The cast ends. An item spell cut at EFFECT still spends its charge. |
| Engine spells with a Casting Time (Entangle Gold Mine) | works | CHANNEL at start, CAST after the Casting Time, EFFECT one cast point later; a cut in between gives CHANNEL then ENDCAST. |
| Casting Time of ability-builder (JSON) spells | works | CHANNEL, then CAST after the Casting Time, EFFECT one cast point later; a stop, stun or pause-ex cut gives ENDCAST only and spends nothing. |
| Ability-builder spells that reuse the Casting Time for something else (transformations, Shadow Strike, Replenish, Flame Strike), toggles, item abilities, Casting Time under 0.05 s | deliberate | CAST still fires with CHANNEL; no pre-cast wait. |
| Engine pause (transport, Mass Teleport) during the casting time, cast point or backswing | unverified | The wait is frozen and resumes after the unpause. |
| Pause during a Channel (`ANcl`) cast | works | The follow-through ends on schedule, then the order reads 851973 until the unpause (the overrun is unverified). |
| Castless item abilities (ability-builder, e.g. a potion on Brilliance Aura) | works | All five spell events fire in the use tick with the item ability id; then one charge is spent and `EVENT_UNIT_USE_ITEM` fires. |
| Castless item ability that cannot pay its mana at the cast point | bugfix | Ends with ENDCAST only: no charge spent, no `EVENT_UNIT_USE_ITEM`. |
| `BlzSetUnitBooleanField` | gap | Not implemented; it is the last missing native that the reference map's spell code calls. |

Details:
- Channel event order: CHANNEL, CAST after the casting time, EFFECT (mana and cooldown taken), then FINISH and ENDCAST. An interrupted cast fires ENDCAST only.
- `GetUnitCurrentOrder` reads 851973 between a `BlzPauseUnitEx` pause and the re-issue.

### Stock ability classes

| Native | Status | What differs |
|---|---|---|
| Animate Dead (`AUan`, `AIan`, `ACad`) | works | Raises the 6 nearest raisable corpses as timed summons of the caster's owner; refused without mana or cooldown cost when none is in range. |
| Animate Dead details | unverified | Corpse choice, hero and building refusal, exploding on death, no food charge and ignored Inherit Upgrades are not confirmed in Reforged. |
| Animate Dead takeover | deliberate | No `EVENT_UNIT_CHANGE_OWNER` or `EVENT_PLAYER_UNIT_CHANGE_OWNER` fires when the corpse changes owner; only SUMMON events fire. |
| Storm Bolt (`AHtb`), Attribute Bonus (`Aamk`) | works | A missile with speed 0 or no art arrives at once. |
| Cripple (`Acri`), Drunken Haze (`ANdh`) | works | Implemented; Drunken Haze `Nsi1` is a bit mask (1 melee, 2 ranged, 4 special, 8 spells). |
| Drunken Haze overlapping | gap | Two from the same caster: the disarm ends when the first expires. |
| Cleave (`ANca`) | unverified | Melee only; each unit in the area takes the full factor and no attack event fires. |
| Item sight bonus (`AIsi`) | unverified | Raises sight while carried; the sight field reads the value without it. |
| Spell damage reduction (`AIsr`) | unverified | Reduces `ATTACK_TYPE_NORMAL` and `ATTACK_TYPE_MAGIC` damage. |
| `Aclf`, `Aprg`, `AEer` | gap | Do nothing: no engine class yet. |
| Spell Book (`Aspb`) | partial | The book button opens a sub-card of the listed abilities, keyed by base order id; children cast normally with their own spell events. |
| Spell Book children | unverified | Shown only in the book and hidden per ability; removing the book removes them, disabling keeps them; Minimum and Maximum are ignored. |
| Spell Book shared cooldown (`spb2`) | unverified | Casting a listed ability puts the book on the same cooldown; siblings keep their own; a cast without a target closes the book. |
| `StartUnitAbilityCooldown`, `BlzStartUnitAbilityCooldown` on a Spell Book child | bugfix | An idle book stays at 0, as measured; a book cooling down from a cast follows the child; a book cooldown set by script stays (unverified). |
| `BlzSetUnitAbilityCooldown`, `BlzEndUnitAbilityCooldown` on a Spell Book child | unverified | Move the book too; the last call wins, and ending also frees a book cooldown set by script. |
| Stock wands (`wneg`, `sneg`, `wcyc`, `wshs`, `wswd`, `wlsd`, `will`, `woms`) | gap | Their base abilities are not implemented; use and order calls return false. |
| Hardened Skin (`Assk`) | unverified | Attacks lose the ignored damage down to the minimum, before armor; attacks only, with chance and ranged or melee flags. |
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
- Polymorph: heroes and summons refused; harvest, build, repair, load and items off. The unit turns at the sheep's rate; collision is unchanged (known gap).

### Targeting rules

| Native | Status | What differs |
|---|---|---|
| Ability target flags (all abilities, `IssueTargetOrder`) | unverified | Flags of one kind are alternatives; "enemy,neutral" also accepts Neutral Passive units. |
| Target lists with "self" | bugfix | "self" lets the caster be targeted; a self-only list accepts only the caster. |
| Physical spells on ethereal targets | bugfix | Refused with `Notethereal`. |
| Ability-builder autocast with "enemies" | deliberate | Never picks its own caster. |
| Script-added ability without its `Requires=` research | unverified | Reforged refuses the cast (measured); WarMash enforcement not checked. |
