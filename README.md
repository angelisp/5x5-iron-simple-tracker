# 5x5 Iron Simple Tracker (`5x5-iron-simple-tracker`)

A fast, zero-dependency, offline-first **$5 \times 5$ Progressive Overload Barbell Strength Tracker** designed for mobile gym use and installable as a native full-screen **WebAPK** on Google Pixel / Android devices.

---

## Features

- **Workout A / B Auto-Alternation:** Automatically tracks which training day is **UP NEXT** (`Workout A`: Squat, Bench Press, Barbell Row; `Workout B`: Squat, Overhead Press, Deadlift $1 \times 5$).
- **One-Tap Gym Set Circles:** Tap any empty circle once to log `5` completed reps. Tap again to cycle down (`4`, `3`, `2`, `1`, `0`) if you miss a rep.
- **Smart Floating Rest Timer:** Automatically launches a **1:30** rest timer after a completed set of 5 (with a quick **3:00 Hard** toggle) or a **5:00** recovery timer after a missed set, with Web Audio synthesized chimes.
- **Barbell Plate & Warm-Up Calculator:** Calculates the exact plates needed per side on a standard $20\text{ kg}$ Olympic barbell alongside progressive warm-up ramps ($50\%$, $70\%$, $85\%$).
- **Linear Progression & $10\%$ Deload Engine:** Automatically adds $+2.5\text{ kg}$ ($+5.0\text{ kg}$ on Deadlift) after every successful session and suggests a $10\%$ deload after 3 consecutive stalls on any lift.
- **Full Data Portability:** Local-first `localStorage` persistence with active-session draft auto-save, interactive SVG strength progression charts, **CSV export**, and **JSON backup/restore**.

---

## Repository File Structure

Your `5x5-iron-simple-tracker` repository root should contain these 4 files:

```text
5x5-iron-simple-tracker/
├── index.html      # Main single-file application (with PWA meta + SW registration)
├── manifest.json   # Web App Manifest for Android WebAPK installation
├── sw.js           # Offline Service Worker caching shell & styles
└── README.md       # Documentation & setup instructions
```

> **Tip:** You can either copy `manifest.json` and `sw.js` from the code blocks below, or click **History & Charts -> PWA Repo Files** inside the live `index.html` preview to download both files with one click.

---

## Companion PWA Files for Your Repository

### 1. `manifest.json`
Create a file named `manifest.json` in the root of `5x5-iron-simple-tracker`:

```json
{
  "name": "5x5 Iron Simple Tracker",
  "short_name": "5x5 Iron",
  "description": "Offline-first 5x5 progressive overload barbell strength workout tracker.",
  "start_url": "./",
  "scope": "./",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#090d16",
  "theme_color": "#090d16",
  "icons": [
    {
      "src": "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 512 512'%3E%3Crect width='512' height='512' rx='112' fill='%23090d16'/%3E%3Cpath d='M112 176h48v160h-48zm240 0h48v160h-48zM64 208h32v96H64zm352 0h32v96h-32zM160 240h192v32H160z' fill='%2310b981'/%3E%3C/svg%3E",
      "sizes": "192x192 512x512",
      "type": "image/svg+xml",
      "purpose": "any maskable"
    }
  ]
}
```

### 2. `sw.js`
Create a file named `sw.js` in the root of `5x5-iron-simple-tracker`:

```javascript
const CACHE_NAME = '5x5-iron-simple-tracker-v1';
const ASSETS = [
  './',
  './index.html',
  './manifest.json',
  'https://cdn.tailwindcss.com',
  'https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@500;600;700;800&display=swap'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(ASSETS))
  );
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(
        keys.map((key) => {
          if (key !== CACHE_NAME) return caches.delete(key);
        })
      )
    )
  );
  self.clients.claim();
});

self.addEventListener('fetch', (event) => {
  if (event.request.method !== 'GET') return;
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      return (
        cachedResponse ||
        fetch(event.request).then((networkResponse) => {
          return caches.open(CACHE_NAME).then((cache) => {
            cache.put(event.request, networkResponse.clone());
            return networkResponse;
          });
        })
      );
    })
  );
});
```

---

## Deploying to GitHub Pages & Installing on Google Pixel

1. **Upload the Files:**
   - Open your `5x5-iron-simple-tracker` repository on GitHub.
   - Click **Add file -> Upload files** and drag in `index.html`, `manifest.json`, `sw.js`, and `README.md`.
   - Click **Commit changes**.

2. **Enable GitHub Pages:**
   - Go to **Settings -> Pages** in your repository.
   - Under **Build and deployment**, set **Source** to **Deploy from a branch**.
   - Select the **`main`** branch and **`/ (root)`** folder, then click **Save**.
   - After ~60 seconds, your app will be live at:
     `https://<your-github-username>.github.io/5x5-iron-simple-tracker/`

3. **Install as a Standalone WebAPK on Your Pixel:**
   - Open `https://<your-github-username>.github.io/5x5-iron-simple-tracker/` in **Chrome** on your Google Pixel.
   - Tap the **Install App** button in the top header (or tap Chrome's three-dot menu and select **Add to Home screen -> Install**).
   - Android will compile and install **5x5 Iron** directly into your Pixel app drawer with full offline support and zero browser chrome.