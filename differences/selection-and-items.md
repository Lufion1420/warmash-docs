---
title: Selection, input, items and shops
nav_order: 6
parent: Differences from Warcraft III
---

# Selection, input, items and shops
{: .no_toc }

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

Shop purchases by script or UI work; the stock rules are unverified. Sell events fire, pawn events do not.

| Native | Status | What differs |
|---|---|---|
| `BlzGetItemIntegerField` with `'iarm'` and tint fields | gap | Return 0 with one log line. |
| `BlzSetItemName`, `BlzSetItemDescription`, `BlzSetItemTooltip`, `BlzSetItemExtendedTooltip`, `BlzSetItemIconPath`, `BlzGetItemTooltip`, `BlzGetItemExtendedTooltip` | works | Per-item overrides; a script icon also replaces the disabled icon. |
| `BlzItemRemoveAbility`, `UnitDropItemPoint` | works | Drops at once within 32 units, otherwise orders a drop. |
| `BlzSetItemBooleanField` | partial | Four fields; others return false. |
| `UnitAddItem`, `UnitHasItem` | bugfix | `UnitAddItem` returns a boolean. |
| `UnitUseItem`, `UnitUseItemTarget` | deliberate | False when the order is refused (a potion at full life); Reforged returns true. |
| `UnitUseItemPoint` | deliberate | True when the order is accepted; WC3 always returns false. |
| `IssueNeutralImmediateOrderById` | partial | Buys items and units; no rally point or tavern revive yet. |
| `EVENT_PLAYER_UNIT_SELL`, `EVENT_PLAYER_UNIT_SELL_ITEM`, `EVENT_UNIT_SELL`, `EVENT_UNIT_SELL_ITEM` | bugfix | Fire synchronously during the purchase, on the shop and its owner; order against the pickup event is unmeasured. |
| `EVENT_PLAYER_UNIT_PAWN_ITEM`, `EVENT_UNIT_PAWN_ITEM` | gap | Not implemented. |
| `EVENT_UNIT_DROP_ITEM`, `EVENT_PLAYER_UNIT_DROP_ITEM` | unverified | Now fire, unit list then player list, once per drop, before the item leaves; `UnitHasItem` is still true inside. |
| Drop events: what counts as a drop | unverified | Ground drop, give, pawn or sell, `UnitDropItemPoint`, `UnitRemoveItem` and death drops fire; `RemoveItem`, used-up charges and slot swaps stay silent. |
| Drop event from inside its own handler | unverified | A nested drop of the same item, or a death drop after the handler kills the dropper, fires nothing. |
| Give item | unverified | Fires the drop event, then the receiver's pickup event. |
| `UnitRemoveItemFromSlot`, `UnitDropItemSlot`, `UnitDropItemTarget` | works | Implemented (were missing); `UnitDropItemSlot` swaps slots without an event. |
| Drop natives on an undroppable item | gap | Return true. |
| Handler that kills the giver during a give | gap | The give still completes. |
| `IssueNeutralPointOrder`, `IssueNeutralPointOrderById`, `IssueNeutralTargetOrder`, `IssueNeutralTargetOrderById` | unverified | `neutralinteract` sets the shop selection for the ordering player only. |
| `IssueNeutralImmediateOrder` (string form) | gap | Not implemented on purpose. |
