---
title: Players, game state and timers
nav_order: 5
parent: Differences from Warcraft III
---

# Players, game state and timers
{: .no_toc }

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
| Food change (`SetPlayerState` `FOOD_CAP`, `FOOD_USED`, `FOOD_CAP_CEILING`, unit death, new food building) | bugfix | A waiting train item starts and pays inside the call; its START fires player form, then unit form. Measured for `SetPlayerState`; unit-death case unverified. |
| `GetPlayerTechCount` with a food-blocked train item | bugfix | In-progress count is 0 (was -1). |
| Food checks (train, altar revive order, build, upgrade, unit sale, JSON morph, JSON ability food cost) | unverified | Use min(food cap, `FOOD_CAP_CEILING`). `ReviveHero` checks the ceiling only. |
| `PLAYER_STATE_FOOD_CAP_CEILING` default | unverified | 100 (was 101); not checked against Warcraft III. |
| Food refused by the ceiling | gap | Shows the not-enough-food message, not the max-supply one. |
| HUD resource bar food cap | gap | Shows the raw cap, not min(cap, ceiling). |
| `SetUnitOwner` and Charm: food | unverified | Food used, paid front-of-queue unit food and food made move to the new owner. Warcraft III not confirmed. |
| `SetUnitOwner` and altar-revive food, worker build reservations, tech in-progress counts | gap | These do not move to the new owner. |
| `SetUnitOwner` and upgrades | gap | Swaps no upgrades. |
| `GetUnitFoodMade` on a constructed or upgraded building | bugfix | Returns the type's supply (was 0). |
| Death or `RemoveUnit` of a constructed building | bugfix | Lowers its owner's food cap (was never released). |
| Change-owner handler that changes the owner again | bugfix | No stale change-owner event fires. |
| `RemoveUnit` on a building with a queue | bugfix | Refunds gold, lumber and food shortly afterwards without events; training as measured, research and revive unverified. |
| Food-making unit created while an item waits | unverified | `CreateUnit` places the unit, then the waiting item's START fires, still inside the call. |
| `EndGame` | gap | Leaves to the menu, no score screen. |
| `RestartGame`, `ChangeLevel` | deliberate | Leave to the menu. |
| `DisplayTimedTextFromPlayer`, `GetDefaultDifficulty`, `UnitSuspendDecay` | works | Implemented. |
| `GetSellingUnit`, `GetBuyingUnit`, `GetSoldUnit`, `GetSoldItem` | bugfix | Return the shop, buyer, new unit and item during a sale. |
| `GetRescuer` | stub | Always nil: no rescue events. |
| `GetIntegerGameState` | stub | Returns 0 for divine intervention and disconnected. |
| `ConvertMapSetting`, `ConvertMapVisibility` | stub | Always return nil. |
| `GetTournamentScore`, `GetTournamentFinishNowPlayer`, `GetTournamentFinishNowRule`, `GetTournamentFinishSoonTimeRemaining`, `GetSaveBasicFilename` | gap | Raise an error when called. |
| `PLAYER_STATE_GOLD_GATHERED`, `PLAYER_STATE_LUMBER_GATHERED` | stored-only | Written only by `SetPlayerState`, never by gameplay, so state events on them never fire in play. |
| `PauseGame`, `DisplayLoadDialog`, `PingMinimap`, `UnitAddIndicator`, `VolumeGroupSetVolume`, `AddItemToStock`, `RemoveItemFromStock`, `DestroyUbersplat`, `RemoveWeatherEffect` | stub | One log line, no effect. |

## Timers, triggers and hashtables

Timers fire at their real period, also below the 0.05 s tick.

| Native | Status | What differs |
|---|---|---|
| Repeating timer under 0.05 s | bugfix | Fires at its real period, up to 50 times per tick. |
| Spell, use-item, drop-item and pawn-item events registered on a unit and on a player | bugfix | The player event fires first whatever the registration order, as in Reforged (measured). |
| Other unit events registered on a unit and on a player (death, damage, attacked, hero level, pickup, orders, change owner, selection, construct, train and research finish, sell) | unverified | Reforged order not measured; WarMash fires the unit event first. |
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
