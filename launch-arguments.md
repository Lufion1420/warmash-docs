---
title: Launch arguments
nav_order: 4
---

# Launch arguments

Arguments for the desktop launcher that players and map authors can use. None of them exists in real Warcraft III. Arguments that start with `-qa` are test tooling and are listed under [QA and debug hooks](qa-hooks.html).

When WarMash is started from its source tree, arguments are passed through Gradle:

```
./gradlew desktop:runGame -Pargs="-loadfile <path to map>.w3x -window"
```

Paths inside `-Pargs` are absolute Windows paths and must not contain spaces, because the Gradle task splits the string at whitespace.

## Starting a map

- `-loadfile <map>`: start the given map directly (absolute Windows path accepted).
- `-window`: start windowed.
- `-nograb`: do not confine or catch the cursor (implied by `-qalog`).

## Loading options

Icon loading and effect preloading change when assets are loaded, not what a script observes.

- `-deferredicons` (opt-in): ability, unit and item icons load in the background after the map is shown, under a per-frame budget; icons of units on the map load in the first frame, and a new unit type's icons load in its creation frame. Default (no flag) is eager: all icons load at map load, as before, with no visible difference for map authors. Known limits of the opt-in mode: one-off stalls of about 9 to 14 ms while the warm-up runs (seen in 3 of 8 runs), up to about 60 ms in a frame that creates many new unit types. (2026-11-12)
- `-eagericons`: forces the eager default; wins when `-deferredicons` is also given (one stderr warning). (2026-11-12)
- `-iconbudget <ms>`: per-frame budget of the background loader, default 2; values below 0 count as 0 (one icon per frame); an unparsable value is ignored with a stderr line. Only with `-deferredicons`, otherwise a stderr warning. (2026-11-12)
- Launch argument `-nopreloadfx` (WarMash only, deliberate): skips the effect model preload at the end of map load (see "Special effects and sound" in [wc3-differences.md](wc3-differences.html)); the `effectPreload` field in the `mapLoaded` and exit records reports what was loaded. (2026-11-13)

## Memory

- Game heap (2026-11-16): `runGame` and `debug` start the game with a 3g heap by default; override with `-PgameHeap=<size>` or the environment variable `WARMASH_HEAP` (an `-Xmx` in `JAVA_TOOL_OPTIONS` still wins). `installDist` start scripts default to `-Xmx3g` and honour `WARMASH_HEAP` and an `-Xmx` in `JAVA_TOOL_OPTIONS` the same way (values with `&`, `|`, `>` or quotes are not supported).

## Diagnostic switches

JVM properties for comparing renderer behavior. They are not needed for normal play; details are on the [QA and debug hooks](qa-hooks.html) page.

- `-Dwarmash.noEmitterBatch=true`: turns the merging of additive particle emitters off.
- `-Dwarmash.noEmitterGlCache=true`: turns the emitter GL-call cache off.
- `-qahealthbars on|off`: overrides the "Always show Health Bars" option for one run without changing the saved setting.

## Related behavior: folder data sources

- Behavior for map authors and users: folder data sources (`[DataSources]` folder entries in `warsmash.ini`, folder maps) answer lookups from an in-memory file index built when the folder is opened and rebuilt whenever a map starts or a map list is built. A file added to or deleted from such a folder while a map runs is not seen until the next map start. (2026-11-12)
