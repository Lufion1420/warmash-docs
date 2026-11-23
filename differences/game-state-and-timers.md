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
| `EndGame` | gap | Leaves to the menu, no score screen. |
| `RestartGame`, `ChangeLevel` | deliberate | Leave to the menu. |
| `DisplayTimedTextFromPlayer`, `GetDefaultDifficulty`, `UnitSuspendDecay` | works | Implemented. |
| `GetSellingUnit`, `GetBuyingUnit`, `GetSoldUnit`, `GetSoldItem` | bugfix | Return the shop, buyer, new unit and item during a sale. |
| `GetRescuer` | gap | Always nil: no rescue events. |
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
