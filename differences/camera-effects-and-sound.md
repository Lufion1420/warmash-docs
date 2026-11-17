---
title: Camera, effects and sound
nav_order: 9
parent: Differences from Warcraft III
---

# Camera, effects and sound
{: .no_toc }

## Camera

| Native | Status | What differs |
|---|---|---|
| Camera setup field slots 7 to 13 | gap | Accepted and ignored. |
| `SetCameraField`, `AdjustCameraField` | unverified | Interpolated over the duration; the mouse wheel drops the script values. |
| `CAMERA_FIELD_ROLL` and the local pitch, yaw and roll fields | stored-only | The view does not change. |
| `GetCameraField` | unverified | Radians for angles. |
| `GetCameraEyePositionX/Y/Z/Loc`, `GetCameraTargetPositionX/Y/Z/Loc` | unverified | Implemented. |
| `SetCameraPosition` | bugfix | Cancels a running pan. |
| `CAMERA_FIELD_ZOFFSET` | deliberate | Replaces the preset height until the camera is reset. |
| `CAMERA_FIELD_FARZ` | deliberate | The far plane is raised when the view needs more, so zooming out no longer cuts the ground to black; larger values apply, `GetCameraField` returns the set value. |
| `CameraSetSourceNoise`, `CameraSetTargetNoise` and their Ex forms | bugfix | A sinusoidal sway (shape inferred). |
| Camera bounds | unverified | The target is clamped every frame, as in a Reforged run. |
| `BlzCameraSetupApplyForceDurationSmooth`, `BlzCameraSetupGet/SetCameraType`, `BlzCameraSetupGet/SetLabel`, `BlzGet/SetCameraAllowsHotkeyTargetLock`, `CameraSetDepthOfFieldScale`, `CameraSetFocalDistance`, `CameraSetSmoothingFactor`, `Get/SetCameraFieldControlledByInput`, `SetCameraOrientController`, `SetCameraQuickPosition`, `SetCameraRotateMode`, `SetCinematicCamera`, `StopCamera` | gap | Not implemented (17 natives). |

## Special effects and sound

Effects attached to a unit ignore position, orientation and alpha. Images are not drawn.

### Special effects

| Native | Status | What differs |
|---|---|---|
| `BlzSetSpecialEffect*` position, scale, orientation, colour, alpha, time and `BlzPlaySpecialEffect` | works | Angles in radians, colour 0 to 255; a missing animation falls back to Stand. |
| Position, orientation or alpha natives on an attached effect | deliberate | Ignored without a crash (Reforged reportedly crashes). |
| Effects on bones of a faded-out geoset | bugfix | Drawn at their current spot, not frozen. |
| `BlzSpecialEffectAddSubAnimation`, `RemoveSubAnimation`, `ClearSubAnimations` | partial | `ROOTED`, `ENTANGLE` and `BERSERK` are ignored. |
| `AddSpecialEffectTarget` attachment names | unverified | Case-insensitive token match; an unknown name attaches to origin. |
| `DestroyEffect` | bugfix | Death plays once, at once; particles fade out. |
| Effect whose model has no Stand | unverified | Loops until `DestroyEffect`. |
| `DestroyEffect` at time scale 0 | deliberate | Death plays at speed 1 so the effect is removed. |
| `BlzSetSpecialEffectAlpha`, `BlzSetSpecialEffectColor` on particles | unverified | Do not reach particles and ribbons. |
| `BlzSetSpecialEffectPositionLoc`, `BlzResetSpecialEffectMatrix` | gap | Not implemented. |
| Drawn position of a moved effect | deliberate | Smoothed between ticks; getters return the script value. |
| Global sequences of a held effect | bugfix | Keep running, as seen in Reforged. |
| One-shot engine art off-screen | bugfix | Finishes and leaves. |
| `BlzPlaySpecialEffect` with a type other than STAND or WALK | deliberate | Plays once, then Stand. |
| `BlzSetSpecialEffectTimeScale(e, 0)` with `BlzSetSpecialEffectTime` | bugfix | The set frame is held. |
| `AddSpecialEffect` with an empty model path | bugfix | Returns an invisible handle instead of nil. |
| `CreateImage` family | stored-only | State is kept; nothing is drawn. |

### Models and textures

| Native | Status | What differs |
|---|---|---|
| MDX texture wrap flags | bugfix | Applied per model; one model no longer changes another. |
| MDX texture animation id out of range | unverified | Ignored instead of failing the model. |
| `Preload` with a model path | bugfix | Loads the model for `AddSpecialEffect`. |
| Effect model preload at map load | deliberate | Model paths in the script are loaded during the loading screen; `-nopreloadfx` turns it off. |
| Ribbon with no visibility key in the animation | unverified | Does not emit. |
| Splats and ribbons under distance fog | bugfix | Each uses its own fog setting; no flicker. |
| Models spawned by model events (gibs, blood) | unverified | Leave within 5 s of their end. |
| TGA types 3, 10 and 11, other origins | bugfix | Load. |
| 32-bit TGA with 0 alpha bits | unverified | The 4th byte is alpha if any pixel uses it. |
| TGA above 8192x8192 | unverified | Rejected. |
| JPEG BLP with an implausible size | deliberate | Decoded the old way; no visible difference. |

### Sound

| Native | Status | What differs |
|---|---|---|
| `StopSound` | gap | Stops at once; `fadeOut` is ignored. |
| `KillSoundWhenDone`, `SetSoundVolume`, `ClearMapMusic` | partial | Work on `CreateSound` handles only, not label or MIDI sounds. |
| `StartSound` on a playing looping sound | unverified | Does nothing; a refused start is retried by the next call. |
| Decoded sound data | deliberate | Shared per file; only memory and start time differ. |
| `SetSoundPosition` | partial | Used at the next start; `SetSoundDistances` and `AttachSoundToUnit` are missing. |
| Sounds never killed | bugfix | Released when the map closes. |
| 3D sound distance | unverified | Measured from the camera target. |
