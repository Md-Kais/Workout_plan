# Weekly Gym Plan — README

A fully self-contained single-file web app for tracking a personal weekly gym program. No server, no database, no dependencies to install — just open the HTML file in any browser.

---

## What it does

- Displays a structured weekly workout plan across 6 training days (Sun–Thu + Fri/Sat rest)
- Tracks completed exercises and full training days using browser localStorage
- Auto-opens today's day tab based on the device's current date
- Shows a weekly progress panel on the Overview screen
- Pulls a daily motivational quote from a free public API
- Works offline and can be installed to the home screen as a PWA

---

## File structure

The entire app lives in a single file — `weekly_gym.html` — divided into four parts:

### 1. `<head>` — Meta & fonts
PWA metadata tags (theme color, apple-mobile-web-app), Google Fonts (DM Sans + DM Mono), and the PWA manifest `<link>` tag (the manifest itself is generated at runtime via a Blob URL, so no separate file is needed).

### 2. `<style>` — CSS
All styling is written with CSS custom properties (`--bg-card`, `--text-p`, etc.) defined in `:root`. Key sections:
- **Tokens** — color palette, spacing radii, legacy variable aliases
- **Layout** — `.app-wrap`, `.app-header`, `.day-tabs`
- **Quote banner** — motivation strip at the top with refresh button
- **Progress panel** — weekly dots row, progress bar, reset button
- **Timeline** — `.tl-item` / `.tl-card` grid layout for each exercise block
- **Tick button** — `.tick-btn` positioned absolute at bottom-right of every card
- **Toast** — fixed bottom notification for user feedback

### 3. `<body>` — HTML shell
Static markup is minimal — just the header, the quote banner, the tab bar, an empty `#content` div, and the toast element. All workout content is rendered dynamically by JavaScript into `#content`.

### 4. `<script>` — JavaScript
The script is organised into these sections:

| Section | What it does |
|---|---|
| **Color tokens (`C`)** | Named color objects (blue, teal, amber, etc.) used across all data |
| **Data** | `overviewWeeks`, `equipment`, `days[]` — all workout content as plain JS arrays |
| **Storage** | `loadState()` / `saveState()` — reads/writes to localStorage using an ISO week key (`gymtracker_YYYY-Www`); old week keys are deleted automatically on load |
| **Tracking helpers** | `isDayDone()`, `isBlockDone()`, `toggleBlock()`, `toggleDayDone()`, `resetWeek()`, `updateTabBadges()` |
| **Quotes** | `fetchQuote()` — tries QuoteSlate API first, falls back to DummyJSON, then falls back to a built-in pool of 15 quotes |
| **Toast** | `toast(msg)` — shows a brief bottom notification |
| **Date logic** | `getTodayTabIndex()` maps `Date.getDay()` to the correct tab index; `markTodayTab()` adds the teal dot indicator |
| **Render** | `showDay(i)` — builds and injects all HTML for either the Overview (i=0) or a specific day (i=1–6) |
| **PWA setup** | `setupPWA()` — creates the manifest via `Blob` + `URL.createObjectURL()` and registers an inline service worker (also via Blob) for offline caching |
| **Init** | Bottom of script — calls `setupPWA()`, `fetchQuote()`, `markTodayTab()`, `updateTabBadges()`, and `showDay(todayIndex)` on page load |

---

## Data storage

Workout progress is stored in `localStorage` with weekly auto-expiry:

- **Key format:** `gymtracker_2026-W12` (ISO year + week number)
- **Value:** JSON object with two maps — `dayDone` (day index → bool) and `blockDone` ("`dayIdx_blockIdx`" → bool)
- **Expiry:** On every save, any key starting with `gymtracker_` that isn't the current week is deleted
- No data is ever sent anywhere — everything stays in the browser

---

## Quotes API

| Priority | Source | Notes |
|---|---|---|
| 1st | [QuoteSlate](https://quoteslate.vercel.app/api/quotes/random) | Free, no API key, CORS enabled, filtered by tags like `motivation`, `discipline` |
| 2nd | [DummyJSON](https://dummyjson.com/quotes/random) | Free, no API key, reliable fallback |
| 3rd | Built-in pool | 15 hardcoded fitness quotes — works fully offline |

---

## PWA installation

The app registers a service worker and manifest entirely from Blob URLs — no web server required. To install:

- **Android (Chrome):** tap the three-dot menu → "Add to Home Screen"
- **iOS (Safari):** tap the Share icon → "Add to Home Screen"

Once installed, the app works offline using the service worker cache.
