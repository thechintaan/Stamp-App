# OneStamp

A mobile-first app for punching circular photo stamps and sharing them with custom backgrounds.

## Features

- 📸 Camera capture with real-time preview
- 🎯 Adjustable punch aperture in three formats (tall, standard, small)
- 🎨 5 share card variants with sampled gradients from photo colors
- 📱 Full offline support via bundled HTML
- 💾 Collections and stamp management
- 📥 Import from camera roll or take live photos

## Getting Started

### Local Development

```bash
python -m http.server 3000
# or
npx http-server -p 3000
```

Visit `http://localhost:3000/OneStamp.html`

### Deploy to Vercel

```bash
npm run deploy
```

Or drag `OneStamp.html` into Vercel's dashboard.

## Technical Notes

- Built as a self-contained HTML Design Component (DC)
- Uses Canvas API for stamp rendering and export
- Camera access requires HTTPS (except localhost)
- Photos are stored in browser memory during the session
- Share cards export as PNG with perforated edges and custom backgrounds
