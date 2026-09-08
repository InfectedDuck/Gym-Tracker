# Forge
Private strength training with Next.js, React, Tailwind CSS, and Supabase PostgreSQL.

## Run locally
Install dependencies with npm install. Copy .env.example to .env.local and enter your Supabase project URL and publishable key. Apply the SQL migrations in supabase/migrations in filename order using the Supabase SQL editor. Run npm run dev and open http://127.0.0.1:3000.

The app displays a configuration screen until Supabase is configured. It never stores workout history in browser storage.

## Account configuration
Enable email/password authentication and email confirmation. Set Site URL to your application origin and allow /auth/callback and /auth/confirm. Set the confirmation email link to the application's /auth/confirm endpoint with token_hash and type query parameters as described in docs/SETUP.md. Configure production SMTP and test recovery delivery.

## Project references
- [Architecture](docs/ARCHITECTURE.md)
- [Development plan](docs/DEVELOPMENT_PLAN.md)
- [Setup and release](docs/SETUP.md)

## Verification
npm test runs calculations, validation, and PostgreSQL policy/transaction tests with an embedded PostgreSQL engine. npm run typecheck and npm run build validate production source. npm run test:e2e runs the real UI against an isolated PostgreSQL test harness. npm run test:live verifies real authentication and application APIs with a configured Supabase test project and two dedicated test accounts. See docs/SETUP.md.

Generated application source is intentionally compact and comment-free per the project's coding constraint.
