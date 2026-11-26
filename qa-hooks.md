---
title: QA and debug hooks
nav_order: 5
---

# QA and debug hooks
{: .no_toc }

**Test tooling, not an API for maps.** These hooks let automated runs drive the game and read its state. They can change without notice and do not exist in real Warcraft III. Outside a test run every `WarMashQa*` name is nil.

- `-qalog <file>` writes one JSON record per line: map load, state every 5 s, exit.
- `-qalua <file>` runs a Lua file in the map's VM after load; all `WarMashQa*` functions exist only there.
- Functions marked "JSON" return a JSON string.

```
./gradlew desktop:runGame -Pargs="-loadfile <path to map>.w3x -window -qalog <output folder>/run.jsonl -qaexit 25 -qalua <output folder>/check.lua"
```

```lua
-- check.lua: put a value into the -qalog output
WarMashQaReport("local.player", tostring(GetPlayerId(GetLocalPlayer())))
```

Most launch arguments below need `-qalog` and `-loadfile`. Full field lists are in the engine source (`QaStateLogger`).

## Run control

| Hook | What it does |
|---|---|
| `-qalog <file>` | JSON log of load, state and exit records; also counts script errors and missing natives. |
| `-qaexit <seconds>` | Exits that long after the map has loaded. |
| `-qaleave <seconds>` | Returns to the menu after that long and loads the map once more. |
| `-qaleavecount <n>`, `-qaleavemap <map>` | Repeats the leave `n` times; restarts load `<map>`. |
| `-qaquestdialog <gameSeconds>` | Opens the quest dialog once at that game time, as F9 does. |
| `-qalua <file>[;<file>...]` | Runs Lua files in order in the map's VM; a failing file stops the rest. |
| `-qablpsync` | Turns off worker-thread BLP decoding at map load (decode on the render thread). |
| `-qasoundsync` | Decodes script sounds on the render thread instead of the background (for timing comparisons). |
| `-qablpticksync` | Keeps texture decoding during a script tick on the render thread (for timing comparisons). |
| `-qareadsync` | Keeps archive reads on the render thread (for timing comparisons). |
| `-qaunitsoundsync` | Loads unit response sounds on the render thread (for timing comparisons). |
| `-qapendingpixmap` | Uses the old pending UI texture upload (zeroed Pixmap, then a second upload) for comparison. |
| `-qagrab` | Keeps cursor catch and edge scrolling on (`-qalog` alone turns them off). |
| `-qacursorpin <x>,<y>` | Pins the cursor position the in-game UI reads (window pixels, top-left origin); the OS cursor does not move. Needs `-qalog` and `-loadfile`. |
| `WarMashQaReport(key, value)` | Writes a value into the log; at most 3000 keys per run. |
| `WarMashQaExitNow()` | Ends the run like `-qaexit`. |
| `open_spot.lua` (helper script) | `WarMashQaOpenSpot` and `WarMashQaClearance` find open walkable ground for test units. |
| `walkable_gen.mjs <outDir> <baseMap>` (Node script) | Builds the walkable bridge test map; `<outDir>` must be below the temp folder and new, empty or generator-marked. |
| `walkable_scene.lua` (scene script) | Measures walkable heights, clicks, crossing, destroy and revive on that map; ends the run itself. |
| `walkable_deck_scene.lua` (scene script) | Checks the bridge deck natives, walking over and under, and build and item limits on that map. |
| `walkable_doodad_deck_scene.lua` (scene script) | Checks the doodad bridge deck natives, rails, and build and item limits on that map. |
| `walkable_deck_draw_scene.lua` (scene script) | Checks drawn heights of units, items, effects and projectiles around a bridge deck on that map. |
| `walkable_deck_touch_scene.lua` (scene script) | Checks orders, melee, follow, pickup and unit clicks across the two levels on that map. |

## Log fields worth knowing

| Field | What it holds |
|---|---|
| `exit.missingNatives`, `missingNativeCalls`, `stubNativeCalls` | Missing and stub natives with call counts. |
| `exit.offThreadReads`, `blpAsync.deferredReads`, `soundAsync.deferredReads` | Archive reads done off the render thread, and reads deferred to workers. |
| `exit.luaCaughtErrors`, `exit.luaCaughtErrorMessages` | Errors caught by `xpcall` handlers (up to 20 messages). |
| `state.cursor.edgeScroll` | Edge-scroll counters: frames on the edge, scrolled or not (with reason), focus changes. |
| `state.camera`, `localSelection` | Camera fields and the local player's selection. |
| `state.camera.projectionFarZ`, `state.worldPixelRows` | Actual far plane; dark share of 7 world rows (black-cut check). |
| `state.worstFrame`, `state.frameGapMaxMs` | The slowest frame and its top 10 natives. |
| `state.worstFrame.unitCreate`, `unitResponses` | Unit creation cost per part; unit response sound counters (plays, late or silent plays, dropped files). |
| `state.worstFrame.frameTexture`, `gcCount`, `gcMs`, `gcByCollector` | `BlzFrameSetTexture` cost per part in the slowest frame; JVM collections since map load. |
| `hud` | World frame, hero buttons, tooltip, buff bar, multiboard, chat, FPS text. |
| `state.healthBars` | Floating bars and text tags. |
| `damageEvents` (exit) | Counts and last values of damage events. |
| `mapLoaded.phases`, `loadCost` | Load time by phase and cost bucket. |
| `mapLoaded.loadDetail`, `exit.loadDetail` | BLP decode steps, sound read and decode (5 slowest), paths of missing textures (`imageInputNull`). |
| `effectPreload` timing fields | `wallMs`, `blpSummedMs`, `blpThreads`, `blpWallShare`, `workWallShare`. |
| `exit.blpAsync` | Worker decode counters: `enabled`, `threadsPerBatch`, `batches`, `submitted`, `completed`, `workerFailures`, `workerTimeouts`, `inlineRuns`, `maxInFlight`, `waits`, `waitMs`, `lateFailures`, `threads`. |
| `loadCost.blpDecode` vs `loadDetail.blpMs` | With workers on, `blpDecode` is render-thread time only; `blpMs` sums all threads. `blpAsync.submitted` also counts UI and command-card icons. |
| `assetLoadFailures` | Distinct missing asset paths, each counted once (not every load attempt). |
| `state.mapTextures`, `groundTextures`, `exit.textureFailures` | Textures served from the map, ground tiles, failed paths. |
| `iconPreload` | Icon loading progress and frame cost. |

## Frames and HUD

| Hook | What it does |
|---|---|
| `-qaclick <frame[#ctx]> <s>` | Clicks a script frame once. |
| `-qahover`, `-qawheel`, `-qamouseup <frame[#ctx]> <s>` | Fires mouse enter, wheel or mouse-up on a script frame. |
| `-qahideorigin <s>` | Calls `BlzHideOriginFrames` once. |
| `-qadump <prefix>`, `-qadumpat <turn>` | Dumps script frames by name prefix, at a given game turn. |
| `-qatooltip <s> <slot>` | Shows a command card tooltip (slot 100+i: hero button i). |
| `-qahealthbars on` or `-qahealthbars off` | Overrides "Always show Health Bars" for one run. |
| `-qaherobuttonclick <s> <index>` | Clicks a hero button. |
| `WarMashQaFrameInfo`, `WarMashQaFrameJson`, `WarMashQaFrameDump`, `WarMashQaStockFrameDump` | Frame state, by frame or name (JSON). |
| `WarMashQaTextLayout(frame)` | Line and segment layout of a TEXT frame (JSON). |
| `WarMashQaHudInfo()` | World frame, hero name bar, menu buttons, resource texts, minimap (JSON). |
| `WarMashQaCommandCard()` | The 4x3 command card with hotkeys, icons and disabled state (JSON). |
| `WarMashQaShowTooltip(slot)`, `WarMashQaTooltipInfo()` | Shows a tooltip and reports its box (JSON). |
| `WarMashQaHeroButtons()`, `WarMashQaHeroButtonClick(i)`, `WarMashQaHeroHotkey(i)` | Hero button state, click and F-key press (JSON). |
| `WarMashQaMultiboard()`, `WarMashQaMultiboardMinimizeClick()` | Drawn multiboard state and its minimize button. |
| `WarMashQaHealthBars()`, `WarMashQaHoldAlt(held)`, `WarMashQaHoverUnit(u)` | Floating bar state, Alt and hover simulation. |
| `WarMashQaHotkeyMode(mode)` | Hotkey mode for the run: 0 letters, 1 grid, -1 config. |
| `WarMashQaPortraitInfo()`, `WarMashQaPortraitClick([right])` | Portrait state and a click on it. |
| `WarMashQaReadPixels`, `WarMashQaSetStockFrameVisible`, `WarMashQaStockOnScreen` | Pixel readback and stock frame checks. |
| `WarMashQaArmUnitTarget(orderId)` | Arms a unit-target command of the selected unit; also arms attack and move. |
| `WarMashQaIconPreload()` | Icon loading state (JSON). |

## Units, input and camera

| Hook | What it does |
|---|---|
| `-qaspawn <s> <rawcode>` | Creates a unit for the local player at its start location. |
| `-qaselect <s>`, `-qaselectcycle <s>` | Selects a hero, or cycles through unit types every 0.5 s. |
| `-qacamfield <s> <field> <value>` | Applies `SetCameraField` at game second `s`; repeatable. |
| `-qakey <s> <key>` | Presses a key (LibGDX name, case-sensitive). |
| `-qachat <s> <n>` | Posts `n` chat lines. |
| `-qafogprobe <s>` | Compares the visibility natives with the raw fog of war. |
| `-qaplace <s> <rawcode> <dx> <dy>`, `-qabuildmode <s> <rawcode> <hold>` | Building placement checks and build mode. |
| `WarMashQaUnitInfo(u[, detail])` | Unit state: screen position, guard, pause, buffs, speed and more (JSON). |
| `WarMashQaKeyPress(name)`, `WarMashQaChat(line)`, `WarMashQaChatType(text)` | Key presses and chat input. |
| `WarMashQaTargetClick`, `WarMashQaPointerPress`, `WarMashQaPointerRelease` | Clicks at a world point or unit. |
| `WarMashQaTargetClick(x, y, right, z)` | The optional `z` clicks at that height instead of the terrain (JSON). |
| `WarMashQaWalkable(op, ...)` | Read only (JSON): `"at", x, y`; `"dest"`, `"item"` or `"unit"` with a handle; `"projectiles"`. |
| `WarMashQaMouseMove(x, y)` | Fires a mouse move event at a ground point. |
| `WarMashQaRayPickAt(unit)`, `WarMashQaCursorAt(x, y)`, `WarMashQaCursorInfo()` | Picking and cursor tint. |
| `WarMashQaCanPlace`, `WarMashQaBuildMode`, `WarMashQaBuildModeInfo` | Placement checks (JSON). |
| `WarMashQaShopInfo(unit[, player])` | What a shop sells and its stock (JSON). |
| `WarMashQaQueueOrder(unit, orderId[, target])` | Queues an order (Shift) on a unit or widget target, or with no target; true when an ability took it (stop needs none). |

## Effects, sound and performance

Runs with `-qamem`, `-qadsverify` or `-qablpverify` are slow: never use them for timing.

| Hook | What it does |
|---|---|
| `WarMashQaEffectInfo(effect)` | Script and render state of an effect (JSON). |
| `WarMashQaEffectScan(text)`, `WarMashQaEffectWatch(text, frames)` | Logs live effects whose model path contains the text. |
| `WarMashQaAttachInfo(unitOrEffect)` | Attach state of a unit or effect, simulation and drawn positions (JSON). |
| `WarMashQaRenderEffects()` | Counts of live and stuck entries in the effect list (JSON). |
| `WarMashQaDnc()` | State of the day and night light models (JSON). |
| `WarMashQaModelInfo(path)`, `WarMashQaEffectCandidates()` | Model contents and preload candidates (JSON). |
| `WarMashQaWmfxInfo(pathSuffix)` | Load counts of a `.wmfx.json` sidecar (JSON); `{}` if none was read. |
| `WarMashQaResult(key)` | A stored QA result, such as a pixel-read value, as text; nil if not set. |
| `-qaperf`, `WarMashQaPerfBegin`, `WarMashQaPerfEnd`, `WarMashQaPerfFrames`, `WarMashQaPerfSnapshot` | Frame profiler and measuring windows. |
| `-qanovsync` | Turns vsync and the 60 fps cap off. |
| `-qanoloadcost` | Turns load-cost tracking off. |
| `-qamem`, `WarMashQaMemory()`, `WarMashQaGlSites()` | Memory and live GL object reports. |
| `-qablpverify` | Checks every BLP decode against the old decoder. |
| `-qadsverify` | Checks every folder file lookup against the disk. |
| `-qauitexturekeys` | Lists UI texture cache keys in the exit record. |
| `WarMashQaBatchCompare(tag[, points[, files[, mode]]])` | Compares a frame with emitter merging on and off; `mode` `"cache"` or `"ribbonscroll"` switches that instead. |
| `-Dwarmash.qa.portraitInPass=true` | JVM switch for portrait drawing checks. |
