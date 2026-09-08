# Forge architecture

## Product and design
Forge is a private strength-training tracker. Next.js App Router, React, TypeScript, Tailwind CSS, Supabase Auth, and PostgreSQL implement the approved plan. The interface follows the Blackplate design system described in DESIGN_BRIEF.md: near-black neutral surfaces, bone-white ink, and a single ember accent reserved for the primary action and the completed state, with a white-hot mark for all-time bests. Barlow Condensed carries every number, Inter carries the interface, and both are self-hosted through next/font. Every colour is a semantic custom property on :root in app/globals.css, so the Daylight variant under prefers-color-scheme:light is a token override rather than a second stylesheet. Desktop is a 64px icon rail with no top bar and a single content column; phones keep a four-tab bottom bar with a top bar for the settings entry point. Text contrast is at least 4.5:1, interactive boundaries at least 3:1, controls at least 44px, focus is always visible, and reduced motion is respected.

## System boundaries
Server components load authenticated initial data. Client components own unsaved form state and call same-origin Next.js route handlers. Services validate every request and use the caller's Supabase session. PostgreSQL row-level security independently enforces ownership. The application never uses a service-role key for user requests.
Supabase email/password supports confirmation, recovery, and cookie sessions. Missing configuration displays a setup screen rather than accepting nonpersistent workout data. No demo records are mixed into real accounts.

## Database
- profiles: authentication user ID, display name, kg/lb preference, IANA timezone, timestamps.
- exercises: nullable owner (null means standard), name, original description, muscle group, equipment, weighted/bodyweight/assisted tracking type, archive timestamp.
- workout_sessions: user, local workout date, start/completion timestamps, title, notes, revision, last mutation ID.
- workout_exercises: session, exercise, position, notes, rest seconds.
- exercise_sets: workout exercise, set number, working/warmup type, decimal kg weight, input unit, reps, completion timestamp.
- workout_templates: user, name, archive timestamp, revision, last mutation ID.
- template_exercises: template, exercise, position, target sets, target reps, optional target rep ceiling for a range, rest seconds.
All records use UUID keys. Owned relationships are protected by policies on both reads and writes. Child foreign keys cascade when a workout is explicitly deleted. Exercise and template removal is archival. Standard seeds use deterministic IDs.

## Writes and retries
Whole-workout and whole-template saves use transactional PostgreSQL functions. Client-generated IDs preserve record identity. Revision checks reject stale writes. A stable mutation ID lets an interrupted save retry without duplicate records. Children cannot be moved between unrelated parents. Copied sets receive fresh IDs and remain incomplete. Templates instantiate as independent sessions.

## Interfaces
- GET /api/bootstrap?date=YYYY-MM-DD: profile, active catalog, selected day's sessions, last 84 days of completed history, and templates.
- GET /api/history/:exerciseId?weeks=4|12|all&page=1: date-range session aggregates and 10 sessions of set details; includes previous-session information.
- PUT /api/profile: profile preferences.
- POST /api/exercises; PUT /api/exercises/:id; DELETE /api/exercises/:id: private custom catalog management.
- PUT /api/workouts/:id: transactional versioned session save; DELETE removes an owned session.
- PUT /api/templates/:id: transactional versioned routine save; DELETE archives it.
- GET /api/health: readiness without secrets.
All bodies have bounded schema validation. Errors carry a safe message. Auth is mandatory except readiness. Mutations require matching request origin. Application pages are dynamic and private responses are never shared-cacheable.

## Progression
Only completed working sets count. Weighted charts show heaviest weight or sum(weight x reps); bodyweight shows maximum reps; assisted records use minimum assistance for a selected repetition count. No bodyweight or assisted loads are included in weighted volume. Dates and Monday week boundaries use stored workout dates. Current week comparisons are labelled incomplete. Missing activity has null chart values. Canonical kg is never rewritten when display units change. History and records derive from current data, so edits and deletions take immediate effect. Next targets use double progression: if every completed working set at the top load of the previous session reached the first set's repetitions, the target adds 2.5 kg or 5 lb (bodyweight adds one repetition; assisted subtracts the step); otherwise the target repeats the load. Personal bests compare a logged set against all-time history excluding the current workout. The weekly muscle summary counts completed working sets per muscle group for the current and previous Monday-based weeks.

## UI flows
Today supports multiple sessions, ordered exercises, copy-previous drafts, set saves, completion, and retry feedback. Quick entry is the alternative to set-by-set logging: exercises are added from the catalog picker and each row carries weight, reps, and a set count on unit-aware steppers, prefilled from the same double-progression target, so a whole session saves as one completed workout in a single request; bodyweight movements drop the weight column. A secondary text mode still parses typed lines through lib/quick.ts. Both modes produce the same parsed lines and share one save path. Exercise library supports name, muscle, and equipment filters and custom creation/edit/archive. Dedicated exercise history provides 4-week/12-week/all-time ranges, metrics, records, and paginated details. Routines specify ordered exercises, sets, reps, and rests. Each routine exercise can take a named set and rep scheme from lib/schemes.ts, grouped by goal (strength, hypertrophy, endurance, power, general fitness), which sets the set count, reps, rep ceiling, and rest together; editing any of those fields returns the selector to Custom. Reps are either a single target or a range such as 8-12, stored as target_reps with a nullable target_reps_max and rendered as "3 x 8-12". Instantiating a routine seeds the bottom of the range as each set's target. Timer stores only its device-local end time. Settings handles units, timezone, profile, and sign-out. Each exercise card shows the previous session, the all-time best, and a next target that can fill the unlogged sets in one tap; a personal best shows a toast and a trophy on the set. The status line reports offline state, and a failed save retries once the connection returns; drafts never leave memory. The app ships a web manifest and home-screen metadata so it installs as a standalone app. Routines can be shared: the client encodes the routine (name, exercise ids for standard movements, exercise names, sets, reps, an optional rep ceiling, rest) as a base64url code carried in /routines?import=code; importing decodes it locally, maps exercises to the reader's catalog, reports unmatched names, and opens the editor as a new unsaved routine. The rep ceiling is optional in the encoding, so codes produced before rep ranges existed still import. No server endpoint or table is involved and nothing private is included.

## Source format
Code contains no comments or formatting whitespace; required language whitespace and text spaces remain. Markdown remains readable. Framework-generated and dependency files are excluded. A formatting check validates owned source. The repository normalises line endings to LF through .gitattributes so the single-line rule survives checkouts on Windows.

## Hosting and external prerequisites
The requested Next.js and Supabase stack takes precedence over the Sites starter's alternative runtime/database defaults. Sites registration is retained in .openai/hosting.json. A Cloudflare-compatible adapter is required for private Sites deployment. Conventional Node hosting can run the same Next.js app.
Separate development and production Supabase projects, public project URL/key, migrations, authentication URLs, and email delivery setup are required. Credentials are never committed.

## References
- [Next.js installation](https://nextjs.org/docs/app/getting-started/installation)
- [Supabase server-side authentication](https://supabase.com/docs/guides/auth/server-side)
- [Supabase row-level security](https://supabase.com/docs/guides/database/postgres/row-level-security)
