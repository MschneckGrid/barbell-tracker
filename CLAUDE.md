# Barbell Log — Weight Training Tracker

## Project Overview
Mobile-first PWA for tracking barbell training workouts. Logs sets, reps, and weight for Olympic and powerlifting movements. Calculates estimated 1RM using the Epley formula. All data stored in Supabase.

## Tech Stack
- **Frontend**: Single-file HTML/CSS/JS (no build step)
- **Backend**: Supabase (PostgreSQL + REST API)
- **Hosting**: Netlify (static site)
- **PWA**: Service worker + manifest for home screen install

## Supabase Details
- **Project**: Gridiron Performance
- **Project ID**: `nzinvxticgyjobkqxxhl`
- **URL**: `https://nzinvxticgyjobkqxxhl.supabase.co`
- **Table**: `public.workout_sets`
- **RLS**: Disabled (single-user app)

## Database Schema — `workout_sets`
| Column       | Type       | Notes                                    |
|-------------|------------|------------------------------------------|
| id          | bigint     | Auto-generated identity                  |
| exercise    | text       | CHECK constraint for valid exercises     |
| sets        | integer    | Number of sets (≥1)                      |
| reps        | integer    | Reps per set (≥1)                        |
| weight_lbs  | numeric    | Weight in pounds (>0)                    |
| workout_date| date       | Defaults to CURRENT_DATE                 |
| notes       | text       | Optional                                 |
| created_at  | timestamptz| Auto                                     |
| updated_at  | timestamptz| Auto                                     |

## Valid Exercises
Back Squat, Front Squat, Snatch, Push Press, Strict Press, Jerk, Hang Clean, Hang Clean & Jerk

## Indexes
- `idx_workout_sets_exercise` — exercise lookups
- `idx_workout_sets_date` — date-sorted queries
- `idx_workout_sets_exercise_date` — combined exercise + date

## Key Features
- Exercise dropdown with 8 barbell movements
- Multi-row set entry (add rows for different rep schemes)
- Current max weight display per exercise
- Estimated 1RM using Epley formula: `weight × (1 + reps/30)`
- Chronological workout history grouped by date
- PWA installable on mobile home screen
- Dark theme optimized for gym use

## File Structure
```
barbell-tracker/
├── index.html       # Full app (HTML + CSS + JS)
├── manifest.json    # PWA manifest
├── sw.js            # Service worker
├── icon-192.png     # PWA icon
├── icon-512.png     # PWA icon
├── netlify.toml     # Netlify config
└── CLAUDE.md        # This file
```

## Deployment
- Push to GitHub repo
- Connect repo to Netlify
- Auto-deploys on push to main

## Common Claude Code Commands
```bash
# Run locally
npx serve .

# Push to GitHub
git add -A && git commit -m "update" && git push

# Add new exercise to the CHECK constraint
# UPDATE the CHECK in Supabase SQL editor AND the <select> in index.html
```

## Future Enhancements to Consider
- Volume tracking (total tonnage per session)
- Progress charts over time
- Workout templates / saved routines
- Bodyweight tracking
- Rep PR badges
- Export to CSV
