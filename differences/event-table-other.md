---
title: "Event table: player, game, frame and other events"
nav_order: 15
parent: Differences from Warcraft III
---

# Event table: player, game, frame and other events
{: .no_toc }

Everything that is not a unit event, then every way to register an event. Quick answers: [which getter to use](event-use.html). Unit events: [unit event table](event-table-units.html).

**Fires** is `yes`, `partial` or `never`. In these tables `GetTriggerUnit` and `GetTriggerWidget` are nil unless a row says otherwise.

## Player events

`GetTriggerPlayer` is the player concerned, and `GetTriggerEventId` works.

| Event | Fires | Event getters with a value |
|---|---|---|
| `EVENT_PLAYER_STATE_LIMIT` | yes | `GetEventPlayerState` |
| `EVENT_PLAYER_ALLIANCE_CHANGED` | yes | None; no getter tells which alliance or other player |
| `EVENT_PLAYER_CHAT` | yes | `GetEventPlayerChatString`, `GetEventPlayerChatStringMatched` |
| `EVENT_PLAYER_MOUSE_DOWN`, `EVENT_PLAYER_MOUSE_UP` | yes | Mouse position getters, `BlzGetTriggerPlayerMouseButton` |
| `EVENT_PLAYER_MOUSE_MOVE` | yes | Mouse position getters; the button is nil |
| `EVENT_PLAYER_KEY_DOWN`, `EVENT_PLAYER_KEY_UP` | yes | `BlzGetTriggerPlayerKey`, `BlzGetTriggerPlayerMetaKey`, `BlzGetTriggerPlayerIsKeyDown` |
| `EVENT_PLAYER_KEY` | partial | Never reported as the event id; a key handler reads KEY_DOWN or KEY_UP |
| `EVENT_PLAYER_END_CINEMATIC` | yes | Raised by the Escape key |
| `EVENT_PLAYER_LEAVE` | partial | Raised only by `RemovePlayer` on a player other than the local one |
| `EVENT_PLAYER_DEFEAT`, `EVENT_PLAYER_VICTORY` | never | |
| `EVENT_PLAYER_ARROW_LEFT_DOWN`, `EVENT_PLAYER_ARROW_LEFT_UP`, `EVENT_PLAYER_ARROW_RIGHT_DOWN`, `EVENT_PLAYER_ARROW_RIGHT_UP` | never | Use `BlzTriggerRegisterPlayerKeyEvent` |
| `EVENT_PLAYER_ARROW_DOWN_DOWN`, `EVENT_PLAYER_ARROW_DOWN_UP`, `EVENT_PLAYER_ARROW_UP_DOWN`, `EVENT_PLAYER_ARROW_UP_UP` | never | Use `BlzTriggerRegisterPlayerKeyEvent` |
| `EVENT_PLAYER_SYNC_DATA` | never | |

Details:

- Mouse position getters: `BlzGetTriggerPlayerMouseX`, `BlzGetTriggerPlayerMouseY`, `BlzGetTriggerPlayerMousePosition`.

## Game events

`GetTriggerPlayer` is nil, and `GetTriggerEventId` works unless noted.

| Event | Fires | Event getters with a value |
|---|---|---|
| `EVENT_GAME_ENTER_REGION` | yes | `GetTriggerUnit`, `GetEnteringUnit`, `GetTriggeringRegion`; no trigger widget |
| `EVENT_GAME_LEAVE_REGION` | yes | `GetTriggerUnit`, `GetLeavingUnit`, `GetTriggeringRegion`; no trigger widget |
| `EVENT_GAME_TIMER_EXPIRED` | yes | Timer interval: event id only. Timer object: `GetExpiredTimer`, event id nil |
| `EVENT_GAME_VARIABLE_LIMIT` | partial | None; JASS globals only |
| `EVENT_GAME_STATE_LIMIT` | partial | `GetEventGameState`; time of day only |
| `EVENT_GAME_VICTORY`, `EVENT_GAME_END_LEVEL` | never | |
| `EVENT_GAME_LOADED`, `EVENT_GAME_SAVE` | never | |
| `EVENT_GAME_TRACKABLE_HIT`, `EVENT_GAME_TRACKABLE_TRACK` | never | |
| `EVENT_GAME_SHOW_SKILL`, `EVENT_GAME_BUILD_SUBMENU`, `EVENT_GAME_CUSTOM_UI_FRAME` | never | |
| `EVENT_GAME_TOURNAMENT_FINISH_SOON`, `EVENT_GAME_TOURNAMENT_FINISH_NOW` | never | |

## Widget and dialog events

`GetTriggerUnit` and `GetTriggerPlayer` are nil.

| Event | Fires | Event getters with a value |
|---|---|---|
| `EVENT_WIDGET_DEATH` | yes | `GetTriggerWidget`; `GetTriggerDestructable` for a destructable; no dying unit or killer |
| `EVENT_DIALOG_BUTTON_CLICK` | yes | `GetClickedButton`, `GetClickedDialog` |
| `EVENT_DIALOG_CLICK` | yes | `GetClickedButton`, `GetClickedDialog` |

## Frame events

Register with `BlzTriggerRegisterFrameEvent`. A firing event sets `BlzGetTriggerFrame`, `BlzGetTriggerFrameEvent`, `BlzGetTriggerFrameValue`, `BlzGetTriggerFrameText` and `GetTriggerPlayer` (local player); `GetTriggerEventId` is nil.

| Event | Fires | Frames |
|---|---|---|
| `FRAMEEVENT_CONTROL_CLICK` | yes | Buttons |
| `FRAMEEVENT_MOUSE_ENTER`, `FRAMEEVENT_MOUSE_LEAVE` | yes | Button, text, edit box, slider |
| `FRAMEEVENT_MOUSE_UP`, `FRAMEEVENT_MOUSE_WHEEL` | yes | Button, text, edit box, slider |
| `FRAMEEVENT_CHECKBOX_CHECKED`, `FRAMEEVENT_CHECKBOX_UNCHECKED` | yes | Check box |
| `FRAMEEVENT_EDITBOX_TEXT_CHANGED`, `FRAMEEVENT_EDITBOX_ENTER` | yes | Edit box |
| `FRAMEEVENT_POPUPMENU_ITEM_CHANGED` | yes | Popup menu |
| `FRAMEEVENT_SLIDER_VALUE_CHANGED` | yes | Slider |
| `FRAMEEVENT_MOUSE_DOWN`, `FRAMEEVENT_MOUSE_DOUBLECLICK` | never | |
| `FRAMEEVENT_SPRITE_ANIM_UPDATE` | never | |
| `FRAMEEVENT_DIALOG_ACCEPT`, `FRAMEEVENT_DIALOG_CANCEL` | never | |

## Ways to register

All 26 register natives, plus the `TimerStart` callback.

| Register with | Fires | What to know |
|---|---|---|
| `TriggerRegisterUnitEvent` | partial | Per event: [unit table](event-table-units.html) |
| `TriggerRegisterPlayerUnitEvent` | partial | Per event: [unit table](event-table-units.html); the filter reads `GetFilterUnit` |
| `TriggerRegisterPlayerEvent` | partial | Only mouse, end cinematic and leave events can fire |
| `TriggerRegisterPlayerChatEvent` | yes | Chat |
| `BlzTriggerRegisterPlayerKeyEvent` | yes | Key down and up |
| `TriggerRegisterPlayerStateEvent` | yes | Player state limit |
| `TriggerRegisterPlayerAllianceChange` | yes | Alliance change |
| `TriggerRegisterUnitStateEvent` | yes | Unit state limit |
| `TriggerRegisterDeathEvent` | yes | Widget death |
| `TriggerRegisterEnterRegion` | yes | The filter reads `GetFilterUnit` |
| `TriggerRegisterLeaveRegion` | yes | The filter reads `GetFilterUnit` |
| `TriggerRegisterTimerEvent` | yes | `GetExpiredTimer` is nil |
| `TriggerRegisterTimerExpireEvent` | yes | `GetExpiredTimer` works |
| `TimerStart` callback | yes | `GetExpiredTimer` works; no other event data |
| `TriggerRegisterVariableEvent` | partial | JASS globals only; a Lua global raises an error |
| `TriggerRegisterGameStateEvent` | partial | Time of day only; other states raise an error |
| `TriggerRegisterDialogEvent` | yes | Any button of the dialog |
| `TriggerRegisterDialogButtonEvent` | yes | One button |
| `BlzTriggerRegisterFrameEvent` | partial | See frame events above |
| `TriggerRegisterGameEvent` | never | The registration is dropped and logged |
| `TriggerRegisterFilterUnitEvent` | never | Missing native |
| `TriggerRegisterUnitInRange` | never | Missing native |
| `TriggerRegisterCommandEvent` | never | Missing native |
| `TriggerRegisterUpgradeCommandEvent` | never | Missing native |
| `BlzTriggerRegisterPlayerSyncEvent` | never | Missing native; `BlzSendSyncData` is missing too |
| `TriggerRegisterTrackableHitEvent` | never | Raises an error |
| `TriggerRegisterTrackableTrackEvent` | never | Raises an error |

WarMash-only events are on the [script API page](../warmash-extensions.html).

## Completeness

Counted against the bundled 3.0.1 `common.j`. Each constant appears once in these two tables.

| Constant type | Constants | Table |
|---|---|---|
| `unitevent` | 50 | [Unit events](event-table-units.html) |
| `playerunitevent` | 49 | [Unit events](event-table-units.html) |
| `playerevent` | 22 | This page |
| `gameevent` | 16 | This page |
| `dialogevent` | 2 | This page |
| `widgetevent` | 1 | This page |
| `frameeventtype` | 16 | This page |
| Total | 156 | |
