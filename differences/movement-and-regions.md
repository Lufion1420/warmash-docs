---
title: Movement, terrain and regions
nav_order: 4
parent: Differences from Warcraft III
---

# Movement, terrain and regions
{: .no_toc }

## Movement, pathing and terrain

```lua
-- Speed reads are not rounded (base 270 with Bloodlust +25%)
print(GetUnitMoveSpeed(u))                     --> 337.5
print(BlzGetUnitRealField(u, UNIT_RF_SPEED))   --> 337.5
```

| Native | Status | What differs |
|---|---|---|
| `SetUnitX`, `SetUnitY` | bugfix | A single script step (one timer, trigger or event run) moving a unit over 120 units draws it at the new point at once; 120 or less glides. |
| `SetUnitPosition`, `SetUnitPositionLoc`, Blink, Mass Teleport, Way Gate | works | Draw the unit at the new point at once. |
| Short teleports and swaps (under 120 units) | partial | Glide for 0.05 s instead of appearing instantly. |
| Script waiting between `SetUnitX` and `SetUnitY` | partial | The wait splits one jump into two steps, so a diagonal teleport can glide. |
| Dash moving more than 120 units per timer fire | partial | Snaps every fire instead of gliding. |
| `SetUnitFacing`, `SetUnitFacingTimed` | bugfix | Turn gradually at the turn rate, at most about 382 degrees per second (cited formula). |
| `BlzSetUnitFacingEx` | deliberate | Instant; cancels a pending turn. |
| `SetUnitFlyHeight` | bugfix | The rate is honoured; 0 or NaN is instant. |
| `SetUnitFlyHeight` below 0 | unverified | Ground unit types are clamped at 0. |
| `GetUnitFlyHeight`, `BlzGetUnitRealField(UNIT_RF_FLY_HEIGHT)` on a netted flyer | bugfix | Read the height from before the net, as measured; the drawn unit still sinks. |
| `SetUnitFlyHeight` on a netted unit | unverified | Changes the read and the height restored after the net; the drawn unit stays down. |
| Fly height after `UnitAddType(u, UNIT_TYPE_SNARED)` | unverified | Reads stay at the pre-snare height, which removal restores instead of the type default. |
| Snared units (`UNIT_TYPE_SNARED`) | bugfix | Do not walk and read move speed 0, as measured. |
| `GetUnitMoveSpeed`, `BlzGetUnitRealField(UNIT_RF_SPEED)` with a speed buff | bugfix | Unrounded value, as measured; movement itself still uses whole numbers. |
| `SetUnitMoveSpeed`, move speed limits | deliberate | A map's `MaxUnitSpeed` or the unit's own maximum clamps only values set by script; Reforged also caps at 522, WarMash has no cap. |
| Move speed floor | deliberate | The final speed is at least 1 or the minimum from the map or unit, buffs included (Slow alone on 270 reads 150); speed-0 types stay 0. |
| Polymorph move speed | works | Uses the sheep type's minimum speed (100). |
| `GetUnitMoveSpeed`, `SetUnitMoveSpeed` on a polymorphed unit | unverified | The read shows the sheep speed; a set during the morph and speed gains made meanwhile show after it ends. |
| `SetUnitMoveSpeed`, `BlzSetUnitRealField(UNIT_RF_SPEED)` during Polymorph | bugfix | The value is clamped to the unit type's own minimum and maximum speed, not the sheep's. |
| `SetUnitMoveSpeed` on a slowed unit | bugfix | A Thunder Clap slow is a percent of the written value (320 with 20% reads 256) and ends cleanly; WC3 value not measured. |
| `SetUnitTurnSpeed`, `BlzSetUnitRealField(UNIT_RF_TURN_RATE)` during Polymorph | bugfix | The unit turns at the sheep's rate; a value written during the morph survives its end; Reforged not measured. |
| Changing a unit's type from script | bugfix | Active slows apply their percent to the new type's speed, and a type that can move never loses its move ability. |
| Collision size growing mid-walk (type change) | bugfix | The walk re-fits its path instead of getting stuck. |
| Thunder Clap slow and Polymorph | bugfix | A slow cast before or during the morph slows the sheep and still applies to the unit afterwards (270 with 20% reads 216); WC3 not measured. |
| Overlapping Thunder Clap slows | unverified | Only the strongest applies to move speed instead of compounding; flat speed bonuses are not slowed. |
| `BlzSetUnitRealField(UNIT_RF_SPEED)` | deliberate | Acts like `SetUnitMoveSpeed` (same limits) and returns true; not confirmed in Reforged. |
| `UnitRemoveAbility(u, 'Amov')` | bugfix | Returns true: the unit stops walking and drops move orders but still attacks in range; attack-move to a point is refused. |
| `UnitAddAbility(u, 'Amov')` | deliberate | Re-adds Amov after a script removal; false otherwise. |
| `GetUnitDefaultMoveSpeed` | unverified | The unit type's speed from object data; 0 for a null unit. |
| Thunder Clap slow or speed upgrade while a speed buff is active | bugfix | The buff is never baked into the base speed; WC3's combination rule is unverified. |
| `IsTerrainPathable` | partial | Works for walk, fly, build and float pathing; other types return false. |
| `IsTerrainPathable` on a cell opened by a bridge or doodad | deliberate | Opted-in openings and lifted doodad deck rails read pathable; rail blocking is not counted. |
| `SetTerrainPathable` and blight on an opened cell | deliberate | The write is stored and takes effect after the cell closes. |
| `SetTerrainPathable` | unverified | `true` clears the cell's blocking bit, or blight with `PATHING_TYPE_BLIGHTPATHING`. |
| `SetTerrainPathable` with peon-harvest, amphibious or any pathing | gap | Ignored with one log line. |
| `GetTerrainType` | unverified | The tile of the nearest terrain corner. |
| `SetTerrainType` | stub | The ground does not change. |
| `SetUnitPathing` | gap | Not implemented: the call does nothing. |

### Walkable surfaces

Walkable destructables and opted-in doodads raise units and `GetLocationZ`; some terrain reads still ignore them.

| Native | Status | What differs |
|---|---|---|
| `GetLocationZ` | bugfix | Returns ground or walkable surface height, whichever is higher; a nil location returns 0. |
| `GetLocationZ` over water | unverified | Water height is not counted; documented Warcraft III includes it. |
| `CreateDestructable` without a height | bugfix | Takes the walkable surface height, even if the bridge was never drawn. |
| `GetTerrainCliffLevel` | gap | Terrain level only; documented Warcraft III adds a walkable destructable's cliff height. |
| `SetTextTagPos` | gap | Height starts at the terrain; documented Warcraft III also includes walkable surfaces. |
| `BlzGetUnitZ`, `BlzGetLocalUnitZ` | gap | Not implemented; documented Warcraft III (1.30+) returns surface height plus the unit's occluder height. |
| Walkable destructable pathing | unverified | By default only adds blocking: water and gaps need pathing that is open in the map. |
| `WmsSetDestructableTypeOpensPathing` (WarMash only) | deliberate | Opted-in bridge types open water, cliff and gap cells at runtime; Warcraft III cannot. |
| `WmsSetDoodadTypeWalkable` (WarMash only) | deliberate | Opted-in doodad types raise units to model height; Warcraft III has no such native. |
| `WmsSetDoodadTypeOpensPathing` (WarMash only) | deliberate | Opted-in walkable doodads open water, cliff and gap cells; blocking texture pixels still block. |
| Walkable destructable scale and terrain tilt | bugfix | Walk height covers the whole scaled model and follows its tilt. |
| Walkable destructable the camera has not drawn | bugfix | Gives walk height from creation. |
| Walkable model with several walk surfaces | bugfix | The highest selectable, visible part sets the height; Reforged not measured. |
| Script-created destructables | bugfix | No longer drawn black. |
| Preplaced destroyed destructable | bugfix | Revive and `RemoveDestructable` no longer leave its destroyed-state blocking behind. |
| Destroyed walkable destructable height | unverified | Parts visible at the end of Death set script heights; bone movement is ignored. |
| Destroyed stock bridges | unverified | `GetLocationZ` still reports the intact deck height. |
| Walkable height and model files | unverified | Script reads use each player's model files, so SD and HD data would differ. |
| Destructable pathing at angles between 90-degree steps | deliberate | Snaps to the nearest 90 degrees, centred and unscaled; Warcraft III rule not measured. |

Details:

- `WmsSetDestructableTypeOpensPathing`, `WmsSetDoodadTypeWalkable`, `WmsSetDoodadTypeOpensPathing`: rules and limits are on the [WarMash API page](../warmash-extensions.html#natives).

### Two-layer bridges

Only bridges declared with `WmsAddBridgeDeck` or `WmsAddDoodadBridgeDeck` behave this way; Warcraft III has one pathing layer.

| Native | Status | What differs |
|---|---|---|
| `WmsAddBridgeDeck`, `WmsRemoveBridgeDeck`, `WmsIsBridgeDeck` (WarMash only) | deliberate | Declared bridges let units walk over and under, changing layer at two ramps. |
| `WmsAddDoodadBridgeDeck`, `WmsRemoveDoodadBridgeDeck`, `WmsIsDoodadBridgeDeck` (WarMash only) | deliberate | The same decks on placed walkable doodads, off while the type is not walkable. |
| `WmsGetUnitPathingLayer`, `WmsSetUnitPathingLayer` (WarMash only) | deliberate | Read or set whether a unit is on a deck or below it. |
| `CreateUnit`, `ReviveHero`, summons, teleports onto a deck | deliberate | New units and moves over 256 land on top; shorter moves follow the walking rule. |
| Build orders near an active deck | deliberate | Refused on deck, ramp and rail cells; `CreateUnit` can still place buildings there. |
| `CreateItem`, `BlzCreateItemWithSkin`, `SetItemPosition`, item drops | deliberate | Items on deck or ramp cells go to the nearest spot off them. |
| Destructable bridge rails with an active deck | deliberate | Stop blocking walking underneath; still block building and, on vision-blocking bridges, sight. |
| Doodad bridge rails with an active deck | deliberate | Stop blocking walking only over dry, level, ramp-free ground; rail cells stay unbuildable. |
| Deck drawing and click targeting | gap | Not done yet; layer-0 units on or under a deck are still drawn at model height. |

Details:

- Ramp sizes, return values and limits are on the [WarMash API page](../warmash-extensions.html#natives).

## Regions and unit groups

```lua
-- Spatial enumerations return corpses, as in WC3: filter them
GroupEnumUnitsInRange(g, x, y, 600.0, Filter(function()
    return not IsUnitType(GetFilterUnit(), UNIT_TYPE_DEAD)
end))
```

| Native | Status | What differs |
|---|---|---|
| `RegionAddRect`, `RegionAddCell`, `RegionAddCellAtLoc` over a standing unit | bugfix | The unit is inside at once but gets no enter event. |
| `RegionClearRect`, `RegionClearCell`, `RegionClearCellAtLoc` | bugfix | `IsUnitInRegion` turns false at once and no leave event fires. |
| `GroupEnumUnitsOfType`, `...OfPlayer`, `...InRect`, `...InRange`, `...InRangeOfLoc`, `...Selected` and their Counted forms | bugfix | Clear the group first; the spatial forms skip Locust units. |
| `GroupEnumUnitsInRect`, `GroupEnumUnitsInRange`, `GroupEnumUnitsInRangeOfLoc` | bugfix | Return dead units until the corpse is gone, as in WC3. |
| `GroupEnumUnitsInRangeOfUnit` and its Counted form (WarMash only) | bugfix | Clear first, skip Locust units, accept a null unit. |
