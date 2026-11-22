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

Shop purchases by script or UI work; the stock rules are unverified. Sell and pawn events fire; `EVENT_UNIT_USE_ITEM` fires on item use.

| Native | Status | What differs |
|---|---|---|
| Destructable and item natives with a nil handle (`RemoveDestructable`, `KillDestructable`, `GetDestructableX`, `GetItemX`, `SetItemPosition`, `UnitRemoveItem` and similar) | bugfix | Setters do nothing and getters return 0, as in Warcraft III; a second `RemoveDestructable` is a no-op. |
| `ShowDestructable` | unverified | `false` hides model, selection and preview and makes it untargetable (harvest and smart orders fail); it keeps its pathing. |
| `QueueDestructableAnimation`, `SetDestructableAnimationSpeed` | unverified | Queued animation plays after the current one ends; speed persists across animation changes. Not checked visually. |
| `SetDestructableMaxLife` | unverified | Current life stays unless above the new maximum, then it is clamped; a maximum of 0 or less never kills. |
| `GetDestructableName`, `IsDestructableInvulnerable` | works | Name is the raw type name, no TRIGSTR lookup; invulnerable reads the `SetDestructableInvulnerable` flag. |
| `BlzGetItemIntegerField` with `'iarm'` and tint fields | gap | Return 0 with one log line. |
| `BlzSetItemName`, `BlzSetItemDescription`, `BlzSetItemTooltip`, `BlzSetItemExtendedTooltip`, `BlzSetItemIconPath`, `BlzGetItemTooltip`, `BlzGetItemExtendedTooltip` | works | Per-item overrides; a script icon also replaces the disabled icon. |
| `BlzItemRemoveAbility` | works | Removes the ability from the item. |
| `UnitDropItemPoint` | unverified | Returns true and drops on a later tick; false for a unit that cannot move when the point is out of drop range. |
| `BlzSetItemBooleanField` | partial | Four fields; others return false. |
| `UnitAddItem`, `UnitHasItem` | bugfix | `UnitAddItem` returns a boolean. |
| `UnitAddItemById`, `UnitAddItemToSlotById` on a dead unit | bugfix | Return nil and create no item, as measured; `UnitAddItem` returns false (inferred). |
| `UnitUseItem`, `UnitUseItemTarget` | deliberate | False when the order is refused (a potion at full life); Reforged returns true. |
| `UnitUseItemPoint` | deliberate | True when the order is accepted; WC3 always returns false. |
| `IssueNeutralImmediateOrderById` | partial | Buys items and units; no rally point or tavern revive yet. |
| `EVENT_PLAYER_UNIT_SELL`, `EVENT_PLAYER_UNIT_SELL_ITEM`, `EVENT_UNIT_SELL`, `EVENT_UNIT_SELL_ITEM` | bugfix | Fire synchronously during the purchase, on the shop and its owner; order against the pickup event is unmeasured. |
| `EVENT_PLAYER_UNIT_PAWN_ITEM`, `EVENT_UNIT_PAWN_ITEM`, `GetSellingUnit`, `GetBuyingUnit`, `GetSoldItem` | works | Fire before the drop event, inline within pawn range; seller is the hero, buyer the shop; a nested pawn of the same item is ignored. |
| Pawn payment | deliberate | Gold arrives in the same tick; Reforged pays about 0.2 s after the events on a walk-then-pawn. |
| `EVENT_UNIT_USE_ITEM`, `EVENT_PLAYER_UNIT_USE_ITEM` | bugfix | Fire on use (player event, then unit event), before the used-up drop; cast item spells fire once per landed effect. |
| Potion used by order `heal` | bugfix | Spends one charge and fires one USE. |
| `GetItemCharges` right after `UnitUseItem` | unverified | Shows the old count in the same thread; it updates later in the same tick. |
| `GetSpellAbilityId` in `EVENT_UNIT_USE_ITEM` | unverified | Returns 0. |
| Item spell cast by order string (`IssueTargetOrder(hero, "charm", u)`) | unverified | Now spends a charge at the landed effect; a cast without an effect keeps the charge. |
| `EVENT_UNIT_USE_ITEM` for powerups | unverified | Fires after the pickup events. |
| `UnitUseItemTarget` with a target far out of range | unverified | Returns false instead of walking in; WC3 behavior not confirmed. |
| `EVENT_UNIT_DROP_ITEM`, `EVENT_PLAYER_UNIT_DROP_ITEM` | unverified | Fire once per drop, before the item leaves; `UnitHasItem` is still true inside. Player event fires before the unit event, as in Reforged. |
| Drop events: what counts as a drop | bugfix | Ground drop, give, pawn or sell, drop natives, `RemoveItem`, used-up charges (charges 0) and death drops fire; slot swaps stay silent. |
| `RemoveItem` on an already removed item | bugfix | Does nothing. |
| `GetItemTypeId` after `RemoveItem` | bugfix | Returns 0 after the drop event, as in Reforged; an item a handler moved elsewhere is taken out again silently. |
| Drop event from inside its own handler | deliberate | A nested drop of the same item fires nothing; Reforged recurses. A death drop after the handler kills the dropper is silent. |
| Give item | unverified | Fires the drop event, then the receiver's pickup event. |
| `UnitRemoveItemFromSlot`, `UnitDropItemSlot`, `UnitDropItemTarget` | works | Implemented (were missing); `UnitDropItemSlot` swaps slots without an event. |
| `UnitDropItemTarget` onto a shop | works | Pawns inline within pawn range when the order would succeed, otherwise walks; a blocked nested pawn still returns true. |
| Drop natives on an undroppable item | gap | Return true. |
| Handler that kills the giver during a give | gap | The give still completes. |
| `IssueNeutralPointOrder`, `IssueNeutralPointOrderById`, `IssueNeutralTargetOrder`, `IssueNeutralTargetOrderById` | unverified | `neutralinteract` sets the shop selection for the ordering player only. |
| `IssueNeutralImmediateOrder` (string form) | gap | Not implemented on purpose. |
