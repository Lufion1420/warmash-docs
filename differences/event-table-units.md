---
title: "Event table: unit events"
nav_order: 14
parent: Differences from Warcraft III
---

# Event table: unit events
{: .no_toc }

Every unit and player-unit event, with what WarMash fills in. Quick answers: [which getter to use](event-use.html). Other events: [player, game, frame and other events](event-table-other.html).

How to read the tables:

- **Fires**: `yes`, `partial` or `never`. A never-firing event registers without an error.
- **Unit / widget / player**: what `GetTriggerUnit`, `GetTriggerWidget` and `GetTriggerPlayer` return; "nil" means no value.
- `GetTriggerEventId` and `GetTriggeringTrigger` work in every event on this page.
- Spell getters keep their values in loops and sub-triggers, not in timer callbacks: see [details](event-use.html#spell-getters-in-loops-and-sub-triggers).

## Combat and death

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_ATTACKED`, `EVENT_PLAYER_UNIT_ATTACKED` | yes | target / target / its owner | `GetAttacker` |
| `EVENT_UNIT_DAMAGING`, `EVENT_PLAYER_UNIT_DAMAGING` | yes | target / target / its owner | All damage getters and setters |
| `EVENT_UNIT_DAMAGED`, `EVENT_PLAYER_UNIT_DAMAGED` | yes | target / target / its owner | All damage getters; `BlzSetEventDamage`; type setters act after armor |
| `EVENT_UNIT_DEATH`, `EVENT_PLAYER_UNIT_DEATH` | yes | dying / dying / its owner | `GetDyingUnit`, `GetKillingUnit` (can be nil) |
| `EVENT_UNIT_DECAY`, `EVENT_PLAYER_UNIT_DECAY` | yes | corpse / corpse / its owner | `GetDecayingUnit` |

Details:

- Damage getters: `GetEventDamage`, `GetEventDamageSource`, `BlzGetEventDamageTarget`, `BlzGetEventIsAttack`.
- Damage types: `BlzGetEventAttackType`, `BlzGetEventDamageType`, `BlzGetEventWeaponType`, each with a `BlzSetEvent...` setter.
- Attacks on trees raise no attack event. Decay fires only for raisable corpses that decay.

## Spells

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_SPELL_CHANNEL`, `EVENT_PLAYER_UNIT_SPELL_CHANNEL` | yes | caster / nil / nil | Spell getters |
| `EVENT_UNIT_SPELL_CAST`, `EVENT_PLAYER_UNIT_SPELL_CAST` | yes | caster / nil / nil | Spell getters |
| `EVENT_UNIT_SPELL_EFFECT`, `EVENT_PLAYER_UNIT_SPELL_EFFECT` | yes | caster / nil / nil | Spell getters |
| `EVENT_UNIT_SPELL_FINISH`, `EVENT_PLAYER_UNIT_SPELL_FINISH` | yes | caster / nil / nil | Spell getters |
| `EVENT_UNIT_SPELL_ENDCAST`, `EVENT_PLAYER_UNIT_SPELL_ENDCAST` | yes | caster / nil / nil | Spell getters |

Details:

- Spell getters: `GetSpellAbilityUnit`, `GetSpellAbility`, `GetSpellAbilityId`, `GetSpellTargetX`, `GetSpellTargetY`, `GetSpellTargetLoc`.
- Only the matching one of `GetSpellTargetUnit`, `GetSpellTargetItem`, `GetSpellTargetDestructable` has a value.

## Orders

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_ISSUED_ORDER`, `EVENT_PLAYER_UNIT_ISSUED_ORDER` | yes | ordered / ordered / nil | `GetOrderedUnit`, `GetIssuedOrderId` |
| `EVENT_UNIT_ISSUED_POINT_ORDER`, `EVENT_PLAYER_UNIT_ISSUED_POINT_ORDER` | yes | ordered / ordered / nil | Order getters above, `GetOrderPointX`, `GetOrderPointY`, `GetOrderPointLoc` |
| `EVENT_UNIT_ISSUED_TARGET_ORDER`, `EVENT_PLAYER_UNIT_ISSUED_TARGET_ORDER`, `EVENT_PLAYER_UNIT_ISSUED_UNIT_ORDER` | yes | ordered / ordered / nil | `GetOrderedUnit`, `GetIssuedOrderId`, target getters |

Details:

- Target getters: `GetOrderTarget`, plus the matching one of `GetOrderTargetUnit`, `GetOrderTargetItem`, `GetOrderTargetDestructable`.
- `EVENT_PLAYER_UNIT_ISSUED_UNIT_ORDER` is the same event as the target order, under a second name.

## Items and shops

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_PICKUP_ITEM`, `EVENT_PLAYER_UNIT_PICKUP_ITEM` | yes | carrier / carrier / nil | `GetManipulatingUnit`, `GetManipulatedItem` |
| `EVENT_UNIT_DROP_ITEM`, `EVENT_PLAYER_UNIT_DROP_ITEM` | yes | carrier / carrier / nil | `GetManipulatingUnit`, `GetManipulatedItem` |
| `EVENT_UNIT_USE_ITEM`, `EVENT_PLAYER_UNIT_USE_ITEM` | yes | user / user / nil | `GetManipulatingUnit`, `GetManipulatedItem` |
| `EVENT_UNIT_SELL`, `EVENT_PLAYER_UNIT_SELL` | yes | shop / shop / shop owner | `GetSellingUnit`, `GetBuyingUnit`, `GetSoldUnit` |
| `EVENT_UNIT_SELL_ITEM`, `EVENT_PLAYER_UNIT_SELL_ITEM` | yes | shop / shop / shop owner | `GetSellingUnit`, `GetBuyingUnit`, `GetSoldItem` |
| `EVENT_UNIT_PAWN_ITEM`, `EVENT_PLAYER_UNIT_PAWN_ITEM` | yes | hero / hero / its owner | `GetSellingUnit` (hero), `GetBuyingUnit` (shop), `GetSoldItem` |
| `EVENT_UNIT_STACK_ITEM`, `EVENT_PLAYER_UNIT_STACK_ITEM` | never | | |
| `EVENT_UNIT_EQUIP_ITEM`, `EVENT_PLAYER_UNIT_EQUIP_ITEM` | never | | |
| `EVENT_UNIT_UNEQUIP_ITEM`, `EVENT_PLAYER_UNIT_UNEQUIP_ITEM` | never | | |

## Heroes and summons

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_HERO_LEVEL`, `EVENT_PLAYER_HERO_LEVEL` | yes | hero / hero / its owner | `GetLevelingUnit` |
| `EVENT_UNIT_HERO_SKILL`, `EVENT_PLAYER_HERO_SKILL` | yes | hero / hero / its owner | `GetLearningUnit`, `GetLearnedSkill`, `GetLearnedSkillLevel` |
| `EVENT_PLAYER_HERO_REVIVABLE` | yes | hero / hero / its owner | `GetRevivableUnit` |
| `EVENT_UNIT_HERO_REVIVABLE` | never | | |
| `EVENT_UNIT_HERO_REVIVE_START`, `EVENT_PLAYER_HERO_REVIVE_START` | yes | hero / hero / its owner | `GetRevivingUnit` |
| `EVENT_UNIT_HERO_REVIVE_CANCEL`, `EVENT_PLAYER_HERO_REVIVE_CANCEL` | yes | hero / hero / its owner | `GetRevivingUnit` |
| `EVENT_UNIT_HERO_REVIVE_FINISH`, `EVENT_PLAYER_HERO_REVIVE_FINISH` | yes | hero / hero / its owner | `GetRevivingUnit` |
| `EVENT_PLAYER_UNIT_SUMMON` | partial | summoned / summoned / its owner | `GetSummonedUnit`, `GetSummoningUnit` |
| `EVENT_UNIT_SUMMON` | partial | summoner / summoner / owner of the summoned unit | `GetSummonedUnit`, `GetSummoningUnit` |

Details:

- Summon events fire only for some summon abilities; see [Events and event getters](event-getters.html#events-that-fire).

## Construction, training and research

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_PLAYER_UNIT_CONSTRUCT_START` | yes | structure / structure / its owner | `GetConstructingStructure` |
| `EVENT_UNIT_CONSTRUCT_CANCEL`, `EVENT_PLAYER_UNIT_CONSTRUCT_CANCEL` | yes | structure / structure / its owner | `GetCancelledStructure` |
| `EVENT_UNIT_CONSTRUCT_FINISH`, `EVENT_PLAYER_UNIT_CONSTRUCT_FINISH` | yes | structure / structure / nil | `GetConstructedStructure` |
| `EVENT_UNIT_UPGRADE_START`, `EVENT_PLAYER_UNIT_UPGRADE_START` | yes | structure / structure / its owner | None beyond the trigger getters |
| `EVENT_UNIT_UPGRADE_CANCEL`, `EVENT_PLAYER_UNIT_UPGRADE_CANCEL` | yes | structure / structure / its owner | None beyond the trigger getters |
| `EVENT_UNIT_UPGRADE_FINISH`, `EVENT_PLAYER_UNIT_UPGRADE_FINISH` | yes | structure / structure / its owner | None beyond the trigger getters |
| `EVENT_UNIT_TRAIN_START`, `EVENT_PLAYER_UNIT_TRAIN_START` | yes | building / building / its owner | `GetTrainedUnitType` |
| `EVENT_UNIT_TRAIN_CANCEL`, `EVENT_PLAYER_UNIT_TRAIN_CANCEL` | yes | building / building / its owner | `GetTrainedUnitType` |
| `EVENT_UNIT_TRAIN_FINISH`, `EVENT_PLAYER_UNIT_TRAIN_FINISH` | yes | building / building / its owner | `GetTrainedUnit`, `GetTrainedUnitType` |
| `EVENT_UNIT_RESEARCH_START`, `EVENT_PLAYER_UNIT_RESEARCH_START` | yes | building / building / its owner | `GetResearchingUnit`, `GetResearched` |
| `EVENT_UNIT_RESEARCH_CANCEL`, `EVENT_PLAYER_UNIT_RESEARCH_CANCEL` | yes | building / building / its owner | `GetResearchingUnit`, `GetResearched` |
| `EVENT_UNIT_RESEARCH_FINISH`, `EVENT_PLAYER_UNIT_RESEARCH_FINISH` | yes | building / building / its owner | `GetResearchingUnit`, `GetResearched` |

## Selection, ownership, state and detection

| Event | Fires | Unit / widget / player | Event getters with a value |
|---|---|---|---|
| `EVENT_UNIT_SELECTED`, `EVENT_PLAYER_UNIT_SELECTED` | yes | unit / unit / selecting player | None beyond the trigger getters |
| `EVENT_UNIT_DESELECTED`, `EVENT_PLAYER_UNIT_DESELECTED` | yes | unit / unit / selecting player | None beyond the trigger getters |
| `EVENT_UNIT_CHANGE_OWNER` | yes | unit / unit / new owner | `GetChangingUnit`, `GetChangingUnitPrevOwner` |
| `EVENT_PLAYER_UNIT_CHANGE_OWNER` | yes | unit / unit / previous owner | `GetChangingUnit`, `GetChangingUnitPrevOwner` |
| `EVENT_UNIT_STATE_LIMIT` | yes | unit / unit / its owner | `GetEventUnitState` |
| `EVENT_UNIT_DETECTED`, `EVENT_PLAYER_UNIT_DETECTED` | never | | |
| `EVENT_UNIT_HIDDEN`, `EVENT_PLAYER_UNIT_HIDDEN` | never | | |
| `EVENT_UNIT_RESCUED`, `EVENT_PLAYER_UNIT_RESCUED` | never | | |
| `EVENT_UNIT_LOADED`, `EVENT_PLAYER_UNIT_LOADED` | never | | |
| `EVENT_UNIT_ACQUIRED_TARGET` | never | | |
| `EVENT_UNIT_TARGET_IN_RANGE` | never | | |

Details:

- Register `EVENT_UNIT_STATE_LIMIT` with `TriggerRegisterUnitStateEvent`.
