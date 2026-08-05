# OneStamp Chrome Extension — Technical Specification

## Overview
OneStamp is a memory collection extension that replaces your new tab page with a personal stamp gallery. Users capture moments (photos/screenshots), tag them with a collection name, and browse via three views: by collection, by week, or timeline.

---

## Architecture

### File Structure
```
manifest.json          # Extension config
background.js         # Service worker
newtab.html          # New tab UI
newtab.js            # App logic
icons/               # Extension icons (16, 48, 128px)
```

### Data Storage
**Chrome Storage API** (`chrome.storage.local`)
- Key: `stamps`
- Value: Array of stamp objects

**Stamp Object Schema**
```javascript
{
  id: "1234567890",           // Unix timestamp as string
  name: "Coffee with Sarah",  // User-entered title
  description: "...",         // Optional note
  collection: "Friends",      // User-defined tag (one per stamp)
  image: "data:image/jpeg;base64,...", // Base64 image data
  createdAt: "2024-08-04T14:32:00Z"    // ISO timestamp
}
```

---

## Features — Phase 1

### 1. New Tab Page (Main View)
**Default state:** Collection view, empty placeholder if no stamps

**Three Toggle Buttons:**
- **By Collection** — Groups stamps by their collection tag (e.g., all "Friends" stamps together)
- **By Week** — Groups by ISO week; weeks with no stamps hidden
- **Timeline** — Chronological single-date groupings; linear scroll top-to-bottom

**Visual:**
- Minimal, soulful aesthetic (soft typography, generous whitespace, muted palette)
- Warm off-white background (#faf8f6)
- Charcoal text (#2a2a2a)
- Subtle shadows, rounded corners (8px standard)
- Stamps appear as cards in collection/week views; small thumbnails in timeline

**Empty State:**
- "No stamps yet" + "Click Add Stamp to create your first memory"

---

### 2. Capture Modal
**Triggered:** Click "+ Add Stamp" button

**Two Capture Methods:**

#### Upload Tab
- File input (drag-drop + click)
- Accept: images only
- Preview image before saving
- Persists across tab switches

#### Camera Tab
- "Start Camera" button → requests permission
- Live video feed (if granted)
- "Capture Photo" button → freezes frame, converts to base64
- Falls back to upload if permission denied

**Form Fields:**
- **Name** (required) — text input, max ~100 chars
- **Description** (optional) — textarea, ~80px min-height
- **Collection** (required, default "Uncategorized") — text input, user-defined

**Actions:**
- Save Stamp → validates (name + image), stores to `chrome.storage.local`, closes modal, re-renders view
- Cancel → closes modal, clears form

---

### 3. Detail View (Full Screen)
**Triggered:** Click any stamp (card or thumbnail)

**Header:**
- Back arrow (left) → closes detail, returns to previous view
- Download + Delete buttons (right)

**Content:**
- Full-size image (max 400px height, contain)
- Stamp name (large, 32px)
- Created date (formatted: "Monday, August 4, 2024")
- Description (if exists)
- Collection tag in metadata row

**Actions:**
- **Download** — triggers browser download as `{name}-{id}.jpg`
- **Delete** — confirms deletion, removes from storage, re-renders view

---

### 4. Storage & Sync
**Persistence:** `chrome.storage.local` (per-device, survives browser restart)
- On extension install: init empty `stamps` array if not present
- On capture save: `unshift()` new stamp (newest first)
- On delete: filter by ID, re-save array
- No cloud sync in Phase 1

---

## UX Flows

### Add Stamp Flow
1. User opens new tab → default collection view
2. Clicks "+ Add Stamp" → capture modal opens
3. Selects upload or camera → selects/captures image
4. Enters name + optional description
5. Selects or types collection name
6. Clicks "Save" → stamp saved, modal closes, view re-renders with new stamp at top

### Browse Stamps
1. User toggles between views (collection/week/timeline)
2. Views re-render instantly (no loading)
3. Clicking any stamp → detail view opens full-screen
4. User reads metadata, downloads, or deletes
5. Back arrow → returns to previous view/scroll position (preserved)

---

## Design System

### Colors
- Background: #faf8f6 (warm white)
- Text: #2a2a2a (charcoal)
- Borders: #e8e6e3 (light taupe)
- Secondary: #999 (soft gray)
- Accents: #c4a574 (warm gold for collection dots)

### Typography
- Font stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif`
- Sizes:
  - Page title: 32px, weight 300
  - Modal header: 24px, weight 500
  - Stamp card name: 14px, weight 500
  - Detail name: 32px, weight 500
  - Body: 14px–15px, weight 400
  - Labels: 13px, weight 600, uppercase, +0.5px letter-spacing

### Spacing
- Container padding: 48px (top/bottom), 32px (left/right)
- Card gap: 24px
- Modal padding: 40px
- Form group margin: 24px bottom

### Corners
- Standard border-radius: 8px
- Buttons: 24px (pill-style for tertiary)
- Cards: 8px

### Shadows
- Cards (rest): `0 2px 8px rgba(0,0,0,0.06)`
- Cards (hover): `0 8px 16px rgba(0,0,0,0.12)`
- Modal: `0 20px 60px rgba(0,0,0,0.3)`

### Interactions
- Hover: subtle lift (translateY -4px) + enhanced shadow
- Focus: 3px outline ring, dark color
- Transitions: 0.2s all (ease-in-out default)

---

## Implementation Notes

### Base64 Images
- All images stored as data URIs (no external file storage)
- Pros: Offline-ready, no backend, syncs within Chrome storage limits (~10MB per extension)
- Cons: Larger storage footprint; future phase may move to IndexedDB or cloud

### Camera Access
- Requires user permission (browser prompt on first use)
- Permission persists; no re-prompt per session
- Falls back gracefully if denied

### View State
- Current view persists in memory during session
- On new tab open: defaults to collection view
- Scroll position not preserved (future enhancement)

### Grouping Logic
- **Collection:** by `stamp.collection` field, alphabetical or insertion order
- **Week:** ISO week (Monday start); empty weeks hidden
- **Timeline:** by `createdAt` date (YYYY-MM-DD); newest first

---

## Future Phases

### Phase 2
- Camera feed improvements (filters, timer)
- Bulk operations (multi-select, export collection as zip)

### Phase 3
- Screen punch capture (screenshot entire tab or region)
- Extracting text from images (OCR-lite for searchability)

### Phase 4
- Cloud sync (optional, user-controlled)
- Theme toggle (dark mode, accent colors)
- Sharing (shareable collection links, not individual stamps in v1)
- Search & tag filtering

---

## Testing Checklist (Phase 1)

- [ ] Extension loads as unpacked extension
- [ ] New tab override works; opens extension page
- [ ] Collection/week/timeline toggle renders correctly
- [ ] Add stamp modal opens/closes
- [ ] File upload works with preview
- [ ] Camera access prompt appears; capture works
- [ ] Form validation (name + image required)
- [ ] Stamps save to storage and persist on reload
- [ ] Detail view opens/closes correctly
- [ ] Download button triggers file download
- [ ] Delete button removes stamp and re-renders
- [ ] Empty state shows when no stamps
- [ ] Responsive on typical monitor widths (1920, 1440, 1024px)
