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
| `SetCameraField`, `AdjustCameraField` | unverified | Interpolated over the duration; Insert and Delete drop the script values. |
| `CAMERA_FIELD_ROLL` and the local pitch, yaw and roll fields | stored-only | The view does not change. |
| `GetCameraField` | unverified | Radians for angles. |
| `GetCameraEyePositionX/Y/Z/Loc`, `GetCameraTargetPositionX/Y/Z/Loc` | unverified | Implemented. |
| Camera target height over terrain, `GetCameraTargetPositionZ` | bugfix | Follows smoothed terrain including cliffs, ignoring water and deformations; the exact smoothing is unverified. |
| `SetCameraPosition` | bugfix | Cancels a running pan. |
| `CAMERA_FIELD_ZOFFSET` | deliberate | Replaces the preset height until the camera is reset. |
| Mouse wheel zoom | deliberate | Changes only the distance (WC3 steps presets and changes the angle); zoom-out limit is the larger of the largest preset and 5000, and a script distance widens it. |
| `GetCameraField(CAMERA_FIELD_TARGET_DISTANCE)` | deliberate | Returns the wheel-zoomed distance; a script distance replaces the wheel zoom, and a camera reset clears it. |
| Wheel and script camera fields | deliberate | The wheel keeps script angle, rotation, field of view and far plane; Insert and Delete presets still use their own far plane. |
| `CAMERA_FIELD_FARZ` | deliberate | The far plane is raised when the view needs more, so zooming out no longer cuts the ground to black; larger values apply, `GetCameraField` returns the set value. |
| `CameraSetupApplyForceDurationWithZ` | works | Height changes linearly over the duration; a camera move, scroll or plain z set cancels it and keeps the height reached. |
| `PanCameraToWithZ`, `PanCameraToTimedWithZ` | bugfix | Z is an absolute target height floored at the ground, advancing only while xy moves; the floor is the smoothed camera ground. Was additive. |
| `PanCameraToWithZ` height after the pan | unverified | Persists until `ResetToGameCamera`, `CAMERA_FIELD_ZOFFSET` or a `CameraSetupApply*WithZ`; WC3 not confirmed. |
| `ResetToGameCamera(d)` | bugfix | Z offset and target height return to default linearly over d; a timed pan stops; the additive z offset is cleared too. |
| `PanCameraTo`, timed `CameraSetupApply` pan | bugfix | Pan at 3000 units/s (measured for `PanCameraTo` only; the `CameraSetupApply` speed is unverified). |
| `SetCameraTargetController` with inherit orientation | bugfix | The camera sits behind the unit and turns with its facing (the angle was converted wrongly). |
| Camera height around `WmsCameraFollowUnit` | deliberate | Exact ground while following, then eases over 0.5 s; instant camera jumps snap. |
| `CameraSetSourceNoise`, `CameraSetTargetNoise` and their Ex forms | bugfix | A sinusoidal sway (shape inferred). |
| Camera bounds | unverified | The target is clamped every frame, as in a Reforged run. |
| `BlzCameraSetupApplyForceDurationSmooth`, `BlzCameraSetupGet/SetCameraType`, `BlzCameraSetupGet/SetLabel`, `BlzGet/SetCameraAllowsHotkeyTargetLock`, `CameraSetDepthOfFieldScale`, `CameraSetFocalDistance`, `CameraSetSmoothingFactor`, `Get/SetCameraFieldControlledByInput`, `SetCameraQuickPosition`, `SetCameraRotateMode`, `SetCinematicCamera`, `StopCamera` | gap | Not implemented (16 natives). |

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
| `AddSpecialEffectTarget` with a missing model | unverified | Returns a null handle (nil in Lua). |
| Attached effects when a unit changes model (skin, morph, Polymorph) | unverified | They move to the matching point of the new model, else origin; Reforged may lose them. |
| `DestroyEffect` | bugfix | Death plays once, at once; particles fade out. Effects of removed units are freed next frame. |
| Effect whose model has no Stand | unverified | Holds its last frame until `DestroyEffect`. |
| Effect with only Birth and Death | bugfix | Appears once and holds its last frame; before, Birth restarted in a loop. Reforged: pulses until destroyed. |
| `DestroyEffect` at time scale 0 | deliberate | Death plays at speed 1 so the effect is removed. |
| `BlzSetSpecialEffectAlpha`, `BlzSetSpecialEffectColor` on particles | unverified | Do not reach particles and ribbons. |
| `BlzSetSpecialEffectPositionLoc`, `BlzResetSpecialEffectMatrix` | gap | Not implemented. |
| Drawn position of a moved effect | deliberate | Smoothed between ticks; getters return the script value. |
| Global sequences of a held effect | bugfix | Keep running, as seen in Reforged. |
| One-shot engine art off-screen | bugfix | Finishes and leaves. |
| `BlzPlaySpecialEffect` with a type other than STAND or WALK | deliberate | Plays once, then Stand. |
| `BlzSetSpecialEffectTimeScale(e, 0)` with `BlzSetSpecialEffectTime` | bugfix | The set frame is held. |
| `AddSpecialEffect`, `AddSpecialEffectLoc` with an empty model path | bugfix | Returns an invisible handle instead of nil, placed on ground or walkable surface. |
| `CreateImage` family | stored-only | State is kept; nothing is drawn. |
| `TerrainDeformCrater`, `TerrainDeformRipple`, `TerrainDeformWave`, `TerrainDeformRandom`, `TerrainDeformStop`, `TerrainDeformStopAll` | stub | Do nothing; the four creators return a null handle, so the terrain never deforms. |

### Models and textures

| Native | Status | What differs |
|---|---|---|
| MDX texture wrap flags | bugfix | Applied per model; one model no longer changes another. |
| MDX texture animation id out of range | unverified | Ignored instead of failing the model. |
| MDX texture animation scale | unverified | U scales with the key's x, V with its y. |
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
| `CreateSound` decoding | deliberate | The handle is valid at once and the file decodes in the background; a `StartSound` before that finishes plays a frame or so late. |
| `TriggerWaitForSound`, `GetSoundDuration` on a decoding sound | deliberate | The wait still counts from `StartSound`; the duration call decodes the sound right away. |
| `CreateSoundFromLabel` | deliberate | The file loads at the sound's first play. |
| `SetSoundPosition` | partial | Used at the next start, not applied to a sound already playing. |
| `SetSoundDistances` | partial | The minimum is the reference distance at the next start; the maximum is stored but not applied; 2D sounds ignore it. |
| `SetSoundDistanceCutoff` | works | Applied at the next start; with stop-when-out-of-range on, a sound beyond the cutoff is not started. |
| `AttachSoundToUnit` | partial | Uses the unit's position at attach and at each start, but does not follow it while playing; 2D sounds ignore it. |
| `StartSoundEx` | deliberate | Same as `StartSound`; the fade-in is ignored. |
| `SetSoundChannel`, `SetSoundConeAngles`, `SetSoundConeOrientation`, `SetSoundVelocity` | stub | Do nothing. |
| `SetCameraOrientController` | stub | Does nothing; WC3 shows no visible effect either (measured). |
| 3D script sounds with a reference distance | deliberate | They now fade with distance, like unit sounds. |
| Sounds never killed | bugfix | Released when the map closes. |
| 3D sound distance | unverified | Measured from the camera target. |
