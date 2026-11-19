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
| `SetUnitX`, `SetUnitY` | bugfix | A single script step moving a unit over 120 units draws it at the new point at once; 120 or less glides. |
| `SetUnitPosition`, `SetUnitPositionLoc`, Blink, Mass Teleport, Way Gate | works | Draw the unit at the new point at once. |
| Short teleports and swaps (under 120 units) | partial | Glide for 0.05 s instead of appearing instantly. |
| Several script moves of one unit in one tick | partial | Moves from separate timers or early event code can add up past 120 and snap. |
| Dash moving more than 120 units per timer fire | partial | Snaps every fire instead of gliding. |
| `SetUnitFacing`, `SetUnitFacingTimed` | bugfix | Turn gradually at the turn rate, at most about 382 degrees per second (cited formula). |
| `BlzSetUnitFacingEx` | deliberate | Instant; cancels a pending turn. |
| `SetUnitFlyHeight` | bugfix | The rate is honoured; 0 or NaN is instant. |
| `SetUnitFlyHeight` below 0 | unverified | Ground unit types are clamped at 0. |
| Snared units (`UNIT_TYPE_SNARED`) | bugfix | Do not walk and read move speed 0, as measured. |
| `GetUnitMoveSpeed`, `BlzGetUnitRealField(UNIT_RF_SPEED)` with a speed buff | bugfix | Unrounded value, as measured; movement itself still uses whole numbers. |
| `SetUnitMoveSpeed` | bugfix | Clamped to 150..400 and read back unrounded, as measured; a map's own minimum or maximum, including 0, is used as the bound (0 for both gives speed 0). |
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
