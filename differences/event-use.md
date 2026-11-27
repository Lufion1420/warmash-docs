---
title: "Events: which getter to use"
nav_order: 13
parent: Differences from Warcraft III
---

# Events: which getter to use
{: .no_toc }

Quick answers first, then the traps. Full tables: [unit events](event-table-units.html) and [player, game, frame and other events](event-table-other.html). Behaviour notes: [Events and event getters](event-getters.html).

The status words describe WarMash. `unverified` means Warcraft III was not compared.

## For this, use that

| You want | Use | Watch out |
|---|---|---|
| Casting unit | `GetSpellAbilityUnit`, `GetTriggerUnit` | Both are nil in `TimerStart` callbacks. |
| Spell id | `GetSpellAbilityId` | Save it in a local before starting a timer. |
| Spell target unit | `GetSpellTargetUnit` | Nil for point and no-target casts. |
| Spell target point | `GetSpellTargetX`, `GetSpellTargetY` | No allocation, unlike `GetSpellTargetLoc`. |
| Player of a caster or ordered unit | `GetOwningPlayer(GetTriggerUnit())` | `GetTriggerPlayer` is nil there. |
| Attacker and attacked unit | `GetAttacker`, `GetTriggerUnit` | Attack events only. |
| Damage source, target, amount | `GetEventDamageSource`, `BlzGetEventDamageTarget`, `GetEventDamage` | `GetAttacker` is nil here. |
| Change damage | `BlzSetEventDamage` | Before armor in DAMAGING, after armor in DAMAGED. |
| Dying unit and killer | `GetDyingUnit`, `GetKillingUnit` | Use a unit death event, not `TriggerRegisterDeathEvent`. |
| Dying destructable | `GetTriggerDestructable` | Register with `TriggerRegisterDeathEvent`. |
| Ordered unit and order | `GetOrderedUnit`, `GetIssuedOrderId` | The target getters depend on the order event. |
| Item picked up, dropped or used | `GetManipulatedItem` | Nil in sale and pawn events. |
| Item bought and its buyer | `GetSoldItem`, `GetBuyingUnit` | The buyer can be nil. |
| Item pawned to a shop | `GetSoldItem`, `GetSellingUnit` | The seller is the hero; the shop is `GetBuyingUnit`. |
| Unit bought or hired | `GetSoldUnit` | `GetTriggerUnit` is the shop. |
| Hero that levelled | `GetLevelingUnit` | |
| Skill learned | `GetLearnedSkill`, `GetLearnedSkillLevel` | |
| Finished building | `GetConstructedStructure` | No getter returns the builder. |
| Trained unit | `GetTrainedUnit` | Finish only; start and cancel have `GetTrainedUnitType`. |
| Research | `GetResearched` | |
| Summoned unit | `GetSummonedUnit` | Prefer the player form of the summon event. |
| Unit entering a region | `GetEnteringUnit` | Inside the filter use `GetFilterUnit`. |
| Chat text and sender | `GetEventPlayerChatString`, `GetTriggerPlayer` | |
| Mouse position | `BlzGetTriggerPlayerMouseX`, `BlzGetTriggerPlayerMouseY` | |
| Clicked frame | `BlzGetTriggerFrame` | |
| Expired timer | `GetExpiredTimer` | Nil with `TriggerRegisterTimerEvent`. |

## Spell getters in loops and sub-triggers

Spell getters keep the event's values in callbacks and sub-triggers. A `TimerStart` callback gets none, so store them first.

| Where | Status | What happens |
|---|---|---|
| `ForGroup`, `ForForce` and rect enumeration callbacks | bugfix | Spell getters keep the event's values; before, they read nil or 0. |
| Filters of `GroupEnumUnits*` and other enumerations | bugfix | Spell getters keep the event's values; before, they read nil or 0. |
| The filter given to `TriggerRegisterPlayerUnitEvent` | unverified | Reads the event's spell values; before, nil or 0. |
| `TriggerExecute` and `TriggerEvaluate` | bugfix | The other trigger reads the caller's spell values; before, nil or 0. |
| Trigger conditions, actions, `ExecuteFunc` functions | works | Spell getters keep their values. |
| `TimerStart` callbacks | unverified | Spell getters return nil or 0, and `GetSpellTargetLoc` a location at 0,0. |

Other event getters, including damage and order getters, also keep their values everywhere above except in timer callbacks.

```lua
local caster, spell = GetSpellAbilityUnit(), GetSpellAbilityId()
TimerStart(CreateTimer(), 0.5, false, function()
    -- use caster and spell here
    DestroyTimer(GetExpiredTimer())
end)
```

## Other traps

| Trap | Status | What happens |
|---|---|---|
| `TriggerRegisterDeathEvent` | unverified | Only `GetTriggerWidget` and `GetTriggerDestructable` can be set; `GetDyingUnit` and `GetTriggerUnit` are nil. |
| Region filter | unverified | Only `GetFilterUnit` works inside it; `GetEnteringUnit` and `GetTriggerUnit` are nil. |
| `TriggerRegisterPlayerEvent` for chat, keys, player state or alliance | unverified | Registers but never fires; use the dedicated register native. |
| Change-owner events | unverified | `GetTriggerPlayer` is the new owner in the unit form, the previous owner in the player form. |
| Player unit events | unverified | Register for the previous owner (change owner), shop owner (sales) or selecting player (selection). |
| `GetOrderPointX`, `GetOrderPointY` in a target order | unverified | Read 0; use the position of the target. |
| `GetOrderPointLoc`, `GetSpellTargetLoc` in an unrelated unit event | unverified | Return a location at 0,0, not nil. |
| Instant item abilities | unverified | Spell events carry no target: target getters are nil or 0. |
| Damage type setters in DAMAGED | unverified | Armor is already applied, so a new attack or damage type does not change the amount. |
| Damage setters after a wait | unverified | Change nothing; getters still return the values of that hit. |
| `GetTriggerEventId` for timer objects and frames | gap | Nil after `TimerStart`, `TriggerRegisterTimerExpireEvent` and `BlzTriggerRegisterFrameEvent`. |
| `TimerStart` callback | unverified | Unit and player event getters are nil; keep values in locals or tables. |
| Selection, mouse, key, chat, dialog and frame events | partial | Raised for the local player only. |

## Same value under two names

In these events both getters return the same object.

| Event | `GetTriggerUnit` equals |
|---|---|
| Damage | `BlzGetEventDamageTarget` |
| Death | `GetDyingUnit` |
| Decay | `GetDecayingUnit` |
| Spell | `GetSpellAbilityUnit` |
| Order | `GetOrderedUnit` |
| Item pickup, drop, use | `GetManipulatingUnit` |
| Sale | `GetSellingUnit` (the shop) |
| Pawn | `GetSellingUnit` (the hero) |
| Hero level, skill | `GetLevelingUnit`, `GetLearningUnit` |
| Hero revivable, revive | `GetRevivableUnit`, `GetRevivingUnit` |
| Construction | `GetConstructingStructure`, `GetCancelledStructure`, `GetConstructedStructure` |
| Research | `GetResearchingUnit` |
| Summon, player form | `GetSummonedUnit` |
| Summon, unit form | `GetSummoningUnit` |
| Change owner | `GetChangingUnit` |
| Region | `GetEnteringUnit`, `GetLeavingUnit` |

## Getters that never give a value

| Getter | Status | What happens |
|---|---|---|
| `GetDetectedUnit`, `GetEventDetectingPlayer` | gap | Missing. |
| `GetEventTargetUnit` | gap | Missing. |
| `GetLoadedUnit`, `GetTransportUnit` | gap | Missing. |
| `GetEquippedItem`, `GetUnequippedItem`, `BlzGetEquippedItem`, `BlzGetUnequippedItem` | gap | Missing. |
| `BlzGetAbsorbingItem`, `BlzGetManipulatedItemWasAbsorbed` | gap | Missing. |
| `BlzGetStackingItemSource`, `BlzGetStackingItemTarget`, `BlzGetStackingItemTargetPreviousCharges` | gap | Missing. |
| `BlzGetTriggerSyncData`, `BlzGetTriggerSyncPrefix` | gap | Missing. |
| `GetRescuer` | stub | Always nil. |
| `GetTriggerEvalCount`, `GetTriggerExecCount` | gap | Always 0. |
| `GetTriggeringTrackable`, `GetWinningPlayer`, `GetSaveBasicFilename` | gap | Raise an error. |
| `GetTournamentFinishSoonTimeRemaining`, `GetTournamentFinishNowRule`, `GetTournamentFinishNowPlayer`, `GetTournamentScore` | gap | Raise an error. |

A missing native logs "had no native implementation" and returns nothing.
