---
title: Home
nav_order: 1
permalink: /
---

# WarMash script API

WarMash is a Java engine that runs Warcraft III maps, a fork of the Warsmash Mod Engine. It runs custom maps, Lua maps for Reforged included, through the same script natives as the real game. It is in development: some natives are missing or behave differently, and these pages record where.

Written for map authors who script in Lua; JASS maps run too.

## Pages

| Page | What it covers |
|---|---|
| [Differences from Warcraft III](wc3-differences.html) | Every native that behaves differently, by topic. Start here. |
| [WarMash script API (Wms)](warmash-extensions.html) | API that exists only in WarMash. |
| [Launch arguments](launch-arguments.html) | Start a map directly, loading and memory options. |
| [QA and debug hooks](qa-hooks.html) | Test tooling, not for maps. |
| [Changelog](changelog.html) | User-visible changes. |

## Status words

| Word | Meaning for your map |
|---|---|
| works | Implemented; listed because it was missing before or has a detail worth knowing. |
| partial | Some fields or arguments work; the rest return 0, false or nil. |
| gap | Not implemented, or behaves differently in a way that is not intended. |
| stub | Exists so scripts do not fail, but does nothing. |
| stored-only | Keeps the value you set and reads it back, with no effect in game. |
| deliberate | Differs from Warcraft III on purpose; a map relying on WC3 behavior needs a change. |
| bugfix | A defect that was fixed, or a Warcraft III bug that WarMash does not copy. |
| unverified | WarMash chose a behavior that nobody has checked in the real game yet. |

"Measured" means checked by a scripted run in real Reforged 3.0.1. "Not confirmed" means the real game was not checked.

## When a native is missing

The call returns nil and writes one line to the error log; the script keeps running:

```
Call to native function that was declared but had no native implementation: <name>
```

If a feature of your map does nothing, look for this line first.

## Reference versions

Lua maps: Reforged 3.0.1, with the bundled 3.0.1 `common.j` and `blizzard.j`. JASS maps: patch 1.29, scripts read from the configured game install.
