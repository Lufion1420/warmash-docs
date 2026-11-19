---
title: HUD and command card
nav_order: 8
parent: Differences from Warcraft III
---

# HUD and command card
{: .no_toc }

Interface parts that continue [UI frames](ui-frames.html).

### Stock HUD and origin frames

| Native | Status | What differs |
|---|---|---|
| `BlzHideOriginFrames` | unverified | Hides the stock HUD; `false` restores it; script children stay visible. |
| `BlzGetOriginFrame` for `CommandBarFrame`, `Leaderboard*` | gap | Empty placeholder frames. |
| `BlzGetFrameByName("ConsoleUIBackdrop", 0)` | bugfix | A real art-less root frame drawn below the stock frames; its real rect and type are unknown. |
| Minimap (`MiniMapFrame`, `ORIGIN_FRAME_MINIMAP`) | works | The real minimap; frame natives take effect. |
| Minimap draw order | unverified | Drawn below all root-level script frames. |
| Minimap when `ConsoleUI` moves | gap | Does not follow. |
| `BlzGetOriginFrame(ORIGIN_FRAME_MINIMAP_BUTTON, 0..4)`, `BlzGetFrameByName` for the minimap button bar and buttons | works | Real clickable frames anchored to `ConsoleUI`, so a map that moves `ConsoleUI` moves them; left click only; positions unverified. |
| Minimap terrain button | works | Hides and shows the minimap terrain image. |
| Minimap signal, ally color, creep and formation buttons | stored-only | Click state and textures change, but there is no ping, ally-color mode, creep markers or formation movement; formation starts on. |
| Creep-camp icon on stock maps | gap | Draws below its square. |
| `ORIGIN_FRAME_WORLD_FRAME` | bugfix | A real frame: once fetched, its rect sets the 3D view. |
| Frames parented to the world frame | works | Draw below the stock HUD. |
| `ORIGIN_FRAME_UNIT_PANEL_BUFF_BAR` and its label | bugfix | Real frames. |
| `ORIGIN_FRAME_HERO_BAR`, `HERO_BUTTON`, `HERO_HP_BAR`, `HERO_MANA_BAR`, `HERO_BUTTON_INDICATOR` | unverified | Real clickable hero icons; not hidden by `BlzHideOriginFrames`. |
| `ORIGIN_FRAME_PORTRAIT` | deliberate | Drawn exactly in the frame rect; Reforged's offset quirk is not copied. |
| Inventory parts when `ConsoleUI` moves | bugfix | Follow its position, not its visibility, alpha or scale. |
| `BlzGetFrameByName` for `InfoPanelIcon*` stat frames | bugfix | Contexts 0, 1, 2 and 6 work; other contexts return nil. |
| Frames that follow a moved `SimpleNameValue` | bugfix | Class value, level bar and progress indicator move with it. |
| Hero XP bar at max level | unverified | Shown full. |
| Minimap camera box | unverified | A trapezoid outline; Reforged shows a larger rectangle. |
| Top menu buttons on Lua maps | deliberate | Reforged labels; the buttons open nothing. |
| `ORIGIN_FRAME_UNIT_MSG` | bugfix | The real message frame: `DisplayTextToPlayer` lines follow it and hide with it; default rect is WarMash's old message position. |
| Status bar fill | bugfix | Clamped to 0..1. |
| FPS text | deliberate | Top right, smaller. |

Details:
- Portrait: frame alpha and scale are not applied; a size-0 frame shows no portrait.
- Hero buttons: a dead hero's icon is darkened; F1 to F3 select heroes; stock layout unverified.

### Tooltips

| Native | Status | What differs |
|---|---|---|
| `ORIGIN_FRAME_UBERTOOLTIP` placement | bugfix | The box follows the script-placed text and stays on screen. |
| `BlzGetOriginFrame(ORIGIN_FRAME_TOOLTIP)` | deliberate | The one-line hero-button tip, not the command tooltip box. |
| `BlzFrameIsVisible`, `BlzFrameSetVisible` on that tip | deliberate | Return the script's flag; the engine shows the tip without changing it. |
| Placement of that tip | deliberate | Above the hovered hero icon until a script sets a point. |

### Floating health bars

| Native | Status | What differs |
|---|---|---|
| HP bars on Frame API maps | works | Drawn under all UI and script frames. |
| "Always show Health Bars" option | deliberate | On by default; holding Alt hides the bars. |
| Bar look and height | unverified | Heroes get a box with level, HP and mana; height from the Stand animation. |

### Command card and cursor

| Native | Status | What differs |
|---|---|---|
| Command button hotkey letters | deliberate | Lua maps show each button's hotkey, as in Reforged; JASS maps show none. |
| Escape in a sub-card | deliberate | Closes one level (spell book, build menu, hero skills) and does not cancel targeting; cancelling targeting returns to the sub-card. |
| Passive item-based abilities | deliberate | Get no command button. |
| Passive ability with no button position | deliberate | Gets no command button; Reforged hides some passives even with one and draws item passives without one. |
| Disabled button without disabled art | bugfix | Normal icon at 45 percent brightness. |
| Cursor tint | deliberate | Red over enemies, green over own or allied units, yellow over neutral units and objects. |

### Chat and text messages

| Native | Status | What differs |
|---|---|---|
| `ORIGIN_FRAME_CHAT_MSG` | bugfix | The real chat area; lines fade after 10 s; local only. |
| `BlzDisplayChatMessage` | gap | Not implemented. |
| `TriggerRegisterPlayerChatEvent`, `GetEventPlayerChatString`, `GetEventPlayerChatStringMatched` | works | Fire from the local player's chat; chat is not sent over the network. |
| `DisplayTextToPlayer` duration | bugfix | `len*0.1 + 3` seconds, as in WC3. |

### Multiboards and quests

| Native | Status | What differs |
|---|---|---|
| `Multiboard*` (27 natives) | deliberate | Drawn at the top right, one board at a time; items are 0-based. |
| `MultiboardSetRowCount`, `MultiboardSetColumnCount` shrinking | unverified | Removed cells lose their values. |
| Multiboard item widths | deliberate | Applied on the next display, as documented. |
| Multiboard and `ShowInterface(false)` | unverified | Hidden; an unset title colour is gold. |
| `MultiboardReleaseItem` | deliberate | Does nothing; the cell stays. |
| Multiboard item with an invalid icon | gap | Draws nothing; WC3 draws green. |
| `MultiboardDisplay(board, false)` | gap | Leaves the shown board alone; WC3 closes it. |
| Multiboard drawing details | unverified | Long text is not clipped; the board is click-through. |
| Multiboard borders | bugfix | Sit on the boxes measured in Reforged. |
| `Quest*` (21 natives) | stored-only | Values are kept and read back; there is no quest dialog. |

### Cinematic filters and interface

| Native | Status | What differs |
|---|---|---|
| `SetCineFilter*`, `DisplayCineFilter`, `IsCineFilterDisplayed`, `CinematicFadeBJ` | unverified | A full-screen fade below all UI, interpolated in game time. |
| Cine filter blend modes | partial | BLEND and ADDITIVE exact; the others approximated. |
| `SetCineFilterTexMapFlags` | stored-only | Ignored. |
| `EnableUserUI` | stored-only | Input is not blocked. |
