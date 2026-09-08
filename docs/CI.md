# Continuous integration

GitHub Actions runs the automated verification described in SETUP.md on every push, every pull request, and on demand through the Actions tab. The workflow lives in .github/workflows/ci.yml and needs no secrets: the production build and every test suite run without a Supabase project.

## Jobs

The workflow has two independent jobs that start in parallel on ubuntu-latest with Node.js 24, the same major version used for local development.

- verify (about 15 minutes at most): npm ci, then npm run check:format, npm run typecheck, npm test, and npm run build, in that order. The first failing step stops the job.
- e2e (about 20 minutes at most): npm ci, then npx playwright install --with-deps chromium, then npm run test:e2e. When a test fails, the traces and screenshots that Playwright writes to test-results are uploaded as the playwright-test-results artifact and kept for seven days.

Both jobs restore the npm cache keyed on package-lock.json. The Playwright browser is downloaded on every e2e run, which keeps the job free of cache invalidation surprises at the cost of a short download.

## Browser tests in CI

playwright.config.ts launches the installed Chrome by default (channel: "chrome") and honours the PLAYWRIGHT_CHANNEL environment variable. GitHub-hosted runners have no Chrome, so the e2e job sets PLAYWRIGHT_CHANNEL=chromium and installs the Playwright-managed Chromium together with its system libraries. Everything else, including the dark colour scheme, retained-on-failure traces, and the single-worker setting, comes from the shared configuration.

The Playwright webServer boots the test harness from tests/harness.ts on 127.0.0.1:3100 with reuseExistingServer disabled, so a second harness on the same host would fail to bind. The e2e job therefore declares the concurrency group e2e-harness-port-3100 with cancel-in-progress set to false. GitHub allows one running and one pending job per group across the whole repository, so browser tests from different branches and pull requests execute one after another instead of at the same time. If a third e2e job arrives while one is running and another is already waiting, GitHub cancels the older waiting job and shows it as superseded; re-run the cancelled job from the Actions tab when you still need its result.

## Superseded runs

At the workflow level, a newer commit on the same branch or pull request cancels the run that is still in progress, which keeps the e2e queue short. Pushes to the default branch are never cancelled, so every commit on main gets a complete result.

Because the workflow listens to push as well as pull_request, a branch that has an open pull request in this repository receives two runs per commit: one for the branch and one for the merge commit that GitHub builds for the pull request. Restrict the push trigger to the default branch in ci.yml if that duplication becomes a cost.

## Environment

The workflow sets NEXT_TELEMETRY_DISABLED=1 and relies on the CI=true variable that GitHub provides. No Supabase variables are configured. Every route that reads them is rendered dynamically and checks the configured() helper first, so npm run build succeeds without them and the unauthenticated pages redirect to /setup at runtime. The PostgreSQL tests use PGlite, an embedded database, so no database service is required.

npm run test:live is intentionally not part of the workflow. It needs a dedicated Supabase project and two verified test accounts, as described in SETUP.md, and should keep running from a developer machine or a separately configured environment with those secrets.

## Reproducing a failure locally

Run the same commands in the same order from a clean install:

```
npm ci
npm run check:format
npm run typecheck
npm test
npm run build
npx playwright install --with-deps chromium
PLAYWRIGHT_CHANNEL=chromium npm run test:e2e
```

On Windows PowerShell set the variable first with $env:PLAYWRIGHT_CHANNEL = "chromium". Omit the variable to use your installed Chrome, which is the default outside CI. Download the playwright-test-results artifact from the failed run and open a trace with npx playwright show-trace path/to/trace.zip.

## Maintenance

- Node.js version: update node-version in both jobs together. Next.js requires at least 20.9.
- Actions: actions/checkout, actions/setup-node, and actions/upload-artifact are pinned to their v7 major tags, which receive compatible updates automatically.
- Browser: change PLAYWRIGHT_CHANNEL and the browser name in the install step together, for example to msedge with npx playwright install --with-deps msedge.
- Port: if the harness moves off 3100, rename the concurrency group so the reason for serialising the job stays visible.
