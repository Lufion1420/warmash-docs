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
| Attack: `EVENT_UNIT_ATTACKED`, `EVENT_PLAYER_UNIT_ATTACKED` | partial | Fires at swing start for unit targets only; attacks on trees and other non-unit targets fire nothing. |
| Death: `EVENT_UNIT_DEATH`, `EVENT_PLAYER_UNIT_DEATH`, `EVENT_WIDGET_DEATH` | works | `GetDyingUnit`, `GetTriggerUnit` and `GetKillingUnit` are set; the killer is nil when nothing killed the unit. |
| Spells: `*_SPELL_CHANNEL`, `_CAST`, `_EFFECT`, `_FINISH`, `_ENDCAST` | works | See the spell rows below; the player event fires before the unit event. |
| Orders: `*_ISSUED_ORDER`, `_ISSUED_POINT_ORDER`, `_ISSUED_TARGET_ORDER` | works | Order id, point and target getters are set for the matching event only. |
| Items: `*_PICKUP_ITEM`, `_DROP_ITEM`, `_USE_ITEM`, `_PAWN_ITEM`, `_SELL_ITEM` | works | Fire with the item getters of that event; see "Traps" for which getter belongs to which event. |
| Shops: `EVENT_UNIT_SELL`, `EVENT_PLAYER_UNIT_SELL` | works | Fire during the purchase for the shop and the shop owner's player. |
| Hero: `EVENT_PLAYER_HERO_LEVEL`, `EVENT_UNIT_HERO_LEVEL` | works | `GetLevelingUnit` and `GetTriggerUnit` return the hero; `GetTriggerPlayer` the owner, `GetTriggerWidget` the hero. |
| Hero skill: `EVENT_PLAYER_UNIT_HERO_SKILL`, `EVENT_UNIT_HERO_SKILL` | works | Fire on every skill learn (`SelectHeroSkill`, learn button), not on `UnitAddAbility` or `SetUnitAbilityLevel`. Player event first (inferred). |
| Hero revive: `EVENT_PLAYER_HERO_REVIVABLE` | partial | Fires with `GetTriggerUnit`, `GetTriggerPlayer` (owner) and `GetTriggerWidget` (hero). |
| Hero revive: `EVENT_PLAYER_UNIT_HERO_REVIVE_START`, `_CANCEL`, `_FINISH` and `EVENT_UNIT_HERO_REVIVE_*` | works | `ReviveHero`/`ReviveHeroLoc`: START then FINISH (START skipped if the hero is already in an altar queue; a failed revive fires nothing; no CANCEL). Altar: START at queue, CANCEL on cancel (timing inferred), FINISH at completion. Player event first. |
| Summon: `EVENT_PLAYER_UNIT_SUMMON`, `EVENT_UNIT_SUMMON` | partial | Only for Figurine, Phoenix, Grizzly, Hawk, Quilbeast, Feral Spirit, Force of Nature, Factory, Pocket Factory; player event (summoned unit's owner) first. |
| Construction and training: `*_CONSTRUCT_FINISH`, `*_TRAIN_FINISH`, `*_RESEARCH_FINISH` | works | `GetConstructedStructure`, `GetTrainedUnit`, `GetTrainedUnitType`, `GetResearched`, `GetResearchingUnit`. |
| Selection: `*_SELECTED`, `*_DESELECTED` | works | `GetTriggerUnit` and `GetTriggerPlayer` set. |
| Ownership: `*_CHANGE_OWNER` | works | `GetChangingUnit`, `GetChangingUnitPrevOwner`. |
| Regions: `TriggerRegisterEnterRegion`, `TriggerRegisterLeaveRegion` | works | `GetEnteringUnit` or `GetLeavingUnit`, `GetTriggeringRegion`, `GetTriggerUnit`. |
| Player: chat, leave, key, mouse, end cinematic | works | Local player only for key and mouse. |
| Dialogs: `TriggerRegisterDialogEvent`, `TriggerRegisterDialogButtonEvent` | works | `GetClickedDialog`, `GetClickedButton`. |
| Timers and variables: `TriggerRegisterTimerEvent`, `TriggerRegisterTimerExpireEvent`, `TriggerRegisterVariableEvent` | works | Fire as expected. |

## Events that never fire

| Event | Status | What differs |
|---|---|---|
| `EVENT_UNIT_HERO_REVIVABLE` | gap | Never fires; only the player form does. |
| Summons from JSON ability-builder abilities and script `CreateUnit`; Water Elemental summon | gap | Fire no summon event (the Water Elemental summon is not registered). |
| `EVENT_PLAYER_UNIT_DECAY`, `EVENT_UNIT_DECAY` | gap | Never fire; `GetDecayingUnit` is missing. |
| `EVENT_PLAYER_UNIT_DETECTED`, `EVENT_UNIT_DETECTED`, `*_HIDDEN` | gap | Never fire; `GetDetectedUnit` and `GetEventDetectingPlayer` are missing. |
| `EVENT_PLAYER_UNIT_RESCUED`, `EVENT_UNIT_RESCUED` | gap | Never fire; `GetRescuer` exists but returns nil. |
| `EVENT_UNIT_ACQUIRED_TARGET`, `EVENT_UNIT_TARGET_IN_RANGE`, `TriggerRegisterUnitInRange`, `TriggerRegisterFilterUnitEvent` | gap | Never fire; the two register natives are missing and `GetEventTargetUnit` is missing. |
| `*_CONSTRUCT_START`, `*_CONSTRUCT_CANCEL` | gap | Never fire; `GetConstructingStructure` and `GetCancelledStructure` are missing. |
| `*_UPGRADE_START`, `_CANCEL`, `_FINISH` | gap | Never fire. |
| `*_TRAIN_START`, `*_TRAIN_CANCEL`, `*_RESEARCH_START`, `*_RESEARCH_CANCEL` | gap | Never fire. |
| `EVENT_PLAYER_UNIT_LOADED`, `EVENT_UNIT_LOADED` | gap | Never fire; `GetLoadedUnit` and `GetTransportUnit` are missing. |
| `EVENT_UNIT_STACK_ITEM`, `*_EQUIP_ITEM`, `*_UNEQUIP_ITEM` | gap | Never fire; `BlzGetStackingItem*`, `BlzGetEquippedItem`, `BlzGetUnequippedItem`, `BlzGetAbsorbingItem` are missing. |
| `TriggerRegisterPlayerStateEvent`, `GetEventPlayerState` | unverified | Fires when the limit comparison turns from false to true; already true at registration does not fire, and it re-arms when false again. |
| `TriggerRegisterPlayerAllianceChange` | works | Fires only when that alliance type really changes; no getter tells which type or other player. |
| `EVENT_PLAYER_DEFEAT`, `EVENT_PLAYER_VICTORY`, `EVENT_GAME_VICTORY`, `EVENT_GAME_END_LEVEL`, `EVENT_GAME_LOADED`, `EVENT_GAME_SAVE` | gap | Never fire. |
| `EVENT_PLAYER_ARROW_*`, `EVENT_PLAYER_SYNC_DATA` | gap | Never fire; `BlzGetTriggerSyncData`, `BlzGetTriggerSyncPrefix` and `BlzTriggerRegisterPlayerSyncEvent` are missing. |
| `TriggerRegisterUnitStateEvent` | stub | Does nothing useful: returns a null event, and only when the continue-on-error setting is on; `GetEventUnitState` is missing. |
| `TriggerRegisterGameStateEvent` | partial | Only time of day is handled; other states raise an error. The limit is now read from the right argument (bugfix). |
| `TriggerRegisterTrackableHitEvent`, `TriggerRegisterTrackableTrackEvent`, `GetTriggeringTrackable`, `GetEventGameState`, `GetWinningPlayer` | gap | Raise "not yet implemented" when called. |

## Getters by event

| Getter | Status | What differs |
|---|---|---|
| `GetTriggerUnit` | works | Set in every firing event above; for sale events it is the shop, for pawn events the pawning unit. |
| `GetTriggerPlayer` in spell, order, item, hero level, construction and training events | gap | Returns nil; only chat, leave, key, mouse, damage, death, attack, selection, ownership, research and sale events set it. |
| `GetTriggerWidget` in spell, order, item, hero and construction events | gap | Returns nil in spell events; works for death, damage, order and selection events. |
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
| `GetSummonedUnit`, `GetSummoningUnit` | unverified | Summon events: the new unit and the caster. Which unit the unit-form event belongs to is assumed (the summoning unit). Trigger unit/widget = summoned unit. |
| `SelectHeroSkill` on abilities without a registered ability type | gap | Learns nothing and fires no event (for example Holy Light `AHhb`, `AHds`, `AHre`). |
| `GetConstructedStructure`, `GetTrainedUnit`, `GetTrainedUnitType`, `GetResearchingUnit`, `GetResearched` | works | Finish events only. |
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
| Construction | gap | Start uses `GetConstructingStructure`, cancel `GetCancelledStructure`, finish `GetConstructedStructure`; only finish works here. |
| Sale and pawn items | works | In a sale the shop is `GetSellingUnit`; in a pawn the hero is `GetSellingUnit` and the shop `GetBuyingUnit`. |
| Damage on spells | works | Spell damage reports `ATTACK_TYPE_NORMAL`; DAMAGING needs patch 1.31 or later in the real game. |
| Order events | works | Immediate, point and target orders are three separate events with their own getters. |
| Handler runs inline | works | Damage, drop, pawn, sale, hero level and spell actions run before the call that caused them returns. |
