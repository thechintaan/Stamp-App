# OneStamp Chrome Extension — Phase 2 design (Memory Wall)

**Date:** 2026-08-04
**Supersedes/refines:** the divergent parts of `EXTENSION_SPEC.md` (this doc is the source of truth for the memory-wall direction).

## Concept

Replace the Chrome new tab with a **memory wall**: not a folder tree, a *desk pile* of
collection "specimen cards." The framing — *time isn't passing you by, you're collecting
stamps of it* — drives every visual choice: physical, tactile, collectible.

## Foundation decision

Reuse the existing OneStamp app (`index.html`) — its stamp rendering, capture/punch flow,
full-screen collection grid, and stamp detail (share/download) — as the extension's content.
No visual rebuild. The extension adds: the new-tab wall, the collection-card component, and a
storage layer swap.

## Screens & components

### 1. New tab — the Memory Wall
- A **scattered pile of collection mat cards**: each card tilted a few degrees, overlapping
  its neighbors, resting on the app's mat/desk background.
- **Collection / Week toggle** regroups the same cards (Collection = by `collection` tag;
  Week = by ISO week, empty weeks hidden).
- **Newest collection on top.** Hovering a card **raises it to the front** (gentle lift).
- A **"spread" toggle**: default is the resting pile; one tap fans all cards into a scannable
  layout (and back). Beauty by default, scannability on demand.
- **"+ Add"** opens the existing capture/punch flow to create a new stamp.
- Empty state: a single "start your first collection" prompt.

### 2. Collection mat card (the signature component)
One card **represents one collection (or one week)**. On its surface:
- Kraft **cutting-mat texture** + ruler edge (matches the capture screen's mat).
- **Title plate** (collection/week name) + **count badge** (e.g. "30").
- The **~5–6 most-recent stamps fanned** (overlapping, slight random rotation), then a
  **"+N"** indicator for the remainder (e.g. 6 shown, "+15").
- Mat **tone can vary per collection** (kraft / green / slate …) so the wall reads as a varied,
  collectible set — optional polish, not required for v1.
- Tap the card (or "+N") → the collection full-screen.

### 3. Collection — full screen
Tap a card → the **existing full-screen collection grid**, showing all stamps in that
collection. Reuses the current screen; back returns to the wall.

### 4. Stamp detail
Tap a stamp → the **existing detail screen**: full image, name, date, description, collection,
with **Download (↓)** and **Share (↗)**.

## Data & storage

- **Metadata** (stamp records, collections, view state) → `chrome.storage.local`.
- **Images** → **IndexedDB** (blobs), referenced by id from the metadata. This avoids the
  ~10 MB base64 ceiling a growing memory wall would blow past. `chrome.storage.local` holds
  small records; IndexedDB holds the heavy image data.
- **Stamp record (proposed):**
  ```
  { id, name, description, collection, weekKey, imageRef, createdAt, formatKey, rotation }
  ```
  `imageRef` points to the IndexedDB blob; `weekKey` is a precomputed ISO week for fast Week
  grouping; `formatKey`/`rotation` carry the existing stamp's shape/tilt.
- **Migration:** one-time import of the current app's persisted stamps into the new layer on
  first run.
- No cloud sync in Phase 2.

## Technical constraints (must resolve during build)

- **MV3 CSP / no runtime Babel.** The current `index.html` compiles JSX at runtime via
  `@babel/standalone` (needs `eval`/`new Function`), which MV3 extension pages **forbid**
  (`unsafe-eval` is not allowed). The app must be **pre-compiled** (JSX → JS ahead of time)
  and shipped as static files. This is the main porting task, not a rewrite.
- **Camera** (`getUserMedia`) works on extension pages (secure context) — the full-screen
  camera + gallery work from the earlier phase carries over.
- **No remote code.** React/Babel/etc. must be bundled locally (no unpkg CDN at runtime).

## File structure (proposed)
```
manifest.json          # MV3, chrome_url_overrides.newtab -> newtab.html
newtab.html            # Memory wall entry (pre-compiled app)
app.js                 # Pre-compiled OneStamp app + wall
storage.js             # chrome.storage.local + IndexedDB data layer
image-slot.js          # reused as-is (local)
icons/                 # 16/48/128
```

## Scope

**In:** new-tab wall (pile + spread + Collection/Week toggle), collection mat card, reuse of
capture/collection/detail, storage swap (chrome.storage.local + IndexedDB), migration, build
step to remove runtime Babel.

**Out (later phases):** cloud sync, search/filter, per-collection mat theming beyond a basic
palette, screen-region capture, OCR, dark mode.

## Open questions for implementation
- Exact spread animation and pile physics (rotation range, overlap amount, z-order rules).
- Whether "+ Add" lands in a chosen collection or asks for one at save (current app asks at
  save — keep that).
- Build tooling for the pre-compile step (esbuild/vite vs. a one-off compile of the current
  inline component).
