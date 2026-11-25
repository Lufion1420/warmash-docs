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
| `WMS_EVENT_UNIT_BUFF_RECEIVED` / `_BUFF_LOST` / `_ABILITY_ADDED` / `_ABILITY_REMOVED` | unit event | Fires for the unit itself when it gains or loses a buff or ability; register with `TriggerRegisterUnitEvent`. | 2026-11-24 | experimental |
| `WMS_EVENT_PLAYER_UNIT_BUFF_RECEIVED` / `_BUFF_LOST` / `_ABILITY_ADDED` / `_ABILITY_REMOVED` | player unit event | Same four events for any unit of the player; register with `TriggerRegisterPlayerUnitEvent`. | 2026-11-24 | experimental |
| `WmsGetEventAbilityId` | `() -> integer` | Event response in those events: the rawcode of the buff or ability; 0 outside one. | 2026-11-24 | experimental |
| `WmsGetEventAbility` | `() -> ability` | Event response: the ability or buff handle, also valid in the REMOVED and LOST events. | 2026-11-24 | experimental |
| `WMS_EVENT_UNIT_ABILITY_INTERVAL` | unit event | Fires for the owner of a passive ability every N seconds; N is the per-level real field `Wmsi` (0 = off); register with `TriggerRegisterUnitEvent`. | 2026-11-24 | experimental |
| `WMS_EVENT_PLAYER_UNIT_ABILITY_INTERVAL` | player unit event | Same event for any unit of the player; register with `TriggerRegisterPlayerUnitEvent`. | 2026-11-24 | experimental |
| `Wmsi` (ability real level field) | field | Interval in seconds, passive abilities only; set with `BlzSetAbilityRealLevelField(a, ConvertAbilityRealLevelField('Wmsi'), level0Based, seconds)`. | 2026-11-24 | experimental |
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

Details for ability interval events:
- Passive abilities only (built-in passives, auras, ability-builder passives). Handlers use `GetTriggerUnit`, `GetTriggerPlayer`, `WmsGetEventAbilityId`, `WmsGetEventAbility`.
- The first event comes one interval after the ability becomes active. Read the field back with `BlzGetAbilityRealLevelField`. Not a JASS constant: use `ConvertAbilityRealLevelField('Wmsi')`.
- No events while the ability is disabled, or the unit is dead, under construction, upgrading or an entangled gold mine; the timer restarts after those gaps. A paused unit or a script-hidden ability keeps firing.
- At most one event per tick, no catch-up. The interval rounds to the nearest tick (50 ms), minimum 1 tick; 0 turns it off.
- A level change keeps the phase and re-reads the interval; a morph keeps the timer of kept abilities. Setting only `Wmsi` does not re-run the ability's level-change code.
- No ability-JSON option for the interval yet.

```lua
BlzSetAbilityRealLevelField(abil, ConvertAbilityRealLevelField('Wmsi'), 0, 2.0)
```

Details for the quest log:
- The flag is stored for every player; only the local player's HUD changes. `FlashQuestDialogButton` and `ForceQuestDialogUpdate` do not open the dialog while hidden. Null player: no-op.
- Un-hiding shows the button again in its `QuestSetEnabled` state. If the map hid the button with `BlzFrameSetVisible`, un-hiding shows it anyway.
- `BlzFrameSetVisible(button, true)` while hidden shows the button, but it stays unclickable.

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
| Texture names | Give each sidecar texture a base name no other map image has, such as suffix `_sa`. |
| Errors | The model still loads; the bad file, surface or field is skipped and logged with `[wmfx]`. |
| World Editor, HiveWE | Whether they keep the imported file is not confirmed. |

Fields that work today, per entry in `surfaces`:

| Field | Effect (a missing field keeps the MDX value) |
|---|---|
| `hideOtherLayers` | Draws no other layer of this geoset's material, on this geoset only. |
| `twoSided`, `depthTest`, `depthWrite`, `unshaded`, `unfogged` | Booleans that override the layer flags. |
| `textures` with `role` `color0` | Replaces the layer texture; takes `path`, `uvSet` (default 0), `wrapU`, `wrapV`. |
| `tracks.opacity` | Replaces the layer alpha; 0 to 1. |
| `tracks.color` | RGB multiplier, 0 or more; values above 1 brighten. |
| `tracks.uv0Offset` | Moves the texture: `uv * scale + offset`. |
| `tracks.uv0Scale` | Scales from UV (0, 0); U and V must be equal, non-zero and one sign in every key. |

Parsed but ignored for now: `falloff`, `alphaRef`, `vertexColor`, `color1`, `falloff` and `normal` textures, `combine`, `weight0`, `weight1`, `softDistance`, `ribbons`, their tracks, and `glare`, `distortion`. A surface with `pass` `distortion` is skipped entirely.

Details:
- Tracks are `{"interp": "linear" or "step", "keys": [[ms, value...], ...]}`; `ms` is MDX time, ascending. Default `linear`.
- Only keys inside the playing sequence count; the first and last of them are held. Without any, the MDX value applies.
- While a `uv0Offset` or `uv0Scale` track applies, the layer's MDX texture animation is off. An invalid `uv0Scale` track is ignored.
- `color0` also overrides texture flipbooks, replaceable textures (team colour) and texture swaps on that layer.
- Paths must end in `.blp`, `.tga` or `.dds`. Among map files of one base name, `.blp` wins over `.tga`, then `.dds`.
- A missing or broken `color0` file leaves the MDX texture; a broken `.blp` may draw empty instead. Extra loader log lines can appear.

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
