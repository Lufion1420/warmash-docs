---
title: UI frames
nav_order: 7
parent: Differences from Warcraft III
---

# UI frames
{: .no_toc }

The Frame API works for script frames, and the stock parts that custom-UI maps move (minimap, portrait, hero buttons, buff bar, tooltips, world frame) are real. Other origin frames are empty placeholders.

### Frame layout and events

| Native | Status | What differs |
|---|---|---|
| `BlzFrameSetScale` | unverified | The frame's own scale, else the parent's; point offsets scale with the frame (not for TEXT). |
| `BlzFrameGetWidth`, `BlzFrameGetHeight` | unverified | Lay out first and return the scaled size. |
| `BlzFrameIsVisible` | bugfix | False when any ancestor is hidden, as measured. |
| `BlzFrameSetParent` | deliberate | Copies the parent's level; the frame's own alpha resets to 1 (same look). |
| `BlzFrameClearAllPoints` without a new point | unverified | The frame is not drawn or clickable until it gets a point; root frames are exempt. |
| Draw order of overlapping frames | unverified | Siblings by level, then creation; children above their parent. |
| `BlzFrameGetChild`, `BlzFrameGetChildrenCount` on the game UI root | gap | Engine frames `SmashFloatingBars` (0) and `SmashWorldFrame` (1) come first, so indices differ from WC3. |
| `BlzFrameGetChild` on buttons | unverified | Documented order; out of range returns nil. |
| `BlzFrameSetText` on a template-less GLUETEXTBUTTON | bugfix | Shows the text. |
| `BlzCreateFrameByType` EDITBOX or SLIDER without a template | bugfix | Works. |
| `BlzGetOriginFrame`, `BlzTriggerRegisterFrameEvent` with an out-of-range index | unverified | Return nil with one log line. |
| `BlzFrameClick` on a disabled button | bugfix | Fires no click event. |
| `BlzEnableUIAutoPosition`, `BlzFrameSetFocus`, `BlzFrameCageMouse`, `BlzFrameSetTextSizeLimit`, `BlzTextAreaFrameSetAutoScroll` | stored-only | Value stored, one log line, no effect. |
| Frame events: checkbox, edit box, slider, popup menu, mouse enter, leave, up and wheel | works | Fire; disabled frames fire none. |
| Frame events MOUSE_DOWN, MOUSE_DOUBLECLICK, SPRITE_ANIM_UPDATE | deliberate | Never fire, as observed in WC3. |
| Frame events DIALOG_ACCEPT, DIALOG_CANCEL | gap | Never fire: no dialog frame type. |
| Keyboard focus after a click | gap | WC3's focus quirk is not modelled. |

Details:
- Slider events fire only on a value change; popup item index is 0-based; wheel delta is +120 or -120 (sign unverified).

### Text and fonts

| Native | Status | What differs |
|---|---|---|
| Template-less TEXT frame: size | unverified | Sized from its text with one point, else 0.012 high; font height 0.0105. |
| `BlzFrameSetAbsPoint` on a scaled TEXT frame | bugfix | The offset is not scaled; only the font is. |
| Template-less TEXT frame: line pitch | deliberate | 0.0095 at scale 1, scaled with the frame. |
| TEXT frame placed by two points | deliberate | Wraps at the frame width, between words. |
| Colour codes (`&#124;c`, `&#124;r`) | bugfix | Coloured runs line up with the preceding text. |
| Text that overflows a fixed-height TEXT frame | bugfix | Whole lines outside the frame are not drawn. |
| Multi-line CENTER text | unverified | Each line is centred; right-aligned lines still align left. |
| `BlzFrameSetEnable` on TEXT | stub | No visual effect. |
| `BlzFrameSetTextAlignment` CENTER, MIDDLE | unverified | Text is centred in the frame. |
| `BlzFrameSetFont` | partial | Only the height is applied. |
| Script TEXT under a later-sibling BACKDROP | bugfix | Text draws above the backdrop, as seen in WC3. |
| Text at an effective scale below 0.05 | deliberate | Not drawn. |

### Textures and sprite frames

| Native | Status | What differs |
|---|---|---|
| `BlzFrameSetTexture` with a texture that fails to load | unverified | Keeps the old texture; the failure is logged once. |
| `BlzFrameSetTexture` with "" or `NONE` | unverified | Clears the texture. |
| `.png` texture paths | gap | Do not load. |
| Texture by bare file name | deliberate | Exact path only, plus an extension swap. |
| `[FrameDef]` in war3mapSkin.txt | gap | Not applied. |
| `BuildTimeIndicator` skin key | gap | Ignored. |
| Tiled BACKDROP | bugfix | Tiles stay inside the inner rect. |
| SPRITE frame given an image path | unverified | Draws nothing, one log line. |
| `BlzFrameSetSpriteAnimate` | partial | Plays the animation at once; queue flags are ignored. |
| `BlzFrameSetModel` with a camera index the model lacks | unverified | Drawn flat at the frame's bottom left, playing sequence 0. |
| Such screen-space models: draw layer | gap | Drawn above all UI, tooltips included; parent alpha ignored. |
| Such models: visibility and lifetime | unverified | Follow the frame's visibility; freed on `BlzDestroyFrame`. |
| `BlzFrameSetModel` on a hidden frame | unverified | The model loads when the frame is first shown. |
| `BlzFrameSetModel` with a blank path | unverified | Clears the model. |
