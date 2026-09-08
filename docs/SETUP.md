# Setup, verification, and release

## Local application
1. Install Node.js 20.9 or newer and run npm install.
2. Create a development Supabase project.
3. Copy .env.example to .env.local.
4. Set NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY from the project's API settings. The publishable key is intended for clients; never substitute a service-role key.
5. Set NEXT_PUBLIC_APP_URL to the local origin you will consistently use, such as http://127.0.0.1:3000.
6. Apply 202609070001_schema.sql, then 202609070002_exercises.sql from supabase/migrations in the Supabase SQL editor. Each is a complete migration; apply each once.
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
- npm test: calculation/validation tests and PostgreSQL integration tests using PGlite.
- npm run test:db: PostgreSQL policies, triggers, RPC transactions, duplicate retries, conflicts, and ownership tests.
- npm run typecheck: strict TypeScript checking.
- npm run build: the real Next.js production build.
- npm run check:format: confirms owned source is single-line and comment-free.
- npm run test:e2e: browser workflows against a test-only UI harness backed by a fresh embedded PostgreSQL instance. It loads the real migrations, catalog, components, and validators. It exercises logging, reload persistence, failed-save retries, custom exercises, routines, history, unit preferences, timer restoration, keyboard focus, and phone/desktop overflow.
- npm run test:live: tests the actual Next.js application and real Supabase authentication when a dedicated test project and test accounts are configured.

The UI harness lives under tests, listens only on 127.0.0.1:3100, uses synthetic records, and is never included in the production application. Its HTTP adapter and test navigation helpers do not verify Supabase Auth, PostgREST, or Next.js route wiring. The PostgreSQL tests emulate auth.uid() and use actual PostgreSQL role policies. Real Supabase verification remains a separate release requirement.

Browser tests default to installed Chrome. Use PLAYWRIGHT_CHANNEL=msedge to choose Edge, or set the channel to an available Playwright browser in your environment. Browser screenshots and failure traces are placed in test-results.

For live checks, configure E2E_BASE_URL, E2E_EMAIL, E2E_PASSWORD, E2E_SECOND_EMAIL, E2E_SECOND_PASSWORD, NEXT_PUBLIC_SUPABASE_URL, and NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY in the process environment. Use two verified, dedicated test accounts. Never use production accounts. Live tests create and remove their own workout records and archive their custom exercise afterward.

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
Use versioned SQL migrations. Once applied to a live project, never edit historical migrations; add a new migration. The current migrations are initial, unapplied project deliverables.

Before production changes, use Supabase's available backup/export facilities and record the backup timestamp. Rehearse restoration into an isolated Supabase project and verify account/profile mappings, row counts, foreign keys, RLS policies, and sample historical weights. Backup retention and point-in-time recovery availability depend on the project's configured plan.

Prefer additive changes and deploy code compatible with both old and new schema during rollout. A failed release should roll back application code; restore data only through the rehearsed recovery procedure.

## Monitoring
Monitor server error rates, failed save requests, authentication failures, database capacity, and email delivery. Route logs contain error codes rather than workout bodies or passwords. Investigate repeated save conflicts and errors. Keep development and production credentials and data separate.
