# habtracker — Habit Tracker

A single-page habit tracker with weekly grid view, streaks, and full persistence.

## How to Run

This is a zero-dependency single HTML file. No build step, no npm install.

**Option 1 — Open directly:**
```bash
open index.html
# or double-click index.html in your file manager
```

**Option 2 — Local dev server (recommended to avoid any CORS quirks):**
```bash
# Python 3
python3 -m http.server 8080
# then visit http://localhost:8080

# Node.js (npx, no install needed)
npx serve .
# then visit the URL shown
```

No dependencies. No build tools. Works in any modern browser (Chrome,Brave, Safari, Edge).

## Features

- Add, rename, delete habits
- Weekly grid (Mon–Sun) with checkmark toggles
- Today's column highlighted
- Streak counter per habit (updates in real-time)
- Week navigation (prev/next, back to this week)
- Summary bar: today's completion, week total, best streak, progress bar
- Full persistence via `localStorage`
- Responsive: works at 360px–1440px+
- Keyboard accessible throughout