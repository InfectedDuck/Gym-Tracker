# Forge development plan

## Confirmed decisions
Mobile-first responsive; Blackplate design system (near-black neutrals, one ember accent, Barlow Condensed numerals, icon rail, Daylight light variant); set-by-set logging; dedicated chart/history pages; private email/password accounts; routines, rest timer, and personal records; strength exercises in kg/lb; Supabase PostgreSQL; code without comments or formatting whitespace.

## Implementation status
- [x] Save the architecture and development plan.
- [x] Build the Next.js foundation, shared design system, desktop sidebar, and phone navigation.
- [x] Implement SQL migrations, profile creation, ownership policies, transactional saves, and account screens.
- [x] Seed 83 standard exercises and implement search, filters, custom creation, editing, and archival.
- [x] Implement daily sessions, multiple workouts per day, exercise ordering, set logging, copying, edits, deletion, and retry feedback.
- [x] Implement history pagination, charts, kg/lb display, previous-session and weekly comparisons, and personal records.
- [x] Implement independent reusable routines and a rest timer that survives refresh.
- [x] Pass strict TypeScript checking and source-format checks.
- [x] Pass 20 calculation, validation, and PostgreSQL integration checks.
- [x] Pass three browser workflows against the isolated PostgreSQL-backed test harness.
- [x] Verify desktop and phone screenshots, navigation, keyboard focus, and viewport overflow.
- [x] Document environment setup, email templates, release checks, and backup/restore procedures.
- [x] Pass the final Next.js production build after the verification fixes.
- [x] Add overload targets, personal-best detection, the weekly muscle summary, install metadata and reconnect retry with unit and browser coverage (2026-09-08).
- [x] Add routine share links and import with unit and browser coverage (2026-09-08).
- [x] Reconcile the interface against the published Blackplate design canvas, add routine set and rep schemes with rep ranges, and replace typed quick entry with a tap-driven builder (2026-09-08).
- [x] Place the project under version control with an LF normalisation policy and an .env.example template (2026-09-08).
- [ ] Configure development Supabase and verify real account confirmation, recovery, and live API integration.
- [ ] Configure production Supabase, email delivery, and hosting.
- [ ] Prepare and verify a Cloudflare-compatible build adapter if using Sites hosting, then deploy the application privately.

## Verification evidence
The database checks use a fresh embedded PostgreSQL engine with the actual schema, seeds, triggers, RPCs, and RLS policies. They prove owner isolation, protection of standard exercises, idempotent saves, stale revision rejection, transactional rollback, bodyweight constraints, routine isolation, and recalculation after edits/deletes.

Browser tests load the real UI components and validators against a test-only PostgreSQL HTTP adapter. They cover:
1. Entering and saving weights/repetitions, reloading persisted sets, retaining entries after a failed request, retrying, and completing a session.
2. Creating a custom exercise, saving and starting a routine, opening charts, selecting metrics, and changing units without rewriting weights.
3. Desktop/mobile overflow, phone navigation, keyboard focus, and timer restoration.

These tests do not replace live Supabase Auth/PostgREST/Next.js route verification. The separate live test is ready and requires a configured test project and two verified accounts. Test fixture data is isolated under tests and is not shipped as application data.

## Decisions recorded during implementation
- Keep the explicitly requested real Next.js runtime and Supabase database.
- Use transactional whole-workout and whole-template saves with revisions and stable mutation IDs.
- Compare sessions on the same day by start timestamp, then ID.
- Preserve exercise tracking type after creation to keep historical meaning stable.
- Use a dedicated setup screen when Supabase configuration is absent; do not substitute browser storage for saved workout data.
- Use native dialogs with unique accessible titles.
- Keep test-only navigation and HTTP adapters outside the application.
- Keep the Sites registration for future private deployment; registration does not mean publication.

## Product roadmap (decided 2026-09-08)
The owner asked what would make Forge succeed beyond a personal tool. A general gym forum was considered and deferred: it has a cold-start problem, an ongoing moderation cost, and it conflicts with the private-by-default promise. Community features are sequenced after the logging experience and data-backed sharing.

### Phase 1 — logging speed and return visits (in progress)
- [x] Progressive overload targets: each exercise card shows the previous session, the all-time best, and a next target computed by double progression (lib/progress.ts suggestNext); one tap fills the unlogged working sets.
- [x] Personal-best detection: logging a set that beats the all-time best shows a personal-best toast and marks the set with a trophy. Bests are fetched per exercise from the history endpoint, excluding the current workout.
- [x] Weekly muscle-group summary on Progress: completed working sets per muscle group, this week versus last week.
- [x] Installable app: web manifest, theme colour, favicon, Apple home-screen metadata. The proxy skips the manifest route.
- [x] Connectivity: an offline status line on Today, and a failed save retries automatically when the connection returns. Drafts stay in memory only, per the no-browser-storage rule.
- [x] Visual redesign implemented from the Blackplate direction in docs/DESIGN_BRIEF.md (2026-09-08): tokenised palette with a light variant, self-hosted Barlow Condensed and Inter, 64px desktop icon rail replacing the sidebar and top bar, single content column, weekly numbers as one line instead of stat cards, week strip promoted into the session header as the date control, ember progress rule under the session title, redesigned set row with a tap-to-cycle set type and unit-aware weight and repetition steppers, full-height log control, docked rest timer that no longer covers the log column, and noun page titles with the slogan copy removed.
- [x] Reconciled against the Blackplate design canvas (2026-09-08). The canvas was read by unpacking the exported bundle rather than through design-system authorization. Applied from it: hairline rather than edge borders on plate cells, 52px set rows with 22px index numerals, the record mark moved off the log button onto the value, grey warm-ups in place of yellow, ink-on-ground selection across segmented controls, chips, day cells and session tabs, the full-measure ember rule, shadow removal on dialogs, toasts and the rest timer, the canvas button and input states, and the struck-bar engraved brand mark. Deliberately kept over the canvas: the Daylight light variant, the 1180/760/480 breakpoints, visible focus rings, reduced-motion support, and borderless toolbar icon buttons.
- [ ] Weekly summary email or Monday recap card (sets per muscle, volume trend, PRs).
- [ ] Apply 202609080005_rep_ranges.sql to the development and production Supabase projects; the harness runs it but no hosted project has it yet.
- [ ] Cover the quick entry builder with unit and browser tests. It is now the default alternative to guided logging and has neither; tests/quick.test.ts only covers the secondary text parser.
- [ ] Surface routine rep ranges as targets during guided logging. Instantiating a routine currently seeds only the bottom of the range, so an 8-12 routine shows 8 with no sign of the ceiling.

### Phase 2 — acquisition without a community
- [x] Routine sharing: every routine has a share link (/routines?import=code) that opens the editor prefilled in any account; standard exercises map by id, custom ones by name, and unmatched names are reported (lib/share.ts).
- [ ] Preserve the import link through login and sign-up (a next parameter on /login) so a new user lands on the shared routine.
- [ ] Program library seeded by the owner (5/3/1, PPL, GZCLP, Starting Strength) with one-tap import. lib/schemes.ts and rep ranges now provide the set and rep vocabulary these programs need, so this is largely seed data carried by the existing share encoder.
- [ ] Share cards: an image of a finished session or a new PR for social stories.
- [ ] CSV import from Strong and Hevy to remove switching cost.

### Phase 3 — social layer, only with active users
- [ ] Optional public profile and a friends feed of completed workouts, behind an explicit public/private toggle and new RLS policies.
- [ ] A members-only board only if Forge targets a specific physical gym; otherwise no forum.

### Deferred with reasons
- Service worker and offline shell: private responses must never be cached; revisit after the redesign with a static-assets-only worker.
- Persisting unsaved drafts in browser storage: contradicts the current product rule; needs an explicit decision.
- Push notifications for rest timers: valuable but needs the PWA install first.

## Remaining external prerequisites
.env.example records the three required variables; real values are supplied locally in .env or .env.local and are never committed. Set up the development and production projects following SETUP.md. Real account email delivery, live application route isolation, and a deployed URL are not yet verified.

## Implementation discipline
The project is a git repository; .gitattributes normalises line endings to LF so the single-line source rule survives Windows checkouts, and .gitignore keeps every .env variant except .env.example out of history. Consult ARCHITECTURE.md before changing a subsystem. Update these documents as behavior changes. Mark external milestones only when verified. Keep source compact and comment-free; Markdown remains readable.

## Out of scope
Social features, cardio, body-weight measurements, goals, photos, and offline synchronization.
