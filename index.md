---
title: Home
nav_order: 1
permalink: /
---

# WarMash script API documentation

WarMash is a game engine that runs Warcraft III maps, a fork of the Warsmash Mod Engine written in Java. It reads the data files of a Warcraft III installation and runs custom maps, including Lua maps written for Warcraft III: Reforged, through the same script natives the real game offers (`common.j` and the `Blz*` natives). It is still in development: not every native is implemented, and some behave differently from the real game, which is what these pages record.

The pages are written for map authors who script in Lua. JASS maps run too; where a row applies only to one language it says so.

## Pages

- [Differences from Warcraft III](wc3-differences.html): the complete status reference, native by native, grouped by topic. Look here first when something in your map behaves differently.
- [WarMash script API (Wms)](warmash-extensions.html): script API that exists only in WarMash, and the `Wms` prefix rule.
- [Launch arguments](launch-arguments.html): start a map directly, loading and memory options.
- [QA and debug hooks](qa-hooks.html): test tooling (`-qa...` arguments and `WarMashQa*` functions). Not for use in maps.
- [Changelog](changelog.html): user-visible changes by date.

## How to read the status words

Every row of the differences page compares one native or behavior in real Warcraft III with WarMash and labels it.

| Word | What it means for your map |
|---|---|
| works, real, same | Implemented. The row exists because of a detail worth knowing, or because it was a gap before. |
| gap | Not implemented, or only partly. The row says what your script observes instead. |
| stub, no-op | The native exists so that scripts do not fail, but calling it does nothing. |
| stored-only | The native keeps the value you set, and the matching getter returns it, but the game does not act on it. |
| partial | Some arguments or fields work; the row lists them. The rest returns 0, false or nil, usually with one line in the log. |
| deliberate | WarMash differs from Warcraft III on purpose. The row says why. A map that relies on the Warcraft III behavior needs a change. |
| bugfix | A defect that is fixed, either an earlier WarMash bug or a Warcraft III bug that WarMash does not copy. The row says what changed. |
| unverified | WarMash had to choose a behavior and nobody has confirmed what the real game does. It may change once it is measured. |
| measured, cited, inferred | How well the "Real WC3" column is backed: by a scripted run in the real game, by documentation or community sources, or by reasoning from a neighbouring case. |
| Not confirmed | In the "Real WC3" column: the real game's behavior has not been checked. It does not mean the real game does nothing. |
| resolved | The gap or bug is fixed. Resolved rows stay for one release so that you can see what changed, then they are removed. |

A native that does not appear on the differences page is either believed to match Warcraft III or is missing altogether. Missing natives are not listed one by one; see "Native coverage and constants" on that page for the current count and how to get the list for your own map.

## What happens when a map calls a missing native

The call returns nil (null in JASS) and one line is written to the error log:

```
Call to native function that was declared but had no native implementation: <name>
```

No error is raised, so the script keeps running with the nil value. If a feature of your map does nothing in WarMash, check the error log for this line first.

## Reference versions

Lua maps are compared with Warcraft III: Reforged 3.0.1 and always use the bundled 3.0.1 `common.j` and `blizzard.j`. JASS maps are compared with patch 1.29 and read both scripts from the configured game installation.
