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
| `WmsIsUnitFacingUnit` | `(unit, target, coneDegrees) -> boolean` | True when the target is inside the unit's facing cone (full angle). | 2026-11-23 | experimental |
| `WmsIsUnitFacingUnitInRange` | `(unit, target, coneDegrees, range) -> boolean` | Facing cone plus centre-to-centre range (no collision size, unlike `IsUnitInRange`). | 2026-11-23 | experimental |
| `WmsIsUnitFacingPoint` | `(unit, x, y, coneDegrees) -> boolean` | Same cone test against a point. | 2026-11-23 | experimental |
| `WmsGetUnitFacingDelta` | `(unit, x, y) -> real` | Signed degrees in (-180, 180] from the unit's facing to the point; positive is left (counter-clockwise). | 2026-11-23 | experimental |
| `WmsAddPlayerGold` / `WmsAddPlayerLumber` | `(player, integer) -> integer` | Adds (negative spends), clamps the result to 0..2147483647, returns the change applied; HUD updates, no floating text. | 2026-11-23 | experimental |
| `WmsCanPlayerAfford` | `(player, gold, lumber) -> boolean` | True if the player holds both amounts; spends nothing; a cost of 0 or less is always covered. | 2026-11-23 | experimental |

Details for facing:
- Cone: true when `|delta| <= cone/2` (inclusive); `cone >= 360` is always true, a negative cone false. Angles: 0 = east, 90 = north.
- A point on the unit's own position, and `unit == target`, counts as facing (delta 0), so a group-enum cone filter includes the caster; exclude it in the script. A negative range is never met.
- Null or dead unit: false, delta 0.

Details for resources:
- Goes through the same player gold and lumber setter as `SetPlayerState`. Null player: 0 / false.
- WarMash has no resource cap and `SetPlayerState` is unclamped; only these natives clamp.

```lua
if WmsCanPlayerAfford(Player(0), 100, 0) then WmsAddPlayerGold(Player(0), -100) end
print(WmsIsUnitFacingPoint(u, 0, 0, 90))
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

## Planned

No further native is designed yet.

Direction only, no design yet: a Lua-facing custom UI API (beyond the Frame API that follows WC3) and WarMash's own extended native API on top of `common.j` and the `Blz*` natives.

## Runtime extensions without the prefix

These exist today and predate the prefix rule. They are described in the rows of the [differences page](wc3-differences.html):

- `require`, `package.preload` and `package.loaded` in Lua maps (Lua runtime).
- `GroupEnumUnitsInRangeOfUnit` and its `Counted` form (Regions and unit groups).
- `SetAbilityDisabled`, `SetAbilityIconShowing` and the extra disable type `ConvertAbilityDisableType(7)` (Abilities and spells).
- `SetUnitAnimationToWalk` (Units and heroes).
