# Full-screen camera + gallery toggle — design

**Date:** 2026-08-04
**File under change:** `index 11.html` (isolated copy of `index.html`; original untouched)

## Problem

Today the capture screen renders the camera/photo inside a small centered "card" sized
to the cutter aperture, on a green cutting-mat background. Two issues:

1. **Small, boxed preview** — the user wants the live camera to fill the whole screen with
   the stamp cutter floating on top (native-camera feel).
2. **Punch produces a wrong crop** — the exported stamp shows only a top strip of the photo
   with the rest black. Root cause: `cropFeed` maps the aperture to source pixels with a
   uniform `scale = mediaWidth / boxWidth` and no centering offset, which is wrong for a
   `object-fit: cover` element whose aspect ratio differs from its box. The sampled source
   rectangle runs past the media's bottom edge → black.

## Goals

- Camera fills the entire capture screen; metal cutter frame overlays centered; aperture is a
  transparent window onto the live feed (matches the reference screenshot).
- A **Camera | Gallery** toggle (two pills), default **Camera**.
- Gallery-picked photo behaves exactly like the camera: full-bleed behind the frame,
  pan/zoom to position, then punch. One shared punch/crop path.
- Fix the crop so the punched stamp matches what the aperture framed.
- No disruption to review/save, stored stamp shape, grid, collection detail, formats,
  reposition/adjust, quota/paywall.

## Design

### Layout
- The media layer (`<video>` for camera, `<img>` for a picked gallery photo) is positioned
  `inset: 0` and fills the capture root with `object-fit: cover`. The existing `feedView`
  pan/zoom transform still applies to it.
- The cutter SVG + `#capture-hit` aperture stay centered on top (unchanged markup/geometry).
- Green mat shows only when neither camera nor a picked photo is available.

### State / mode
- New `captureMode: 'camera' | 'gallery'` (default `'camera'`).
- New `pickedSrc` (data URL of the gallery photo; `null` until picked).
- Hidden `<input type="file" accept="image/*">` drives gallery picking (revives the
  `_fileInput` pattern from earlier versions).
- `cameraActive` is only pursued in camera mode.

### Toggle behavior
- **Camera pill:** `captureMode='camera'`, clear `pickedSrc`, `requestCamera()`.
- **Gallery pill:** `captureMode='gallery'`, `stopCamera()`, open the file picker.
  - File chosen → read as data URL → `pickedSrc = url` → full-bleed `<img>` shown.
  - Picker cancelled → **fall back to Camera** (`captureMode='camera'`, `requestCamera()`).
- Placement: bottom, just above the Tall/Standard/Small format selector.

### Punch crop (the fix)
Rewrite `cropFeed` to be cover-aware for whichever media is live:
1. `mediaEl` = the video (camera) or the picked `<img>` (gallery); intrinsic `mediaW/mediaH`.
2. Read the element's on-screen rect (already reflects the `feedView` transform).
3. Compute `coverScale = max(rectW/mediaW, rectH/mediaH)` and the centered content offset
   `contentLeft/contentTop` (content overflows the rect and is center-clipped).
4. Map the aperture rect (`#capture-hit`) into source pixels via `coverScale` + offsets,
   clamp to `[0,mediaW]×[0,mediaH]`, then `drawImage` the crop and the full-source export.
This single path serves both camera and gallery since both are full-bleed `cover` media.

### Unchanged
Review screen, Save Stamp, stored stamp object, grid, collection detail, reposition/adjust
flow, formats, quota/paywall. Only the capture screen layout + `cropFeed` change.

## Fallbacks / edge cases
- Camera denied/unsupported in Camera mode → "camera unavailable" state nudging toward Gallery.
- Gallery cancelled → back to Camera.
- Punch with no media ready → returns the existing empty `fallback` (no crash).

## Implementation notes (as built)
- Pan/zoom model changed from "resize a card sized to the aperture" to a transform on a
  full-bleed element: `transform: translate(x,y) scale(s)` with `object-fit: cover`.
  `clampFeedView` is now viewport-based; default `feedView.s` is `1`.
- `cropFeed` computes the cover scale + centering offset and maps the aperture into source
  pixels (clamped to the source), replacing the old width-only scale that read back black.
- Camera lifecycle: the Reposition/adjust screen also keeps the camera alive
  (`adjustScreenRef`, camera-mode only) and `requestCamera` now permits the `adjust` screen,
  so live-camera repositioning shows the feed instead of the mat.

## Out of scope
Any change to `index.html`, `image-slot.js`, `support.js`, or the non-capture screens.
