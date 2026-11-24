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
| Snared units (`UNIT_TYPE_SNARED`) | bugfix | Do not walk and read move speed 0, as measured. |
| `GetUnitMoveSpeed`, `BlzGetUnitRealField(UNIT_RF_SPEED)` with a speed buff | bugfix | Unrounded value, as measured; movement itself still uses whole numbers. |
| `SetUnitMoveSpeed`, move speed limits | deliberate | A map's `MaxUnitSpeed` or the unit's own maximum clamps only values set by script; Reforged also caps at 522, WarMash has no cap. |
| Move speed floor | deliberate | The final speed is at least 1 or the minimum from the map or unit, buffs included (Slow alone on 270 reads 150); speed-0 types stay 0. |
| Polymorph move speed | works | Uses the sheep type's minimum speed (100). |
| `GetUnitMoveSpeed`, `SetUnitMoveSpeed` on a polymorphed unit | unverified | The read shows the sheep speed; a set during the morph and speed gains made meanwhile show after it ends. |
| `SetUnitMoveSpeed`, `BlzSetUnitRealField(UNIT_RF_SPEED)` during Polymorph | bugfix | The value is clamped to the unit type's own minimum and maximum speed, not the sheep's. |
| `SetUnitMoveSpeed` on a slowed unit | bugfix | The slow stays applied to the written value (320 with a 54 reduction reads 266) and is not added again when it ends; WC3 value not measured. |
| `SetUnitTurnSpeed`, `BlzSetUnitRealField(UNIT_RF_TURN_RATE)` during Polymorph | bugfix | The unit turns at the sheep's rate; a value written during the morph survives its end; Reforged not measured. |
| Changing a unit's type from script | bugfix | Active slows stay on the new type's speed, and a type that can move never loses its move ability. |
| Slow (Thunder Clap) on a Polymorph sheep | bugfix | Takes its percent of the sheep's speed; WC3 not measured. |
| `BlzSetUnitRealField(UNIT_RF_SPEED)` | deliberate | Acts like `SetUnitMoveSpeed` (same limits) and returns true; not confirmed in Reforged. |
| `UnitRemoveAbility(u, 'Amov')` | bugfix | Returns true: the unit stops walking and drops move orders but still attacks in range; attack-move to a point is refused. |
| `UnitAddAbility(u, 'Amov')` | deliberate | Re-adds Amov after a script removal; false otherwise. |
| `GetUnitDefaultMoveSpeed` | unverified | The unit type's speed from object data; 0 for a null unit. |
| Slow or speed upgrade while a speed buff is active | bugfix | Change the base speed without baking in the buff; WC3's combination rule is unverified. |
| `IsTerrainPathable` | partial | Works for walk, fly, build and float pathing; other types return false. |
| `SetTerrainPathable` | unverified | `true` clears the cell's blocking bit, or blight with `PATHING_TYPE_BLIGHTPATHING`. |
| `SetTerrainPathable` with peon-harvest, amphibious or any pathing | gap | Ignored with one log line. |
| `GetTerrainType` | unverified | The tile of the nearest terrain corner. |
| `SetTerrainType` | stub | The ground does not change. |

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
