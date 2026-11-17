---
title: Launch arguments
nav_order: 4
---

# Launch arguments

Desktop launcher arguments for players and map authors. None exists in real Warcraft III. Arguments starting with `-qa` are test tooling: see [QA and debug hooks](qa-hooks.html).

```
./gradlew desktop:runGame -Pargs="-loadfile <path to map>.w3x -window"
```

Inside `-Pargs`, paths must be absolute and contain no spaces (the string is split at whitespace).

## Starting a map

| Argument | What it does |
|---|---|
| `-loadfile <map>` | Starts the map directly. |
| `-window` | Starts windowed. |
| `-nograb` | Does not catch the cursor (implied by `-qalog`). |

## Loading

These change when assets load, not what a script observes.

| Argument | What it does |
|---|---|
| `-deferredicons` | Loads icons in the background after the map shows (opt-in; can cause short stalls). |
| `-eagericons` | Loads all icons at map load (the default); wins over `-deferredicons`. |
| `-iconbudget <ms>` | Per-frame budget of the background icon loader, default 2; needs `-deferredicons`. |
| `-nopreloadfx` | Skips the effect model preload at the end of map load. |

## Memory

| Setting | What it does |
|---|---|
| `-PgameHeap=<size>` or `WARMASH_HEAP` | Game heap for `runGame`, `debug` and the start scripts; default 3g. |
| `-Xmx` in `JAVA_TOOL_OPTIONS` | Wins over both. |

## Diagnostic switches

Not needed for normal play.

| Switch | What it does |
|---|---|
| `-Dwarmash.noEmitterBatch=true` | Turns the merging of additive particle emitters off. |
| `-Dwarmash.noEmitterGlCache=true` | Turns the emitter GL-call cache off. |
| `-qahealthbars on` or `-qahealthbars off` | Overrides "Always show Health Bars" for one run; the saved setting stays. |

## Good to know

Files added to or removed from a folder data source (folder entries in `warsmash.ini`, folder maps) while a map runs are seen only at the next map start.
