---
title: WarMash script API (Wms)
nav_order: 3
---

# WarMash script API (Wms)

API that does not exist in real Warcraft III. Differences in natives that do exist in Warcraft III are on the [differences page](wc3-differences.html). Launch arguments have [their own page](launch-arguments.html), and so have the [QA and debug hooks](qa-hooks.html), which are test tooling and not for maps.

## Prefix rule

Decided 2026-11-16: every script native that exists only in WarMash carries the prefix `Wms`, capitalised the same way as Blizzard's `Blz` (for example `WmsDoSomething`), and every WarMash-only constant starts with `WMS_`. A map can therefore tell at a glance which calls will not work in real Warcraft III, and can guard them:

```lua
-- WmsDoSomething is a placeholder name, not a real native
if WmsDoSomething ~= nil then
    -- running in WarMash
end
```

## Natives

Call `Wms` natives from `main()` or later; they are not available during `config()`.

| Name | Signature | Behavior | Since | Stability |
|---|---|---|---|---|
| `WmsSetUnitUntargetable` | `(unit, boolean)` | While on, no other unit's attack, unit-target spell, auto-acquire or return fire can target the unit, whatever its targets-allowed say. | 2026-11-17 | experimental |
| `WmsIsUnitUntargetable` | `(unit) -> boolean` | Reads only this flag. | 2026-11-17 | experimental |
| `WmsIsUnitInCombat` | `(unit) -> boolean` | True from the first hostile action until the combat timeout runs out. | 2026-11-20 | experimental |
| `WmsGetUnitCombatTimeLeft` | `(unit) -> real` | Seconds until the unit leaves combat; 0 when not in combat. | 2026-11-20 | experimental |
| `WmsGetUnitLastCombatSource` | `(unit) -> unit` | The last enemy that put the unit in combat or refreshed it; null if gone or never in combat. | 2026-11-20 | experimental |
| `WmsSetCombatTimeout` / `WmsGetCombatTimeout` | `(real)` / `() -> real` | Map default seconds without a hostile action before leaving combat; default 5, 0 leaves at the next tick. | 2026-11-20 | experimental |
| `WmsSetUnitCombatTimeout` | `(unit, real)` | Per-unit timeout; a negative value goes back to the map default. | 2026-11-20 | experimental |
| `WmsGetUnitCombatTimeout` | `(unit) -> real` | The timeout in effect for the unit. | 2026-11-20 | experimental |
| `WmsUnitEnterCombat` / `WmsUnitLeaveCombat` | `(unit)` | Forces the state; fires ENTER or LEAVE (enter has no source). | 2026-11-20 | experimental |
| `WmsGetCombatSourceUnit` | `() -> unit` | Event response in combat events: the other unit; null outside one. | 2026-11-20 | experimental |
| `WMS_EVENT_UNIT_COMBAT_ENTER` / `_LEAVE` | unit event | Fires for the unit itself; register with `TriggerRegisterUnitEvent`. | 2026-11-20 | experimental |
| `WMS_EVENT_PLAYER_UNIT_COMBAT_ENTER` / `_LEAVE` | player unit event | Fires for any unit of the player; register with `TriggerRegisterPlayerUnitEvent`. | 2026-11-20 | experimental |
| `WMS_EVENT_UNIT_BUFF_RECEIVED` / `_BUFF_LOST` / `_ABILITY_ADDED` / `_ABILITY_REMOVED` | unit event | Fires for the unit itself when it gains or loses a buff or ability; register with `TriggerRegisterUnitEvent`. | 2026-11-24 | experimental |
| `WMS_EVENT_PLAYER_UNIT_BUFF_RECEIVED` / `_BUFF_LOST` / `_ABILITY_ADDED` / `_ABILITY_REMOVED` | player unit event | Same four events for any unit of the player; register with `TriggerRegisterPlayerUnitEvent`. | 2026-11-24 | experimental |
| `WmsGetEventAbilityId` | `() -> integer` | Event response in those events: the rawcode of the buff or ability; 0 outside one. | 2026-11-24 | experimental |
| `WmsGetEventAbility` | `() -> ability` | Event response: the ability or buff handle, also valid in the REMOVED and LOST events. | 2026-11-24 | experimental |
| `WMS_EVENT_UNIT_ABILITY_INTERVAL` | unit event | Fires for the owner of a passive ability every N seconds; N is the per-level real field `Wmsi` (0 = off); register with `TriggerRegisterUnitEvent`. | 2026-11-24 | experimental |
| `WMS_EVENT_PLAYER_UNIT_ABILITY_INTERVAL` | player unit event | Same event for any unit of the player; register with `TriggerRegisterPlayerUnitEvent`. | 2026-11-24 | experimental |
| `Wmsi` (ability real level field) | field | Interval in seconds, passive abilities only; set with `BlzSetAbilityRealLevelField(a, ConvertAbilityRealLevelField(FourCC('Wmsi')), level0Based, seconds)`. | 2026-11-24 | experimental |
| `WmsIsUnitFacingUnit` | `(unit, target, coneDegrees) -> boolean` | True when the target is inside the unit's facing cone (full angle). | 2026-11-23 | experimental |
| `WmsIsUnitFacingUnitInRange` | `(unit, target, coneDegrees, range) -> boolean` | Facing cone plus centre-to-centre range (no collision size, unlike `IsUnitInRange`). | 2026-11-23 | experimental |
| `WmsIsUnitFacingPoint` | `(unit, x, y, coneDegrees) -> boolean` | Same cone test against a point. | 2026-11-23 | experimental |
| `WmsGetUnitFacingDelta` | `(unit, x, y) -> real` | Signed degrees in (-180, 180] from the unit's facing to the point; positive is left (counter-clockwise). | 2026-11-23 | experimental |
| `WmsGetUnitFacingDiffToPoint` | `(unit, x, y) -> real` | Same value as `WmsGetUnitFacingDelta`; signed degrees in (-180, 180] from the facing to the point. | 2026-11-24 | experimental |
| `WmsGetUnitFacingDiffToUnit` | `(unit, target) -> real` | Same angle towards the target unit; positive is left (counter-clockwise). | 2026-11-24 | experimental |
| `WmsAddPlayerGold` / `WmsAddPlayerLumber` | `(player, integer) -> integer` | Adds (negative spends), clamps the result to 0..2147483647, returns the change applied; HUD updates, no floating text. | 2026-11-23 | experimental |
| `WmsCanPlayerAfford` | `(player, gold, lumber) -> boolean` | True if the player holds both amounts; spends nothing; a cost of 0 or less is always covered. | 2026-11-23 | experimental |
| `WmsCreateRectCentered` | `(cx, cy, width, height) -> rect` | New rect from a centre and size; a negative, zero or NaN size becomes 0. | 2026-11-23 | experimental |
| `WmsSetRectCentered` | `(rect, cx, cy, width, height)` | Moves and resizes an existing rect the same way; null rect: no-op. | 2026-11-23 | experimental |
| `WmsGetRectWidth` / `WmsGetRectHeight` | `(rect) -> real` | Size of the rect; null rect gives 0. | 2026-11-23 | experimental |
| `WmsIsPointInRect` | `(rect, x, y) -> boolean` | Min and max edges count as inside, like `RectContainsCoords` in Blizzard.j. | 2026-11-23 | experimental |
| `WmsIsUnitInRect` | `(unit, rect) -> boolean` | Tests the unit's centre point; dead units count. Null unit or rect: false. | 2026-11-23 | experimental |
| `WmsCreateCircleRegion` | `(x, y, radius) -> region` | New region shaped as a circle; works with the normal region events and natives. | 2026-11-23 | experimental |
| `WmsSetCircleRegion` | `(region, x, y, radius)` | Moves and resizes a circle region; units standing there enter or leave at once. | 2026-11-23 | experimental |
| `WmsIsCircleRegion` | `(region) -> boolean` | True for a live circle region; false for a rect region or null. | 2026-11-23 | experimental |
| `WmsGetCircleRegionX` / `Y` / `Radius` | `(region) -> real` | Centre and radius of the circle; 0 for a rect region or null. | 2026-11-23 | experimental |
| `WmsIsPointInCircleRegion` | `(region, x, y) -> boolean` | Point test, distance to the centre at most the radius; false for a removed, rect or null region. | 2026-11-23 | experimental |
| `WmsSetUnitGeosetVisible` | `(unit, integer geosetIndex, boolean visible)` | Hides or shows one part (geoset) of the unit's current model; drawing only. | 2026-11-23 | experimental |
| `WmsIsUnitGeosetVisible` | `(unit, integer geosetIndex) -> boolean` | The flag the script set; true by default. Not a test of what is drawn. | 2026-11-23 | experimental |
| `WmsGetUnitGeosetCount` | `(unit) -> integer` | Number of geosets you can index on the unit's current model. | 2026-11-23 | experimental |
| `WmsCameraFrameUnit` | `(player, unit, distance, angleOffsetDeg, height, duration)` | Moves the camera to frame a unit once, over the duration (0 = instant). | 2026-11-23 | experimental |
| `WmsCameraFollowUnit` | `(player, unit, distance, angleOffsetDeg, height)` | Keeps the camera framed on a unit every frame until the follow ends. | 2026-11-23 | experimental |
| `WmsCameraStopFollow` | `(player)` | Ends a follow; the camera stays where it is. | 2026-11-23 | experimental |
| `WmsSetCameraLocked` | `(player, boolean)` | Locks or unlocks the player's own camera input; script camera calls still work. | 2026-11-23 | experimental |
| `WmsIsCameraLocked` | `(player) -> boolean` | The lock flag kept for that player. | 2026-11-23 | experimental |
| `WmsResetCamera` | `(player)` | Restores the map's default camera setup and ends a follow or timed pan; the target does not move. | 2026-11-23 | experimental |
| `WmsSetQuestLogHidden` | `(player, boolean)` | Hides the Quests button for that player; F9 and clicks open nothing and an open dialog closes. | 2026-11-24 | experimental |
| `WmsIsQuestLogHidden` | `(player) -> boolean` | The flag kept for that player; false for null or never set. | 2026-11-24 | experimental |
| `WmsSetDestructableTypeOpensPathing` | `(integer destructableId, boolean flag)` | Lets a walkable destructable type open blocked terrain under its bridges; per game, default off. | 2026-11-25 | experimental |
| `WmsIsDestructableTypeOpensPathing` | `(integer destructableId) -> boolean` | The type's opt-in flag; false for unknown or non-walkable types. | 2026-11-25 | experimental |
| `WmsSetDoodadTypeWalkable` | `(integer doodadId, boolean flag)` | Gives placed doodads of the type a walk surface at model height; per game, default off. | 2026-11-25 | experimental |
| `WmsIsDoodadTypeWalkable` | `(integer doodadId) -> boolean` | The type's walkable flag; false for an unknown id. | 2026-11-25 | experimental |
| `WmsSetDoodadTypeOpensPathing` | `(integer doodadId, boolean flag)` | Lets a walkable doodad type open water, cliff and gap cells under it; default off. | 2026-11-25 | experimental |
| `WmsIsDoodadTypeOpensPathing` | `(integer doodadId) -> boolean` | The stored opt-in flag, also while the type is not walkable; false for an unknown id. | 2026-11-25 | experimental |
| `WmsAddBridgeDeck` | `(destructable, rect rampA, rect rampB) -> boolean` | Declares a walkable destructable's deck and two ramps, so units can walk over and under it. | 2026-11-26 | experimental |
| `WmsRemoveBridgeDeck` | `(destructable) -> boolean` | Removes the deck; false if the destructable has none or is null. | 2026-11-26 | experimental |
| `WmsIsBridgeDeck` | `(destructable) -> boolean` | True while a deck is declared, even if it is currently off. | 2026-11-26 | experimental |
| `WmsGetUnitPathingLayer` | `(unit) -> integer` | 1 while the unit is on a deck, else 0 (also for null). | 2026-11-26 | experimental |
| `WmsSetUnitPathingLayer` | `(unit, integer layer) -> boolean` | Puts the unit on (1) or under (0) the deck without moving it. | 2026-11-26 | experimental |
| `WmsAddDoodadBridgeDeck` | `(integer doodadId, real x, real y, rect rampA, rect rampB) -> boolean` | Declares a deck and two ramps on the placed walkable doodad near the point. | 2026-11-26 | experimental |
| `WmsRemoveDoodadBridgeDeck` | `(integer doodadId, real x, real y) -> boolean` | Removes that doodad's deck; false if there is none. | 2026-11-26 | experimental |
| `WmsIsDoodadBridgeDeck` | `(integer doodadId, real x, real y) -> boolean` | True while that doodad has a declared deck, even if it is currently off. | 2026-11-26 | experimental |
| `WmsAttachUnitToUnit` | `(unit, unit holder, string point) -> boolean` | Holds the unit at the holder's attachment point; its position follows the holder. | 2026-11-26 | experimental |
| `WmsAttachUnitToEffect` | `(unit, effect holder, string point) -> boolean` | Draws the unit at the effect's attachment point; its gameplay position does not change. | 2026-11-26 | experimental |
| `WmsDetachUnit` | `(unit)` | Releases the unit from a unit or effect holder. | 2026-11-26 | experimental |
| `WmsAttachEffectToUnit` | `(effect, unit holder, string point) -> boolean` | Moves the effect onto the unit's attachment point, like `AddSpecialEffectTarget`. | 2026-11-26 | experimental |
| `WmsAttachEffectToEffect` | `(effect, effect holder, string point) -> boolean` | Moves the effect onto another effect's attachment point. | 2026-11-26 | experimental |
| `WmsDetachEffect` | `(effect)` | Takes the effect off its holder and puts it back in the world. | 2026-11-26 | experimental |
| `WmsGetUnitAnimationCount` | `(unit) -> integer` | Number of animations on the unit's current model. | 2026-11-26 | experimental |
| `WmsGetUnitAnimationName` | `(unit, integer index) -> string` | Model name of the animation at the 0-based index. | 2026-11-26 | experimental |
| `WmsGetUnitAnimationIndex` | `(unit, string name) -> integer` | Index of the animation `SetUnitAnimation` resolves the name to, else -1. | 2026-11-26 | experimental |
| `WmsGetUnitAnimationDuration` | `(unit, string name) -> real` | Length in seconds of the named animation. | 2026-11-26 | experimental |
| `WmsGetUnitAnimationDurationByIndex` | `(unit, integer index) -> real` | Length in seconds of the animation at the index. | 2026-11-26 | experimental |
| `WmsGetEffectAnimationCount` | `(effect) -> integer` | Number of animations on the effect's model. | 2026-11-26 | experimental |
| `WmsGetEffectAnimationName` | `(effect, integer index) -> string` | Model name of the animation at the 0-based index. | 2026-11-26 | experimental |
| `WmsGetEffectAnimationIndex` | `(effect, string name) -> integer` | Index the name resolves to with the effect's sub-animations, else -1. | 2026-11-26 | experimental |
| `WmsGetEffectAnimationDuration` | `(effect, string name) -> real` | Length in seconds of the named animation. | 2026-11-26 | experimental |
| `WmsGetEffectAnimationDurationByIndex` | `(effect, integer index) -> real` | Length in seconds of the animation at the index. | 2026-11-26 | experimental |
| `WmsPlayEffectAnimationByIndex` | `(effect, integer index)` | Plays exactly that animation on the effect. | 2026-11-26 | experimental |
| `WmsSetUnitSkinKeepState` | `(unit, integer skinId)` | Like `BlzSetUnitSkin`, but keeps the unit's script look and current animation. | 2026-11-27 | experimental |
| `WmsSetUnitPortraitTexture` | `(unit, string path)` | Shows a 2D image instead of the 3D model in this unit's portrait. | 2026-11-27 | experimental |
| `WmsSetUnitTypePortraitTexture` | `(integer unitTypeId, string path)` | Same for every unit of the type without its own image. | 2026-11-27 | experimental |
| `WmsSetUnitPortraitFlipbook` | `(unit, string path, integer columns, integer rows, real framesPerSecond)` | Plays an image grid as a looping animation in this unit's portrait. | 2026-11-27 | experimental |
| `WmsSetUnitTypePortraitFlipbook` | `(integer unitTypeId, string path, integer columns, integer rows, real framesPerSecond)` | Same for every unit of the type without its own image. | 2026-11-27 | experimental |

Details for ability interval events:
- Passive abilities only (built-in passives, auras, ability-builder passives). Handlers use `GetTriggerUnit`, `GetTriggerPlayer`, `WmsGetEventAbilityId`, `WmsGetEventAbility`.
- The first event comes one interval after the ability becomes active. Read the field back with `BlzGetAbilityRealLevelField`. Not a JASS constant: use `ConvertAbilityRealLevelField(FourCC('Wmsi'))`.
- No events while the ability is disabled, or the unit is dead, under construction, upgrading or an entangled gold mine; the timer restarts after those gaps. A paused unit or a script-hidden ability keeps firing.
- At most one event per tick, no catch-up. The interval rounds to the nearest tick (50 ms), minimum 1 tick; 0 turns it off.
- A level change keeps the phase and re-reads the interval; a morph keeps the timer of kept abilities. Setting only `Wmsi` does not re-run the ability's level-change code.
- No ability-JSON option for the interval yet.

```lua
BlzSetAbilityRealLevelField(abil, ConvertAbilityRealLevelField(FourCC('Wmsi')), 0, 2.0)
```

Details for the quest log:
- The flag is stored for every player; only the local player's HUD changes. `FlashQuestDialogButton` and `ForceQuestDialogUpdate` do not open the dialog while hidden. Null player: no-op.
- Un-hiding shows the button again in its `QuestSetEnabled` state. If the map hid the button with `BlzFrameSetVisible`, un-hiding shows it anyway.
- `BlzFrameSetVisible(button, true)` while hidden shows the button, but it stays unclickable.

Details for terrain opening:
- Applies at once to existing destructables of the type (pre-placed or script-created) and to later ones. `false` closes the cells again.
- Cells under non-black pixels of the alive pathing texture become walkable, swimmable, buildable and flyable. Rails and other overlays then block on top.
- Open only while alive and shown: kill, hide and remove close; revive and show reopen. Overlapping bridges keep a cell open while any covers it.
- Never opens cells outside the playable area or boundary cells. Units on closing cells stay alive and in place; a later move order walks them off.
- Non-walkable type: no effect, one warning per type. Unknown id: no effect, a warning on every call.
- Tree-bridge types with swapped textures never become walkable over water.
- For one tick after `RemoveDestructable`, the removed bridge's destroyed-state blocking still applies.

```lua
local bridgeId = FourCC('B000') -- your walkable destructable type
WmsSetDestructableTypeOpensPathing(bridgeId, true)
print(WmsIsDestructableTypeOpensPathing(bridgeId)) --> true
```

Details for walkable doodads:
- Applies to already placed plain doodads. Terrain (cliff-type) doodads are not supported: the call is stored and has no effect.
- Turning walkable off removes the surfaces and leaves units where they are; a unit over water drops to the bed.
- A doodad whose model file is missing gets no walk surface, but opening and deck declaring still work on it.
- Opening works only while the type is walkable. Blocking pixels, such as rails, still block, except under a declared doodad deck.
- A type without a pathing texture (`none`, `_`, empty or a missing file) opens nothing. An unknown id is logged once per id per game.
- The Doodads.slk `walkable` column has no effect, as in Warcraft III.
- Height comes from each player's model files, as for walkable destructables. There is one height per point.
- Many surfaces stacked on one point are slow: about 68 µs per height lookup with 2000 stacked, under 1 µs spread out.
- Not yet checked in game: items, effects, projectiles, clicks on a doodad walk surface, tilted or unevenly scaled doodads on slopes.

```lua
local deckId = FourCC('D000') -- your doodad type; call from main() or later
WmsSetDoodadTypeWalkable(deckId, true)
WmsSetDoodadTypeOpensPathing(deckId, true)
```

Details for bridge decks (declaring):
- The deck is the cells under pathing texture pixels that are neither black nor red. Ramps are the cells centred in each rect.
- Needs two separate ramps touching the deck, each at least 2 cells (64 units) deep.
- A ramp fits a unit when `2 * collision size <= 32 * depth in cells`: up to 32 needs 2 cells, 40 or 48 need 3, 56 needs 4.
- Deck cells are walkable on top, even over water. Ramp cells and the space under the deck use the terrain's pathing.
- Decks may not share cells or touch another deck; two decks' ramps may touch each other.
- False plus a log line: destructable null, removed, not walkable, already declared or without deck pixels.
- Also false and logged: a ramp null, empty or off the map, a broken deck rule, or a building on deck, ramp or rails.

Details for bridge decks (in play):
- Walking units change layer only at the ramps. A clicked or script point order onto a deck cell targets the top of the deck.
- Units on the deck do not collide with units under it.
- New units on a deck cell start on the deck. Moves over 256 onto it land on top; shorter ones follow the walking rule.
- Flyers and ships ignore layers.
- The deck is off while the destructable is dead or hidden; revive and show bring it back. `RemoveDestructable` removes it.
- When it goes off, units on it drop to the ground alive where they are, even over water.
- On return it lifts only the units it dropped that have not moved or died. The first activation lifts every ground unit on its deck cells.
- A deck that cannot come back (building or other deck in the way) stays off until the next revive or show.
- `WmsSetUnitPathingLayer` returns false for null, flyers, ships, structures, 1 off a deck cell, or while no deck is active.
- Layer numbers other than 0 and 1 return false and are logged.
- A unit under an active deck stands on the ground, hidden by the bridge model. Its health bar shows through, its selection circle does not.
- Ramps must reach where the model meets the ground; short ramps make ground units sink into the slope, then jump up.
- Model ends that miss the ground cause height jumps at the deck ends: about 195 on a stock bridge at flat-ground height.
- Projectiles launch from the height of the unit at the launch point, also bounces and Storm Bolt.
- Removing the deck declaration turns the bridge back into an ordinary single-level walkable surface.

Limits while a deck is active:
- Build orders are refused on deck, ramp and rail cells. Buildings made with `CreateUnit` are still allowed there.
- Ancients cannot root where their footprint covers a deck or ramp cell.
- Items created, moved, dropped or already lying on deck or ramp cells go to the nearest spot off them. An item moved onto the rails is drawn on top of the bridge, but only a ground-level unit can pick it up.
- Destructable rails no longer block walking underneath, but still block building and, on vision-blocking bridges, sight.
- On bridges pre-placed over water or cliffs in the World Editor, the terrain pathing under the deck is lost.
- Units with collision size 40 or more can get stranded on 2-cell ramps when created or teleported onto a deck.
- A portrait click on a unit sends a plain point, so on a two-layer point the deck wins.

Details for orders and actions across levels:
- Patrol and guard return points keep the unit's level.
- Orders on a unit walk to its level and track changes. A follower is with its target only where they could touch.
- Melee attacks (instant attacks up to range 128 included) need the same level.
- So do item pickup, give, drop and pawn, repair, building, gold-mine harvest and returning resources.
- So do load, unload, way gates, shop service and unit merging.
- Ranged attacks, spells, auras, area effects, script natives and tree harvest ignore levels.
- Auto-acquire and retaliation prefer a touchable target at equal priority. Units that cannot walk skip untouchable ones.
- A target that cannot be reached is given up like any unreachable target.
- Clicking picks the nearest unit hit; a declared bridge never hides a unit from clicks.

Details for doodad decks:
- The doodad is the placed plain doodad of that type nearest to the point, within 128; a tie takes the first placed.
- The type must be walkable (`WmsSetDoodadTypeWalkable`). Terrain (cliff-type) doodads are not supported.
- Deck, ramp, deck-to-deck, failure and limit rules are the same as for destructable decks.
- Add also returns false and logs for an unknown type, no doodad near the point or no pathing texture.
- Turning the type walkable off drops units to the ground alive and keeps the declaration.
- Turning it on brings the deck back and lifts only the dropped units that have not moved or died.
- A deck that cannot come back stays off until the type is switched off and on again.
- While the deck is on, rails stop blocking walking only over dry, level ground without ramps.
- A rail cell stays blocked if its 128 tile has any water (also shallow, so shores), a ramp or two cliff levels.
- Rails outside the playable area also stay blocked. Rail cells always stay unbuildable.
- `WmsSetDoodadTypeOpensPathing` still opens the whole pathing texture, water and cliffs included.

```lua
-- bridge: a walkable destructable; westRamp, eastRamp: rects over its two ends
if WmsAddBridgeDeck(bridge, westRamp, eastRamp) then
    local u = CreateUnit(Player(0), FourCC('hfoo'), GetWidgetX(bridge), GetWidgetY(bridge), 0)
    print(WmsGetUnitPathingLayer(u))    --> 1 if that point is a deck cell
    WmsSetUnitPathingLayer(u, 0)        -- now under the deck
end

-- doodad deck: x, y at (or within 128 of) a placed doodad of your type
local deckId = FourCC('D000')
WmsSetDoodadTypeWalkable(deckId, true)
print(WmsAddDoodadBridgeDeck(deckId, x, y, westRamp2, eastRamp2))
WmsSetDoodadTypeWalkable(deckId, false) -- deck off, still declared
print(WmsIsDoodadBridgeDeck(deckId, x, y)) --> true if the Add succeeded
```

Details for attaching:
- Point names match as in `AddSpecialEffectTarget`, including its fallback; nil means origin. Null handles never raise an error.
- Warcraft III has none of these natives.

Details for units held by units:
- Drawn at the point, moving with it; facing, scale and animation stay the unit's own, and fly height does not affect drawing.
- Selection circle, health bar and clicks follow the drawn unit.
- Gameplay x, y follow the topmost holder every tick, overriding script moves; deck layer follows the holder and region events fire.
- The held unit has no collision and cannot walk. It can attack and cast; a move order is kept and resumes.
- It stays targetable at its gameplay position. Its projectiles start there; projectiles aimed at it fly to the drawn height.
- Chains work, and attaching a held unit again changes holder and point.
- False for cycles, itself, a held building, a dead or hidden holder, a dead unit, or either unit inside a transport.

Details for releasing a unit held by a unit:
- Released by `WmsDetachUnit`, death or removal of either unit, a hidden or loaded holder, or the held unit loading or becoming a building.
- On death the release happens before death triggers run. A dead, loaded or removed held unit is not moved.
- Otherwise it goes to the nearest standable spot by the holder, on the holder's layer first, kept inside the map.
- If there is none, it goes near the spot where it was grabbed, else onto that spot.
- Carrying a unit across cliffs or rivers and dropping it therefore works. Pause and stun are not changed.

Details for units held by effects:
- Drawing only: set the gameplay position from the script.
- True for any living unit that is not a building or in a transport, even with a null or broken effect (drawn normally).
- The call also releases the unit from a unit holder, placed as on any release.

Details for attached effects:
- The effect inherits the point's rotation and scale; position, orientation and alpha natives do nothing while it is attached.
- `WmsDetachEffect`, or destroying the holder effect, returns it to its earlier point, else to the former holder's x, y.
- False for a null, destroyed or model-less effect or holder effect, a null or removed holder unit, itself or a cycle.
- The results of `WmsAttachEffectToUnit` and `WmsAttachEffectToEffect` differ per player: do not branch gameplay on them.
- A holder unit that changes model (skin, morph, Polymorph) keeps attached effects and held units on the matching point, else origin.
- A later model that has the point gets them back there. Reforged behaviour is not confirmed.
- A destroyed effect whose holder unit was removed is freed on the next frame; `WmsDetachEffect` then cannot bring its death back.

```lua
if WmsAttachUnitToUnit(victim, carrier, "hand right") then
    -- later: drop the victim next to the carrier
    WmsDetachUnit(victim)
end
```

Details for animation queries:

- Index is 0-based, as in `SetUnitAnimationByIndex`. Lengths are at speed 1, ignoring attack speed, blend time and time scale.
- Bad input reads 0, "" or -1; an invalid play does nothing. The getters change nothing playing.
- Units: dead, hidden, loaded and off-screen units answer, also right after a morph or skin change. Removed units read 0, even in removal handlers.
- Units: names resolve with the unit's animation tags and spell fallback. Effects use their sub-animations, without the spell fallback.
- A nil, unknown or unmatched name gives -1, although `SetUnitAnimation` may still play an untagged animation for an unknown name.
- Several variants: the lowest index is read (maybe a rare variant), with no random draw. Play that index to match the length.
- Effects: null, placeholder and freed effects read empty. Living effects whose holder unit was removed still answer.
- `WmsPlayEffectAnimationByIndex` follows `BlzPlaySpecialEffect`: Stand and Walk repeat; others play once, then Stand or hold.
- On a destroyed effect that is still there, the played animation runs, then the effect leaves.
- Values match across players only with identical model files; per-player paths, skins or data can desync gameplay timed from them.

```lua
local index = WmsGetUnitAnimationIndex(u, "attack")
if index >= 0 then
    SetUnitAnimationByIndex(u, index)
    local seconds = WmsGetUnitAnimationDurationByIndex(u, index)
end
```

Details for `WmsSetUnitSkinKeepState`:

- Kept: `SetUnitScale` scale, vertex colour, `SetUnitColor`, `SetUnitTimeScale`, animation properties, look-at, current animation and queue.
- Not kept: `SetUnitBlendTime` and the position inside the animation. `BlzSetUnitSkin` resets are unchanged.
- A null unit, an unknown skin id or the current skin id does nothing.

```lua
WmsSetUnitSkinKeepState(u, FourCC("hkni"))
```

Details for 2D portraits:

- Local drawing only: for a per-player portrait, call inside a `GetLocalPlayer` check. There are no getters.
- A unit's own image beats its type's image. `""`, nil or `"NONE"` removes that entry, whatever the grid.
- The type is the current `GetUnitTypeId`. Polymorph keeps it in WarMash, so the original type's image stays; a skin change keeps it too.
- A unit's own image stays through model changes and revives, until the unit is removed.
- Only the in-game unit portrait changes. Talking heads stay 3D on purpose; items and destructables stay 3D.
- The image is centred on black at its own aspect ratio.
- A `.png` is read by its exact path. Other paths load like HUD textures.
- A `.png` must be real PNG data: at most 4096x4096 pixels in total, 8192 per side and 96 MB.
- A failed or refused load leaves the 3D model; each exact path spelling is tried and logged once per map.
- A unit's own image that failed to load shows the 3D model, not the type's image.
- Flipbook frames run row by row from the top left. A rate of 0 or less shows the first frame.
- Columns and rows are 1 to 256, at most 4096 frames; otherwise the call is ignored and logged once per native.
- A flipbook restarts when the shown unit or its image changes; setting the same values again does not restart it.
- Null or removed unit, or a map without a HUD: no-op.

```lua
local u = CreateUnit(Player(0), FourCC("Hpal"), 0, 0, 0)
WmsSetUnitTypePortraitTexture(FourCC("Hpal"), "war3mapImported\\paladin.png")
if GetLocalPlayer() == Player(0) then
    WmsSetUnitPortraitFlipbook(u, "war3mapImported\\paladin_sheet.blp", 4, 2, 8.0)
end
-- later: remove the unit's own image, the type image shows again
WmsSetUnitPortraitTexture(u, nil)
```

Details for facing:
- Cone: true when `|delta| <= cone/2` (inclusive); `cone >= 360` is always true, a negative cone false. Angles: 0 = east, 90 = north.
- A point on the unit's own position, and `unit == target`, counts as facing (delta 0), so a group-enum cone filter includes the caster; exclude it in the script. A negative range is never met.
- Null or dead unit: false, delta 0. The `DiffTo` natives also give 0 for a null or dead target and for a target on the same position.

Details for resources:
- Goes through the same player gold and lumber setter as `SetPlayerState`. Null player: 0 / false.
- WarMash has no resource cap and `SetPlayerState` is unclamped; only these natives clamp.

```lua
if WmsCanPlayerAfford(Player(0), 100, 0) then WmsAddPlayerGold(Player(0), -100) end
print(WmsIsUnitFacingPoint(u, 0, 0, 90))
```

Details for circle regions:
- A circle region is a normal region handle: `TriggerRegisterEnterRegion`/`LeaveRegion`, `GetTriggeringRegion`, `GetEnteringUnit`, `GetLeavingUnit` and `IsUnitInRegion` work unchanged.
- A unit is inside when its collision rect touches the circle (the rule rects use); `IsPointInRegion` and `WmsIsPointInCircleRegion` test a point.
- Units already inside at creation get no enter event. `WmsSetCircleRegion` fires enter and leave at once for standing units (`RegionAddRect`/`RegionClearRect` stay silent, as measured in Reforged).
- Called inside an enter or leave trigger, the shape changes at once, but unit events and the tree removal of `RemoveRegion` wait until that unit's region check ends.
- Dead and hidden units (also loaded into a transport) are skipped when the circle moves or resizes. Not confirmed against Warcraft III.
- A negative or NaN radius becomes 0; values are limited to plus or minus 1e6.
- `RegionAddRect`, `RegionClearRect` and `AddCell` on a circle region do nothing. A removed circle fires nothing and fails point checks. A null or rect handle gives false, 0 or a no-op.

```lua
local zone = WmsCreateCircleRegion(0, 0, 400)
local t = CreateTrigger()
TriggerRegisterEnterRegion(t, zone, nil)
TriggerAddAction(t, function() print(GetEnteringUnit()) end)
WmsSetCircleRegion(zone, 256, 0, 600) -- units now inside enter at once
```

Details for geosets:
- Index is 0-based among the drawn parts (level of detail 0). Lower-detail geosets are skipped, so on HD models it is not always the raw index in the model file.
- Only drawing changes: collision, attachments, emitters and geoset animations keep running. A part with geoset-animation alpha 0 stays invisible even when the flag is true.
- The state is kept per model and re-applied after animation and selection changes.
- A model change (skin, morph, Polymorph) draws the new model whole; switching back hides the old parts again.
- The selection portrait is unaffected. Local only: not synced in multiplayer.
- Null or removed unit, or a bad index: no-op, reads false (visible) or 0 (count).

```lua
WmsSetUnitGeosetVisible(u, 2, false)
print(WmsIsUnitGeosetVisible(u, 2), WmsGetUnitGeosetCount(u)) --> false  <count>
```

Details for camera helpers:
- Angle offset is in degrees: 0 puts the eye in front of the unit looking at its face, 180 behind it. Height is the eye height above the target; distance is the 3D eye-to-target distance.
- Null or non-finite arguments do nothing. Only the local player's camera is moved; for another player the call does nothing.
- Follow reads the drawn unit and snaps on its first frame (no easing); it keeps following when the unit's drawn model is replaced. It ends on unit removal or decay, `WmsCameraStopFollow`, `WmsResetCamera` or `SetCameraTargetController(unit)`.
- Height uses the exact ground while following, then eases over 0.5 s to the smoothed ground; instant camera jumps snap.
- While following, player input cannot change the framing, and script position or pan calls are overridden each frame.
- The lock blocks edge scroll, arrow keys, Insert and Delete, the wheel, minimap click and drag, middle-mouse drag, the hero-centre hotkey and holding the portrait. Script camera calls still work. The flag is kept per player.

```lua
WmsCameraFrameUnit(Player(0), u, 400, 0, 120, 1.0) -- look at its face
WmsSetCameraLocked(Player(0), true)
WmsCameraFollowUnit(Player(0), u, 600, 180, 250)   -- behind it
-- later
WmsCameraStopFollow(Player(0)); WmsSetCameraLocked(Player(0), false)
```

Details for `WmsSetUnitUntargetable`:
- Refused orders show the error "unable to target this unit"; a right-click by another unit becomes follow. Dummy casters and autocast are blocked too.
- Not affected: utility orders (load, repair, Moon Well, harvest, mounting, Mass Teleport, Entangle Gold Mine), area effects, script damage and missiles already in flight.
- Works from Lua and JASS; a map function with the same name shadows it.

```lua
local u = CreateUnit(Player(0), FourCC("hfoo"), 0, 0, 0)
WmsSetUnitUntargetable(u, true)
print(WmsIsUnitUntargetable(u)) --> true
```

Details for buff and ability events:
- Fire when the outermost add or remove ends, in the order the list changed; the unit is the triggering unit. Script-added buffs and abilities count.
- A timed-buff refresh fires nothing; replacing a buff with the same id fires LOST, then RECEIVED. A change a handler undoes before its event fired cancels both events.
- Nothing fires for unit creation, built-in abilities (Move, Attack, Hero, Inventory, Rally), a building's in-progress build ability, or units being removed.
- Death clears buffs: LOST fires after the DEATH trigger conditions and before the DEATH actions. A unit type change fires ABILITY_REMOVED for lost abilities, then ADDED for new ones, buffs kept.
- Handlers nest up to 32 deep; deeper changes are logged and skipped. Warcraft III has no such events (maps poll `GetUnitAbilityLevel`).

Details for combat state:
- Enters combat (both units, once per ENTER): hostile damage above 0, an attack swing, a hostile unit-target spell (also without damage), enemy buffs and enemy auras. Each repeat refreshes the timer.
- Does not enter: allies, self, zero damage. An invulnerable target enters neither unit; an invulnerable attacker hitting a vulnerable enemy enters both.
- Death has no special handling: the timer runs and the corpse gets its LEAVE. A dead unit still in combat when removed (decay, explode, `RemoveUnit`) gets one LEAVE at removal, on the next update; a living unit removed by `RemoveUnit` gets none.
- Limits: buff-only area spells without damage or a unit target are not covered; during the removal LEAVE the unit is already out of the handle table, so `ReviveHero`-style lookups fail; if a source's ENTER handler removes the target, the target's ENTER still fires and no LEAVE follows.
- Handlers run inline up to their first wait. A LEAVE handler may remove other units safely. A null unit is a no-op (getters return false, 0 or null).

```lua
local t = CreateTrigger()
TriggerRegisterPlayerUnitEvent(t, Player(0), WMS_EVENT_PLAYER_UNIT_COMBAT_LEAVE, nil)
TriggerAddAction(t, function() print(GetTriggerUnit(), WmsGetCombatSourceUnit()) end)
```

`Stability` is `experimental` (may change without notice) or `stable`.

## Model effect sidecar (`.wmfx.json`)

An optional JSON file that changes how WarMash draws parts of a model. Warcraft III does not support it and shows the plain MDX model (not checked in game). Since 2026-11-25, experimental.

| Topic | Rule |
|---|---|
| File name | `Name.wmfx.json` beside `Name.mdx`; import both into the same folder. |
| Models | MDX version 800 only. |
| Required | Root `version` = 1; per surface `geoset`, `layer` (inside its material) and `vertexCount` (must match). |
| Texture paths | Resolved like MDX texture paths, from the map or data-source root, never the model's folder. |
| Texture names | Give each texture a path, without extension, that no other map image has, such as suffix `_sa`. |
| Errors | The model still loads; the bad file, surface or field is skipped and logged with `[wmfx]`. |
| World Editor, HiveWE | Whether they keep the imported file is not confirmed. |

Fields that work today, per entry in `surfaces`:

| Field | Effect (a missing field keeps the MDX value) |
|---|---|
| `hideOtherLayers` | Draws no other layer of this geoset's material, on this geoset only. |
| `twoSided`, `depthTest`, `depthWrite`, `unshaded`, `unfogged` | Booleans that override the layer flags. |
| `textures` with `role` `color0` | Replaces the layer texture; takes `path`, `uvSet` (default 0), `wrapU`, `wrapV`. |
| `textures` with `role` `color1` | Second texture, combined with the colour texture in one pass; same entry keys as `color0`. |
| `combine` | How `color1` combines: `over`, `mix`, `multiply` (default) or `multiplyAddAlpha`. |
| `weight0`, `weight1`, their tracks | Combine weights, 0 to 1, default 1; the track wins. |
| `tracks.uv1Offset`, `tracks.uv1Scale` | Moves and scales `color1`: `uv * scale + offset`. |
| `alphaRef`, `tracks.alphaRef` | Discards pixels whose tested alpha is at or below it; 0 to 1; the track wins. |
| `textures` with `role` `falloff` | View-angle ramp (`path` only); its green channel multiplies alpha, RGB is untouched. |
| `falloff`, `tracks.falloff` | Picks the ramp row; 0 (default, top) to 1; the track wins. |
| `vertexColor` | Per-vertex RGBA, integers 0 to 255 (255 = 1.0), multiplying colour and alpha. |
| `positionCrc32` | Optional check that `vertexColor` matches the geoset's vertex positions. |
| `tracks.opacity` | Replaces the layer alpha; 0 to 1. |
| `tracks.color` | RGB multiplier, 0 or more; values above 1 brighten. |
| `tracks.uv0Offset` | Moves the texture: `uv * scale + offset`. |
| `tracks.uv0Scale` | Scales U and V separately from UV (0, 0); zero and negative values work. |

Parsed but ignored for now: `normal` textures, `softDistance`, ribbon tracks, and the `glare` and `distortion` tracks. A surface with `pass` `distortion` is skipped entirely.

Details:
- Tracks are `{"interp": "linear" or "step", "keys": [[ms, value...], ...]}`; `ms` is MDX time, ascending. Default `linear`.
- Only keys inside the playing sequence count; the first and last of them are held. Without any, the MDX value applies.
- `alphaRef` is tested before opacity, geoset and effect alpha, so fading keeps the shape. It replaces the model's fixed cutoffs; 0 discards nothing.
- The tested alpha is texture alpha (combined with `color1`) x falloff x vertex alpha; shadows leave out falloff.
- Without an `alphaRef` field and a key in the playing sequence, the model's fixed cutoffs apply and see vertex alpha.
- Falloff ramp columns: left is a surface seen edge-on, right faces the camera, measured against the camera view axis.
- Falloff applies before `alphaRef` and the fixed cutoffs, not to shadows; the ramp is always clamped (`uvSet`, wrap ignored).
- A `falloff` field or track without a falloff texture does nothing; a missing or undecodable one is dropped with one log line.
- Falloff needs real surface normals in the MDX; non-uniform bone scale is handled.
- While a `uv0Offset` or `uv0Scale` track applies, the layer's MDX texture animation is off.
- `vertexColor` holds 4 x `vertexCount` values in final MDX vertex order, per surface (one geoset and layer).
- `positionCrc32` is the CRC-32 of the MDX vertex positions as stored (x, y, z floats). Without it only `vertexCount` is checked.
- A wrong length, bad value or CRC mismatch skips `vertexColor` with one log line; the rest of the surface applies.
- A key exactly at a sequence's end is never shown in normal playback, for any model. End at least 33 ms after the last key.
- `color0` also overrides texture flipbooks, replaceable textures (team colour) and texture swaps on that layer.
- Paths must end in `.blp`, `.tga` or `.dds`. Among map files with the same path without extension, `.blp` wins over `.tga`, then `.dds`.
- A missing or broken `color0` file leaves the MDX texture; a broken `.blp` may draw empty instead. Extra loader log lines can appear.
- A `color0` entry without `uvSet` reads UV set 0, whatever the layer's MDX `coordId` says.
- HD materials get no sidecar fields, `color1` included.

### Two textures (`color1`)

A is the `color0` (or MDX) sample, B the `color1` sample, r1 is `weight0`, r2 is `weight1`.

| `combine` | Colour | Alpha |
|---|---|---|
| `over` | `A * (1 - B.a*r1) + B * (B.a*r1)` | `A.a` |
| `mix` | `A*r1 + B*(1 - r1)` | Same formula on alpha. |
| `multiply` | `(1 + (A-1)*r1) * (1 + (B-1)*r2)`, clamped to 0..1 | Same formula on alpha. |
| `multiplyAddAlpha` | `B * (1 + (A-1)*r1)`, clamped to 0..1 | `A.a*r1 + B.a*r2`, clamped to 0..1 |

- The result replaces the texture sample before vertex colour, falloff, `alphaRef` and opacity.
- For the second UV set write `uvSet` 1 (default 0); the MDX geoset must have two UV sets.
- An invalid `uvSet` on `color1` drops that entry with one log line.
- A missing or undecodable `color1` draws `color0` alone; a broken `.blp` may draw empty instead.
- Without a loaded `color1`, `combine` and the weights do nothing.
- `color1` ignores MDX texture animation; only `uv1Offset` and `uv1Scale` move it.
- Wrap defaults to the layer's MDX texture wrap flags; set `wrapU` and `wrapV` on a scrolling `color1`.
- Use straight (not premultiplied) alpha, and a unique base name.
- On additive layers using `combine` or `alphaRef`, also override `color0` with a straight-alpha texture.
- Without OpenGL 3 sampler objects, or if the shaders fail to compile: one log line, no `color1`.

### Ribbons (`ribbons`)

Each entry in the root `ribbons` list changes one ribbon emitter. Since 2026-11-25.

| Key | Effect |
|---|---|
| `ribbon` | Required: index in the MDX ribbon emitter list. |
| `uvScroll` | `[u, v]` in texture units per second; positive `u` moves the texture toward the head. |
| `ageColor` | Stops `[age, r, g, b, a]` multiplying the ribbon's MDX colour and alpha per segment. |

- `uvScroll` needs rows = columns = 1 and the layer texture's wrap flag on each scrolled axis.
- The scroll runs on each model instance's animation clock: it continues on a held last frame and stops at time scale 0.
- `age` runs from 0 (newest) to 1 (oldest); list stops in ascending order. Values blend linearly.
- Outside the stops the first or last value holds; equal ages make a step, and the later stop wins.
- Colour and alpha components are clamped to 0..1.
- A bad or duplicate `ribbon` index drops the entry; bad, empty or unsorted stops drop only `ageColor`.
- An unusable `uvScroll` or a dropped entry or field logs one line.
- Ribbons ignore `surfaces`. A ribbon without an entry draws as before.

```json
{ "version": 1, "surfaces": [ {
  "geoset": 0, "layer": 0, "vertexCount": 4, "depthWrite": false,
  "textures": [ { "role": "color0", "path": "war3mapImported\\ring_sa.tga" } ],
  "tracks": { "opacity": { "keys": [[0, 1], [1000, 0]] } }
} ] }
```

## Planned

No further native is designed yet.

Direction only, no design yet: a Lua-facing custom UI API (beyond the Frame API that follows WC3) and WarMash's own extended native API on top of `common.j` and the `Blz*` natives.

## Runtime extensions without the prefix

These exist today and predate the prefix rule. They are described in the rows of the [differences page](wc3-differences.html):

- `require`, `package.preload` and `package.loaded` in Lua maps (Lua runtime).
- `GroupEnumUnitsInRangeOfUnit` and its `Counted` form (Regions and unit groups).
- `SetAbilityDisabled`, `SetAbilityIconShowing` and the extra disable type `ConvertAbilityDisableType(7)` (Abilities and spells).
- `SetUnitAnimationToWalk` (Units and heroes).
