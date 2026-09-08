# Setup, verification, and release

## Local application
1. Install Node.js 20.9 or newer and run npm install.
2. Create a development Supabase project.
3. Copy .env.example to .env.local.
4. Set NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY from the project's API settings. The publishable key is intended for clients; never substitute a service-role key.
5. Set NEXT_PUBLIC_APP_URL to the local origin you will consistently use, such as http://127.0.0.1:3000.
6. Apply every file in supabase/migrations in filename order in the Supabase SQL editor: 202609070001_schema.sql, 202609070002_exercises.sql, 202609070003_rest_timer.sql, 202609070004_entry_mode.sql, 202609080005_rep_ranges.sql, 202609080006_catalog.sql, 202609080007_plates.sql, 202609080008_guards.sql. Each is a complete migration; apply each once. See Pending migrations below for the ones that no hosted project has yet.
7. Start npm run dev and open http://127.0.0.1:3000.

The application shows a setup screen while the public Supabase configuration is absent. Workout persistence is exclusively PostgreSQL. Only the rest timer's deadline is kept in localStorage.

## Email authentication
Enable Supabase email/password authentication and email confirmation. Configure a minimum password length of 12 characters to match the registration interface.

Set the project's Site URL to the application origin. Add these redirect destinations:
- http://127.0.0.1:3000/auth/callback
- http://127.0.0.1:3000/auth/callback?next=/reset-password
- Your deployed origin's matching callback URLs.

For reliable links opened on another device, set the confirmation email link to:
{{ .SiteURL }}/auth/confirm?token_hash={{ .TokenHash }}&type=email

Set the recovery email link to:
{{ .SiteURL }}/auth/confirm?token_hash={{ .TokenHash }}&type=recovery

The default PKCE callback is also implemented. The token-hash templates avoid requiring the original browser's PKCE verifier when users open email on another device.

Configure your production SMTP sender in Supabase and verify signup, resend confirmation, and recovery delivery with real inboxes. Supabase's default development sender has delivery limits; it is not a production email service configuration.

## Automated verification
The same chain runs on GitHub Actions for every push and pull request; CI.md describes the jobs, the serialised browser harness and how to reproduce a failure locally.

- npm test: calculation/validation tests and PostgreSQL integration tests using PGlite.
- npm run test:db: PostgreSQL policies, triggers, RPC transactions, duplicate retries, conflicts, and ownership tests.
- npm run typecheck: strict TypeScript checking.
- npm run build: the real Next.js production build.
- npm run check:format: confirms owned source is single-line and comment-free.
- npm run test:e2e: browser workflows against a test-only UI harness backed by a fresh embedded PostgreSQL instance. It loads the real migrations, catalog, components, and validators. It exercises logging, reload persistence, failed-save retries, custom exercises, routines, history, unit preferences, timer restoration, keyboard focus, phone/desktop overflow, and the quick entry builder.
- npm run test:live: tests the actual Next.js application and real Supabase authentication when a dedicated test project and test accounts are configured.

The UI harness lives under tests, listens only on 127.0.0.1:3100, uses synthetic records, and is never included in the production application. Its HTTP adapter and test navigation helpers do not verify Supabase Auth, PostgREST, or Next.js route wiring. The PostgreSQL tests emulate auth.uid() and use actual PostgreSQL role policies. Real Supabase verification remains a separate release requirement.

Browser tests default to installed Chrome. Use PLAYWRIGHT_CHANNEL=msedge to choose Edge, or set the channel to an available Playwright browser in your environment. Browser screenshots and failure traces are placed in test-results.

For live checks, configure E2E_BASE_URL, E2E_EMAIL, E2E_PASSWORD, E2E_SECOND_EMAIL, E2E_SECOND_PASSWORD, NEXT_PUBLIC_SUPABASE_URL, and NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY in the process environment. Use two verified, dedicated test accounts. Never use production accounts. Live tests create and remove their own workout records and archive their custom exercise afterward.

## Pending migrations
No hosted Supabase project has been migrated past 202609070004_entry_mode.sql. The test harness and the database suite replay every file in the folder (sorted readdir since commit f3c7342), so every check passes locally while a hosted project still runs the old schema. Four migrations are pending as of 2026-09-08; apply them in this order, each once, as a single statement batch in the Supabase SQL editor:

1. 202609080005_rep_ranges.sql adds template_exercises.target_reps_max with a check that the ceiling is at least the target, and replaces save_template with a version that stores the ceiling. It must go first because the catalog file assumes the current function set and because every rep-range feature depends on the column.
2. 202609080006_catalog.sql inserts the 67 standard exercises with IDs 084 to 150 (on conflict do nothing, so it is safe to re-run) and rewrites the descriptions of the original 83 by name. It has no dependency on the first file but should follow it so the two hosted projects stay in filename order.
3. 202609080007_plates.sql adds profiles.bar_weight_kg (null or 0 to 50) and profiles.plates_kg (text up to 200 characters); null means the display unit's default bar and plate set. Add-column-if-not-exists, so it is safe to re-run.
4. 202609080008_guards.sql redefines save_workout and save_template on top of 0005: both reject a payload whose exercise array is absent or not an array instead of silently treating it as empty, and a routine that already references an archived custom exercise stays saveable. It must follow 0005 because it restates save_template with the rep ceiling.

Until 202609080005_rep_ranges.sql is applied:
- Routines load without a target_reps_max field, so every rep range and named scheme renders as a single number and the guided target never shows a ceiling. The client types the field as required, so any code path that reads it gets undefined rather than null.
- Saving a routine calls the old save_template, which ignores the ceiling, so every named scheme and every 8-12 style range silently collapses to its bottom number.
- Importing a share code that carries a rep ceiling saves without it.

Until 202609080006_catalog.sql is applied:
- The picker and library show 83 exercises with the old boilerplate descriptions. Searches that rely on aliases still work, but the 67 newer movements (planks, sumo and trap bar deadlifts, preacher curls, carries, the band and kettlebell movements) are absent.
- Starter programs or share codes that reference the newer seed IDs report those exercises as unmatched.

Until 202609080007_plates.sql is applied:
- Saving the plate fields on Settings fails, because PostgREST rejects the unknown columns; other preference saves still work because profile updates are partial.
- Bootstrap returns no bar weight or plate inventory, so the plate hint falls back to the unit defaults once it is mounted.

Until 202609080008_guards.sql is applied:
- A direct RPC call with the exercise array omitted deletes every child row of the targeted workout or routine, because jsonb_array_length of NULL never trips the size guard. The application never sends such a payload, but the publishable key makes the RPC reachable from a browser.
- A routine that references an archived custom exercise cannot be re-saved.

After applying all four, run npm run test:live against the project to confirm routines with ranges save and load. Any later migration joins this list until it is applied to both hosted projects. CI.md explains why continuous integration cannot catch this gap: the workflow runs against an embedded database that replays every migration file, never against a hosted project.

## Deployment
Use a separate production Supabase project. Apply the same migrations in order, configure its email sender, and set the production origin and callback URLs.

For conventional Node hosting:
1. Set the production public Supabase variables during the build and at runtime.
2. Run npm ci, npm test, npm run typecheck, and npm run build.
3. Start the server with npm run start behind an HTTPS reverse proxy. Bind the start command's hostname to the hosting platform's required interface if necessary.
4. Verify registration, confirmation, sign-in, logging/reload, account isolation, and recovery on the deployed origin.

Sites registration is retained in .openai/hosting.json. The source is a genuine Next.js application. Sites requires a Cloudflare-compatible Worker build adapter before deployment; the standard .next output cannot be uploaded directly. Do not deploy the test harness or a static export as a replacement for this authenticated application.

No production deployment is complete until the live Supabase project, email flows, hosting environment, and build adapter have been configured and verified.

## Database changes, backup, and restore
Use versioned SQL migrations. Once applied to a live project, never edit historical migrations; add a new migration. A migration that changes a PostgreSQL function must restate the whole function body, as 202609080005_rep_ranges.sql does for save_template, so the chain replays cleanly. The current migrations are unapplied project deliverables; the Pending migrations section tracks which ones a hosted project still lacks.

Before production changes, use Supabase's available backup/export facilities and record the backup timestamp. Rehearse restoration into an isolated Supabase project and verify account/profile mappings, row counts, foreign keys, RLS policies, and sample historical weights. Backup retention and point-in-time recovery availability depend on the project's configured plan.

Prefer additive changes and deploy code compatible with both old and new schema during rollout. A failed release should roll back application code; restore data only through the rehearsed recovery procedure.

## Monitoring
Monitor server error rates, failed save requests, authentication failures, database capacity, and email delivery. Route logs contain error codes rather than workout bodies or passwords. Investigate repeated save conflicts and errors. Keep development and production credentials and data separate.
