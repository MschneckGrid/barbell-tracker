# Barbell Log — Weight Training Tracker

## Project Overview
Mobile-first PWA for tracking barbell training workouts. Logs sets, reps, and weight for 8 Olympic and powerlifting movements. Calculates estimated 1RM using the Epley formula. All data stored in Supabase with a single-file frontend deployed on Netlify.

## Tech Stack
- **Frontend**: Single-file HTML/CSS/JS (`index.html`) — no build step, no bundler
- **Fonts**: DM Sans (UI) + DM Mono (numeric data) via Google Fonts
- **Design**: White/light theme, blue accent (`#2563eb`), 14px border radius, mobile-first
- **Backend**: Supabase (PostgreSQL + REST API via `@supabase/supabase-js` v2 CDN)
- **Hosting**: Netlify (static site, auto-deploys on push to `master`)
- **PWA**: Service worker (`sw.js`) + manifest (`manifest.json`) for home screen install
- **Charts**: Pure canvas rendering (no Chart.js or external chart library)

## Supabase Details
- **Project**: mike@gridironpartners.com's Project (dedicated personal-apps project, isolated from Gridiron Performance and sidebet-db)
- **Project ID**: `cgneuzcynfechjjhdega`
- **URL**: `https://cgneuzcynfechjjhdega.supabase.co`
- **Table**: `public.workout_sets`
- **RLS**: Enabled (2026-11-03). Policy `workout_sets_owner_all` allows only Mike's auth user (mikeschneck@gmail.com, id 75e97010-a886-4e92-9aab-d3afd192dcd8); anon has no grants. The app signs in with a one-time email link (Supabase OTP, `shouldCreateUser: false`); without a session the dashboard shows the sign-in card and no data.
- **History**: Do NOT host this table in the Gridiron Performance or sidebet-db projects. It previously lived in both; a sidebet-db "drop gridiron cruft" migration (20260720151624) destroyed the table and all workout history on 2026-07-20.

## Database Schema — `workout_sets`
| Column       | Type        | Constraints / Notes                      |
|-------------|-------------|------------------------------------------|
| id          | bigint      | Auto-generated identity (PK)             |
| exercise    | text        | CHECK constraint for 8 valid exercises   |
| sets        | integer     | Number of sets (CHECK >= 1)              |
| reps        | integer     | Reps per set (CHECK >= 1)                |
| weight_lbs  | numeric     | Weight in pounds (CHECK > 0)             |
| workout_date| date        | Defaults to CURRENT_DATE                 |
| notes       | text        | Optional, nullable                       |
| created_at  | timestamptz | Auto-set on insert                       |
| updated_at  | timestamptz | Auto-set on insert/update                |

## Valid Exercises
Back Squat, Front Squat, Deadlift, Snatch, Push Press, Strict Press, Jerk, Hang Clean, Hang Clean & Jerk

## Indexes
- `idx_workout_sets_exercise` — exercise lookups
- `idx_workout_sets_date` — date-sorted queries
- `idx_workout_sets_exercise_date` — combined exercise + date

## App Architecture

### Dashboard Page
- Shows all 8 exercises as tappable cards
- Each card displays:
  - Exercise name with SVG right chevron
  - **Last workout**: heaviest set from most recent session (weight x reps)
  - **Estimated 1RM**: best all-time Epley calculation (`weight * (1 + reps/30)`)
  - Relative date (e.g. "2 days ago") and total set count from last session
  - "No sets logged yet" for exercises with no data
- Single Supabase query fetches all data, grouped client-side by exercise

### Exercise Detail Page
- Navigated to by tapping a dashboard card
- Back button returns to dashboard (reloads dashboard data)
- **Log Workout form**: weight, reps, sets inputs + date picker + optional notes field + add row button
- **1RM Trend Chart**: pure canvas line chart between log form and history
  - Best estimated 1RM per workout date
  - Blue gradient area fill under the line
  - Y-axis gridlines with weight labels, X-axis date labels
  - Last data point highlighted with larger dot + value label
  - High-DPI canvas rendering (devicePixelRatio scaling)
  - Shows "Log 2+ workouts to see trends" placeholder when < 2 dates
- **Recent History**: grouped by date, each entry shows sets x reps @ weight + e1RM
  - Delete button on each entry (with confirm dialog)
- Chart and history re-render on save and delete

### Key Formulas
- **Adjusted Epley 1RM**: `weight * (1 + (reps + ASSUMED_RIR) / 30)` with `ASSUMED_RIR = 2` (Mike never trains to failure)
- **1RM stats and trend** (dashboard Best/Recent 1RM, chart) use only sets with reps <= `TREND_MAX_REPS` (5). Higher-rep sets log and show in history as "not in trend".
- **Relative dates**: Today, Yesterday, N days ago, N weeks ago, N months ago

## File Structure
```
barbell-tracker/
├── index.html       # Full app (HTML + CSS + JS in single file)
├── manifest.json    # PWA manifest
├── sw.js            # Service worker for offline caching
├── icon-192.png     # PWA icon (192x192)
├── icon-512.png     # PWA icon (512x512)
├── netlify.toml     # Netlify deployment config
└── CLAUDE.md        # This file
```

## Deployment
- GitHub repo: `MschneckGrid/barbell-tracker`
- Branch: `master` (not main)
- Push to `master` triggers Netlify auto-deploy
- No build command needed — static files served directly

## Development
```bash
# Run locally
npx serve .

# Add new exercise
# 1. Update CHECK constraint in Supabase SQL editor
# 2. Add to EXERCISES array in index.html JS

# Commit and deploy
git add index.html && git commit -m "feat: description" && git push origin master
```

## Design Decisions
- Single-file architecture keeps deployment simple and avoids build tooling
- No external chart library — pure canvas for the 1RM trend chart reduces bundle size
- SVG icons inline (chevrons, back arrow) instead of icon fonts or CSS unicode
- All Supabase calls use the JS client library (CDN), not raw fetch
- Dashboard loads all workout data in one query and groups client-side for simplicity

## Future Enhancements to Consider
- Workout templates / saved routines
- Bodyweight tracking
- Rep PR badges / personal record notifications
- Export to CSV
- Volume tracking (total tonnage per session) on dashboard cards

## 2026-11-05: Sign-in removed
Email sign-in removed at Mike's request (magic link redirected to another app). RLS stays on; policy `workout_sets_anon_all` grants anon full access. App opens straight to data, no login.
