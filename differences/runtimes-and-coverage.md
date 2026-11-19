---
title: Runtimes, map loading and native coverage
nav_order: 10
parent: Differences from Warcraft III
---

# Runtimes, map loading and native coverage
{: .no_toc }

## Lua runtime

Lua 5.3 with 64-bit integers. A declared but missing native returns nil and logs one line.

| Native | Status | What differs |
|---|---|---|
| Lua integers | gap | 64-bit, not 32-bit. |
| `require`, `package.preload`, `package.loaded` | deliberate | Available (WC3 has none). |
| `TypeDefine` | unverified | Does nothing. |
| Strings passed to natives | gap | Invalid UTF-8 becomes U+FFFD. |
| Null string results | unverified | Appear as nil. |
| `ExecuteFunc` | unverified | Queued as a new thread; an unknown name is logged once. |
| `TriggerSleepAction` outside a trigger action | unverified | The callback ends with one logged traceback. |
| Handles as weak-table keys | unverified | The key can vanish while the handle lives. |
| `GetLocalPlayer()` stored at load time | unverified | Should work in WarMash; in Reforged call it at use time. |
| `CreateTimer`, `CreateGroup` in `config()` | works | Implemented. |

## JASS runtime

JASS maps read `common.j` and `Blizzard.j` from the configured install (1.29 today).

| Native | Status | What differs |
|---|---|---|
| `GetTriggerUnit`, `GetFilterUnit`, `GetEnumUnit` from `main` | gap | Throw an error. |
| Integer literal with a leading zero (`010`) | bugfix | Read as octal (8), as in pjass; `08` and `09` stop the script with an error. |
| Real literal with a leading zero (`05.5`, `00.5`, `08.5`) | bugfix | Read as a decimal real (5.5, 0.5, 8.5), as in pjass. |
| Frame API natives in JASS maps | gap | Only if the install's `common.j` declares them. |
| One-character rawcode (`'a'`) | unverified | Read as 0x61000000, not 97. |

## Map loading and data

A map with `war3map.lua` is a Lua map. Data a non-melee map does not override comes from the 1.29 files.

| Native | Status | What differs |
|---|---|---|
| Lua or JASS | unverified | Lua when the archive contains `war3map.lua`. |
| `JASS_FILE_LIST` extras for Lua maps | gap | Ignored. |
| Lua maps' `common.j`, `blizzard.j` | deliberate | Bundled 3.0.1 copies, whatever the install. |
| Stock balance data for non-melee maps | deliberate | 1.29 values (Tavern sells 5 heroes, Ensnare 12 s), not Reforged's. |
| A map's `Units\CommandFunc.txt` | unverified | Replaces the stock file; missing sections use the built-in ones. |
| Model texture with replaceable id 21 or an unmapped id | deliberate | Gets an empty texture without a file lookup or a failure count. |
| Model texture path `war3mapImported\X.blp` | unverified | Does not fall back to `X.blp` at the archive root; fix the import path. |
| Missing BLP texture file | works | Logs one line and the texture stays empty. |
| Map object data changes at index above 0 | bugfix | Fields the base row lacks (button positions, per-level tooltips, attack-2 missile fields) are kept instead of dropped. |
| Map textures vs game textures | deliberate | A map file of the same base name wins, also across `.blp`, `.tga` and `.dds`. |

## Native coverage and constants

Missing natives are not listed one by one. Stored-only and stub natives are in their topics above.

| Native | Status | What differs |
|---|---|---|
| Unimplemented natives | gap | 594 declared natives missing in the Lua game VM (count of 2026-11-16); see Details for the list. |
| A declared native without implementation | gap | Returns nil and logs one line; no error is raised. |
| `MathRound` | unverified | Half away from zero. |
| `PLAYER_COLOR_BLACK`, `MAP_RELOADED` | gap | nil. |
| `Convert*` anim, subanim, fog style, key and mouse values | gap | Values exist, but no native uses the new ones yet. |

Details:
- Get the list: launch a Lua map and read the sorted `[lua] natives declared without implementation` list (the second one) in the newest `Logs/*.err.log` in the game's assets folder.
- Earlier counts: a stock JASS melee map 304 (2026-11-16); 865 of 1712 before later work.
