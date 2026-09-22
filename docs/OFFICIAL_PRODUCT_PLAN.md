# Official product plan

Goal: keep the minimalist Log + Exercises + Calendar loop flawless, while closing the gaps that block an official release. Private by default. No new tabs.

## Where we are

- App has never stored a real set for a real person. `test:live` has never run. No hosted Supabase project is migrated past `202609070004_entry_mode.sql`.
- Live route is `SimpleLog`. `today.tsx` deleted. `quick.tsx` UI orphaned but tested and unbundled.
- Tabs are now 3: Log, Exercises, Calendar. Routines and Progress still exist by URL.
- Volume removed, TrendChart is bars, calendar can create/start routines.

## Phase 0 — Ship fundamentals

- [ ] Apply all 11 migrations in filename order on dev Supabase, then production. Includes `202609080011_drop_volume.sql` which docs currently omit.
- [ ] Configure dev auth end to end: Site URL, `/auth/callback`, `/auth/confirm?token_hash`, 12-char passwords, SMTP. Run `test:live` with two dedicated accounts.
- [ ] Production project + hosting: separate Supabase project, email sender, origin/callbacks. Add Cloudflare Worker adapter or Node + HTTPS proxy. Add deploy job to CI (currently verify+e2e only).
- [x] Fix docs drift: `SETUP.md` migration list, `setup/page.tsx` two-files text, pending-migrations section.
- [x] Extend `health` from `{configured}` to `{configured, version, migrations}` so pending-migration breakage is detectable.

## Phase 1 — Reliability, no new features

- [x] Delete `components/today.tsx` (dead, nothing imported it). Shared-hook extraction deferred.
- [x] Decide `components/quick.tsx`: keep. `lib/quick.ts` parser stays tested; the UI is orphaned but costs zero bundle (nothing in `app/` imports it) and stays covered by `quick-entry.test.ts`. Re-wire only if a quick mode returns.
- [x] Add `check`/`allRows` coverage in `tests/database.test.ts` (error mapping 409/404/403/400/500, pagination, reject path). History-window cap deferred: it changes payload shape for Log/Calendar.
- [x] Fix `chartSeries` UTC vs `dateInZone` off-by-one by passing `today` (optional param, `TrendChart` passes timezone-aware today, regression test added). Full `lib/progress.ts` split deferred.
- [x] Clamp `dateSchema` to 1970-01-01..2100-01-01 and add min/max to date inputs.
- [ ] Keep `check:format`, `typecheck`, `test`, `test:db`, `test:e2e` green after each step.

## Phase 2 — Minimal official polish

- [ ] Onboarding: Routines empty state gets 4-6 hand-encoded share payloads. No new program-library system.
- [ ] PWA: add 192/512 PNG + maskable, apple-touch-icon, shortcuts. Flip `layout.tsx` robots off private, add `metadataBase`, canonical, OG. No service worker for private data.
- [ ] In-gym: mount `PlateHint` under qualifying set rows, add vibrate + beep on rest-timer zero.
- [ ] Portability: set-level CSV export only. No import, no restore. Backups stay Supabase PITR with rehearsed restore.
- [x] Error ID in `error.tsx` (`error.digest`, falls back to unknown). Aggregation/monitoring still needs hosted logs.

## Deferred

Warmup-generator, RPE/RIR, supersets, weekly-recap, CSV import, OAuth/MFA, i18n, public feed. Killed or rescoped in `BACKLOG.md`.

## Verification

`check:format`, `typecheck`, `test`, `test:db`, `test:e2e`, then live auth plus two weeks of real logging before any Phase 2 feature.
