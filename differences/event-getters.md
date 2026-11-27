---
title: Events and event getters
nav_order: 11
parent: Differences from Warcraft III
---

# Events and event getters
{: .no_toc }

Audit of which events fire and which event getters return a value, per event family. Many events never fire, and some getters are missing.

A missing getter fails at the call with an "had no native implementation" log line and returns nothing. "Never fires" means the trigger registers without error and its actions simply never run.

## Events that fire

| Event family | Status | What a map sees |
|---|---|---|
| Damage: `EVENT_UNIT_DAMAGED`, `EVENT_PLAYER_UNIT_DAMAGED`, `EVENT_UNIT_DAMAGING`, `EVENT_PLAYER_UNIT_DAMAGING` | works | One DAMAGING then one DAMAGED per hit; all `GetEvent*` and `BlzGetEvent*` damage getters work. |
| Attack: `EVENT_UNIT_ATTACKED`, `EVENT_PLAYER_UNIT_ATTACKED` | works | Fires at swing start for unit targets; attacks on trees and other non-unit targets fire nothing. |
| Death: `EVENT_UNIT_DEATH`, `EVENT_PLAYER_UNIT_DEATH`, `EVENT_WIDGET_DEATH` | works | `GetDyingUnit`, `GetTriggerUnit` and `GetKillingUnit` are set; the killer is nil when nothing killed the unit. |
| Spells: `*_SPELL_CHANNEL`, `_CAST`, `_EFFECT`, `_FINISH`, `_ENDCAST` | partial | `GetTriggerPlayer` and `GetTriggerWidget` are nil; the player event fires before the unit event. |
| Orders: `*_ISSUED_ORDER`, `_ISSUED_POINT_ORDER`, `_ISSUED_TARGET_ORDER` | partial | Order id, point and target getters work; `GetTriggerPlayer` is nil. |
| Items: `*_PICKUP_ITEM`, `_DROP_ITEM`, `_USE_ITEM` | partial | Item getters work; `GetTriggerPlayer` is nil. |
| Items: `*_PAWN_ITEM`, `_SELL_ITEM` | works | Fire with the item getters of that event; see "Traps" for which getter belongs to which event. |
| Shops: `EVENT_UNIT_SELL`, `EVENT_PLAYER_UNIT_SELL` | works | Fire during the purchase for the shop and the shop owner's player. |
| Hero: `EVENT_PLAYER_HERO_LEVEL`, `EVENT_UNIT_HERO_LEVEL` | works | `GetLevelingUnit` and `GetTriggerUnit` return the hero; `GetTriggerPlayer` the owner, `GetTriggerWidget` the hero. |
| Hero skill: `EVENT_PLAYER_UNIT_HERO_SKILL`, `EVENT_UNIT_HERO_SKILL` | works | Fire on every skill learn (`SelectHeroSkill`, learn button), not on `UnitAddAbility` or `SetUnitAbilityLevel`. Player event first (inferred). |
| Hero revive: `EVENT_PLAYER_HERO_REVIVABLE` | works | Fires when the hero death animation ends, with trigger unit, widget (hero) and player (owner). |
| Hero revive: `EVENT_PLAYER_UNIT_HERO_REVIVE_START`, `_CANCEL`, `_FINISH` and `EVENT_UNIT_HERO_REVIVE_*` | works | `ReviveHero`/`ReviveHeroLoc`: START then FINISH (START skipped if the hero is already in an altar queue; a failed revive fires nothing; no CANCEL). Altar: START at queue, CANCEL on cancel (timing inferred), FINISH at completion. Player event first. |
| Summon: `EVENT_PLAYER_UNIT_SUMMON`, `EVENT_UNIT_SUMMON` | partial | Fire for the engine summon abilities (Figurine, Phoenix, Grizzly, Hawk, Quilbeast, Feral Spirit, Force of Nature, Factory, Pocket Factory, Water Elemental, Raise Dead, Animate Dead) and JSON summons that opt in; player event first. Script `CreateUnit` fires nothing, as in Reforged. |
| Construction start and cancel: `EVENT_PLAYER_UNIT_CONSTRUCT_START`, `_CANCEL`, `EVENT_UNIT_CONSTRUCT_CANCEL` | unverified | START fires on placement (not for an instant Entangled Gold Mine); CANCEL only on an explicit cancel, not on building death or removal; player event first. |
| Training and research finish: `*_TRAIN_FINISH`, `*_RESEARCH_FINISH` | works | `GetTrainedUnit`, `GetTrainedUnitType`, `GetResearched`, `GetResearchingUnit`; run inline, player event first, as measured. |
| Construction finish: `*_CONSTRUCT_FINISH` | partial | `GetConstructedStructure` works; `GetTriggerPlayer` is nil. |
| Training finish order | bugfix | `*_TRAIN_FINISH` handlers run first, then the rally order is issued to the trained unit, then the next queue item starts; a handler's own order on the unit is overridden by the rally order. |
| Rally order id | bugfix | A unit trained with a point rally reports `851970` as in Reforged; rally on a unit or item target and altar revive still report `851971`. `OrderId2String(851970)` returns a name, Reforged none. |
| `ISSUED_*_ORDER` events after training | unverified | Dispatched after same-tick train events even when the order came first; the Warcraft III order is not measured. |
| Train and research start and cancel: `*_TRAIN_START`, `_TRAIN_CANCEL`, `*_RESEARCH_START`, `_RESEARCH_CANCEL` | works | START fires when paid production starts; CANCEL only on an explicit cancel, not on building death or removal. |
| Upgrade: `*_UPGRADE_START`, `_CANCEL`, `_FINISH` | unverified | CANCEL (75% refund) only on explicit cancel, not on building death or removal; death refunds nothing; player event first. |
| Decay: `EVENT_PLAYER_UNIT_DECAY`, `EVENT_UNIT_DECAY` | bugfix | Fire once at the end of the death animation (about 3 s, unit still dead) only for raisable and decaying corpses (death type 3), as measured. |
| Decay for other death types and units | works | Decay-only corpses (type 2) fire nothing and go at about 91 s; types 0 and 1 are removed after the animation, as measured. Never for heroes or buildings. |
| Selection: `*_SELECTED`, `*_DESELECTED` | works | `GetTriggerUnit` and `GetTriggerPlayer` set. |
| Ownership: `*_CHANGE_OWNER` | works | `GetChangingUnit`, `GetChangingUnitPrevOwner`; an Animate Dead takeover raises no event. |
| Regions: `TriggerRegisterEnterRegion`, `TriggerRegisterLeaveRegion` | works | `GetEnteringUnit` or `GetLeavingUnit`, `GetTriggeringRegion`, `GetTriggerUnit`. |
| Player: chat, key, mouse, end cinematic | works | Local player only for chat, key and mouse. |
| Player leave: `EVENT_PLAYER_LEAVE` | partial | Raised only by `RemovePlayer` for a player other than the local one. |
| Dialogs: `TriggerRegisterDialogEvent`, `TriggerRegisterDialogButtonEvent` | partial | `GetClickedDialog` and `GetClickedButton` work; `GetTriggerPlayer` is nil. |
| Timer object: `TriggerRegisterTimerExpireEvent` | partial | Fires; `GetTriggerEventId` is null, also in `TimerStart` callbacks. |
| Timer interval: `TriggerRegisterTimerEvent` | unverified | Fires with event id and triggering trigger; `GetExpiredTimer` is null. |
| `TriggerRegisterVariableEvent` | partial | Watches JASS globals only; a Lua map global raises an error. |

## Events that never fire

| Event | Status | What differs |
|---|---|---|
| `EVENT_UNIT_HERO_REVIVABLE` | gap | Never fires; only the player form does. |
| `EVENT_PLAYER_UNIT_DETECTED`, `EVENT_UNIT_DETECTED`, `*_HIDDEN` | gap | Never fire; `GetDetectedUnit` and `GetEventDetectingPlayer` are missing. |
| `EVENT_PLAYER_UNIT_RESCUED`, `EVENT_UNIT_RESCUED` | gap | Never fire; `GetRescuer` exists but returns nil. |
| `EVENT_UNIT_ACQUIRED_TARGET`, `EVENT_UNIT_TARGET_IN_RANGE`, `TriggerRegisterUnitInRange`, `TriggerRegisterFilterUnitEvent` | gap | Never fire; the two register natives are missing and `GetEventTargetUnit` is missing. |
| `EVENT_PLAYER_UNIT_LOADED`, `EVENT_UNIT_LOADED` | gap | Never fire; `GetLoadedUnit` and `GetTransportUnit` are missing. |
| `EVENT_UNIT_STACK_ITEM`, `*_EQUIP_ITEM`, `*_UNEQUIP_ITEM` | gap | Never fire; `BlzGetStackingItem*`, `BlzGetEquippedItem`, `BlzGetUnequippedItem`, `BlzGetAbsorbingItem` are missing. |
| `TriggerRegisterPlayerStateEvent`, `GetEventPlayerState` | unverified | Fires on every change after which the comparison holds; a same-value write fires nothing, and late registration fires nothing until the value changes. A trigger that changes the same resource while its condition holds re-queues itself (unverified). |
| `TriggerRegisterPlayerAllianceChange` | works | Fires only when that alliance type really changes; no getter tells which type or other player. |
| `EVENT_PLAYER_DEFEAT`, `EVENT_PLAYER_VICTORY`, `EVENT_GAME_VICTORY`, `EVENT_GAME_END_LEVEL`, `EVENT_GAME_LOADED`, `EVENT_GAME_SAVE` | gap | Never fire. |
| `TriggerRegisterGameEvent` | stub | The registration is dropped with one log line, so the trigger never fires from it. |
| `EVENT_PLAYER_ARROW_*`, `EVENT_PLAYER_SYNC_DATA` | gap | Never fire; `BlzGetTriggerSyncData`, `BlzGetTriggerSyncPrefix` and `BlzTriggerRegisterPlayerSyncEvent` are missing. |
| `TriggerRegisterUnitStateEvent`, `GetEventUnitState` | works (MANA and max states unverified) | Edge-triggered, EQ never fires, registering fires nothing; measured for life. A death hit fires it after the death; regen events are queued. |
| `TriggerRegisterGameStateEvent` | partial | Only time of day; other states raise an error. |
| `TriggerRegisterTrackableHitEvent`, `TriggerRegisterTrackableTrackEvent`, `GetTriggeringTrackable`, `GetWinningPlayer` | gap | Raise "not yet implemented" when called. |

## Getters by event

| Getter | Status | What differs |
|---|---|---|
| `GetTriggerUnit` | partial | Set in unit events (shop in sales, hero in pawns); nil in dialog events. |
| Event getters outside their event | unverified | Return null, 0 or an empty string (inferred); before, JASS maps ended with `ClassCastException`. |
| `GetTriggerEventId` in timer interval, variable and time-of-day handlers | bugfix | `EVENT_GAME_TIMER_EXPIRED`, `EVENT_GAME_VARIABLE_LIMIT` or `EVENT_GAME_STATE_LIMIT`. |
| `GetTriggeringTrigger` in timer interval, variable and time-of-day handlers | bugfix | Returns the trigger; Lua previously got nil. |
| Unit and player getters, `GetExpiredTimer` in those three handlers | unverified | Return null. |
| `GetOrderPointLoc`, `GetSpellTargetLoc` outside events and in those three handlers | unverified | Return null; other events without an order or spell give a location at 0,0. |
| `GetEventGameState` | bugfix | `GAME_STATE_TIME_OF_DAY` in a time-of-day handler, null elsewhere; before, it raised an error. |
| `GetTriggerEvalCount`, `GetTriggerExecCount` | gap | Always return 0. |
| `ExecuteFunc` (JASS and Lua) | gap | Warcraft III runs the function at once; WarMash queues it to run later. |
| Event getters and triggering trigger in an `ExecuteFunc` function | unverified | Reads the caller's values; null when called from `main`. |
| `BlzSetEventDamage`, `BlzSetEventAttackType`, `BlzSetEventDamageType`, `BlzSetEventWeaponType` after the event | unverified | After a wait or in `ExecuteFunc`: change nothing, type setters return false, logged once. |
| `TriggerSleepAction`, `TriggerWaitForSound` in an `ExecuteFunc` function | unverified | Always wait, even if the caller's trigger has `TriggerWaitOnSleeps` false. |
| `GetTriggerPlayer` in spell, order, pickup, drop, use-item, dialog and construction-finish events | gap | Returns nil. |
| `GetTriggerWidget` in spell events | gap | Returns nil; order, item, hero and construction events set it. |
| `GetKillingUnit`, `GetDyingUnit` | works | Valid in death events only; the killer is nil for script kills such as a max life of 0. |
| `GetAttacker` | works | Valid in attack events only; nil in damage events (use `GetEventDamageSource`). |
| `GetEventDamage`, `GetEventDamageSource`, `BlzGetEventDamageTarget`, `BlzGetEventIsAttack` | works | Valid in damage events; outside one they return 0, nil or false. |
| `BlzSetEventDamage`, `BlzSetEventAttackType`, `BlzSetEventDamageType`, `BlzSetEventWeaponType` | works | Take effect in DAMAGING and DAMAGED; outside a damage event they do nothing. |
| `GetSpellAbilityUnit`, `GetSpellAbilityId`, `GetSpellAbility` | works | Set in all five spell events. |
| `GetSpellTargetUnit`, `GetSpellTargetItem`, `GetSpellTargetDestructable` | works | Only the one matching the target kind is non-nil. |
| `GetSpellTargetX`, `GetSpellTargetY`, `GetSpellTargetLoc` | unverified | The target point, else the target's position, else 0. |
| `GetOrderedUnit`, `GetIssuedOrderId`, `GetOrderPointX`, `GetOrderPointY`, `GetOrderPointLoc` | works | Point getters are 0 in non-point orders. |
| `GetOrderTarget`, `GetOrderTargetUnit`, `GetOrderTargetItem`, `GetOrderTargetDestructable` | works | Only for target orders. |
| `GetManipulatingUnit`, `GetManipulatedItem` | works | Pickup, drop and use-item events; nil in sale and pawn events, as measured. |
| `GetSellingUnit`, `GetBuyingUnit`, `GetSoldUnit`, `GetSoldItem` | works | Sale and pawn events as measured; see the item trap below. |
| `GetLevelingUnit` | works | Hero level events. |
| `GetLearningUnit`, `GetLearnedSkill`, `GetLearnedSkillLevel` | works | Hero skill events: hero, ability id, new level. `GetTriggerUnit` and `GetTriggerWidget` are the hero, `GetTriggerPlayer` the owner. |
| `GetTriggerPlayer`, `GetTriggerWidget` in hero level and `EVENT_PLAYER_HERO_REVIVABLE` handlers | bugfix | Were nil; now the owner and the hero. |
| `GetRevivingUnit`, `GetRevivableUnit` | partial | In revive START and FINISH, `GetRevivingUnit` is the hero and `GetRevivableUnit` is nil (measured). Trigger unit/widget = hero, trigger player = owner. |
| `GetSummonedUnit`, `GetSummoningUnit`, and trigger unit/widget in `EVENT_UNIT_SUMMON` | bugfix | Unit form: trigger unit and widget are the caster, as measured; trigger player is the summoned unit's owner. The player form returns the summoned unit. |
| `GetSummonedUnit` in `EVENT_UNIT_SUMMON` | deliberate | Reforged returns an empty handle here, as measured; WarMash returns the real summoned unit. |
| `SelectHeroSkill` on abilities without a registered ability type | gap | Learns nothing and fires no event. |
| `SelectHeroSkill` and the learn button for Divine Shield `AHds` | bugfix | Now learns it and fires the skill events; before, nothing was learned. |
| `SelectHeroSkill` and the learn button for Holy Light `AHhb` | bugfix | Now learns it and fires the skill events; before, nothing was learned. |
| `SelectHeroSkill` and the learn button for Resurrection `AHre` | bugfix | Now learns it and fires the skill events; before, nothing was learned. |
| Resurrection `AHre` cast | unverified | Raises friendly bone-phase corpses in Area, up to DataA, in map-scan order; a cast with no corpse still spends mana and cooldown. |
| Resurrection `AHre` corpse and mana rules | unverified | Hero corpses and corpses in the first 5 s flesh phase are not raised; the revived unit keeps its death-time mana. |
| Divine Shield `AHds` cast (`divineshield` 852090, `undivineshield` 852091) | unverified | Caster invulnerable for HeroDur; early end only if Can Deactivate (DataA) is 1, never in stock data; script invulnerable flag unaffected both ways. |
| Holy Light `AHhb` cast (order `holybolt`, 852092) | unverified | Heals a living friendly non-mechanical non-self unit below max life, hurts enemy undead for half; self, full-health refusal, damage type and spell shield unconfirmed. |
| `GetConstructingStructure`, `GetCancelledStructure` | unverified | START and CANCEL only: the structure. Trigger unit and widget = the structure, trigger player = its owner. A handler that removes or kills the structure is refunded once; Entangle START/CANCEL, unit-level CANCEL after a player handler removed the structure, and an order given to the builder in START (overridden by the build) are unconfirmed. |
| `GetConstructedStructure`, `GetTrainedUnit`, `GetTrainedUnitType`, `GetResearchingUnit`, `GetResearched` | works | Finish events only. |
| `GetDecayingUnit` | unverified | Decay events only: the decaying unit. |
| `GetTrainedUnitType`, `GetResearchingUnit`, `GetResearched` in START and CANCEL | unverified | Set as in finish events; `GetTrainedUnit` is nil because the unit does not exist yet. |
| `GetRescuer` | stub | Always nil. |
| `GetEnteringUnit`, `GetLeavingUnit`, `GetTriggeringRegion` | works | Region events. |
| `GetEventPlayerChatString`, `GetEventPlayerChatStringMatched` | works | Return an empty string outside a chat event. |

## Traps in Warcraft III

Same in WarMash unless noted.

| Trap | Status | What to know |
|---|---|---|
| Attack events | works | Use `GetAttacker` in ATTACKED, `GetEventDamageSource` in DAMAGED; the other is nil. |
| Death events | works | Use `GetKillingUnit` and `GetDyingUnit`; `GetKillingUnit` is only valid in death events. |
| Hero level and skill | works | `GetLevelingUnit` for levels; `GetLearnedSkill` and `GetLearningUnit` for skills. |
| Construction | unverified | Start uses `GetConstructingStructure`, cancel `GetCancelledStructure`, finish `GetConstructedStructure`; all three are set here. |
| Sale and pawn items | works | In a sale the shop is `GetSellingUnit`; in a pawn the hero is `GetSellingUnit` and the shop `GetBuyingUnit`. |
| Damage on spells | works | Spell damage reports `ATTACK_TYPE_NORMAL`; DAMAGING needs patch 1.31 or later in the real game. |
| Order events | works | Immediate, point and target orders are three separate events with their own getters. |
| Handler runs inline | works | Damage, drop, pawn, sale, hero level and spell actions run before the call that caused them returns. |
