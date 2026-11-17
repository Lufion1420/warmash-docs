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

Details for `WmsSetUnitUntargetable`:
- Refused orders show the error "unable to target this unit"; a right-click by another unit becomes follow. Dummy casters and autocast are blocked too.
- Not affected: utility orders (load, repair, Moon Well, harvest, mounting, Mass Teleport, Entangle Gold Mine), area effects, script damage and missiles already in flight.
- Works from Lua and JASS; a map function with the same name shadows it.

```lua
local u = CreateUnit(Player(0), FourCC("hfoo"), 0, 0, 0)
WmsSetUnitUntargetable(u, true)
print(WmsIsUnitUntargetable(u)) --> true
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
