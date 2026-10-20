# Admissions Showcase Plan — Make Forge Beautiful on First Sight

## 1. Goal

Make `Forge` (Next.js + React + Tailwind + Supabase workout tracker) look **undeniably polished in 30 seconds** to a non-technical admissions officer.

Success = someone opening the GitHub repo sees:
1. A striking hero screenshot / GIF at the top of README
2. A clean feature grid with 6-7 beautiful screenshots
3. Clear “what it is / tech / what I built” story
4. Every screenshot looks intentional, dense with realistic data, no empty states, no login wall

## 2. Non-goals (explicit)

Per owner request — do NOT do:
- No launch, no production deploy, no custom domain, no SEO, no analytics
- No popularization: no social, sharing, onboarding funnel, marketing site
- No new workout features (no RPE, supersets, programs, export, notifications)
- No auth hardening / SMTP / hosted Supabase work beyond what screenshots need
- No refactor of save logic, migrations, or test harness unless it blocks screenshots

We optimize for **static visual impression + README**, not for users.

## 3. Current state (audited 2026-10-19)

What we have:
- App works: `app/(dashboard)/` with Today / Exercises / Progress / Routines / Settings, Blackplate design system (`app/globals.css`: near-black `#0E0F10`, bone ink, ember `#FF6A2B`, Barlow Condensed numerals + Inter body)
- `README.md` already has 7 screenshots from `npm run test:server` seeded demo (`docs/screenshots/*.png`, 18/10/2026): today, exercises, exercise-detail, progress, history, routines, settings
- README Features list is accurate but dry, no hero, no badges, no tech stack, no GIF, no “why this is impressive” framing
- `docs/ARCHITECTURE.md`, `DESIGN_BRIEF.md`, `BACKLOG.md` are strong engineering proof but invisible to admissions

What looks unimpressive right now:
1. No public landing — first paint is login/setup screen, terrible for screenshots
2. Screenshots are raw app screens, inconsistent density, no captions that sell, no mobile + dark/light pairing
3. No visual hero: no logo lockup, no tagline, no color story
4. README buries the lead: tech and engineering rigor (RLS, transactional saves, PGlite tests, CI) at bottom
5. Small UI roughness that photographs badly: empty states, thin charts, default favicon/title, cramped mobile spacing

## 4. Target audience

Admissions officer: skims for 1-2 min, non-technical, cares about:
- Does it look real and finished?
- Did this student design + build + test something hard?
- Can I understand it without running it?

So: big visuals first, plain-English features second, tech proof third.

## 5. Plan — 5 phases, in order

### Phase A — Demo data that photographs well (1-2 hrs)
1. Freeze one demo persona in `tests/browser/entry.tsx` / harness: “Alex Morgan, kg, Asia/Shanghai” with 12 weeks of history, 8-10 named routines, 3 recent PRs.
2. Ensure every screenshot route has data: Today 3/9 sets logged, Progress with 12-week bars + muscle balance, History with dots + completed session, Exercise-detail with 4wk/12wk chart, Routines with 5-exercise cards, Exercises with 162 results.
3. Add `npm run showcase` alias: boots `test:server` on fixed port + fixed date (Mon 7 Sep 2026 from DESIGN_BRIEF) so screenshots are reproducible.
4. Acceptance: all 7 routes render with zero “No workout” / “No sets yet” states.

### Phase B — Screenshot-grade UI polish only (3-5 hrs, highest ROI)
No features, only CSS / copy tweaks in `components/*`, `app/globals.css`:
1. Header/title polish: page titles 28px, consistent overline labels, ember progress rule under session title.
2. Charts: thicker lines, visible dots, gaps not interpolation, “Completed working sets only” caption, record flame `Flame` icon.
3. Cards/lists: hairline rules not nested boxes, 8px cards, tabular numerals everywhere, 44px touch targets hold at 390px.
4. Daylight variant check: `prefers-color-scheme:light` tokens must look clean — admissions often use light mode.
5. Favicon + title: keep `public/icon.svg` engraved F, `<title>Forge · Workout Tracker</title>`, og description “Private strength training — log sets, track PRs, see progress.”
6. Acceptance: side-by-side dark + light Today at 1440px and 390px both look intentional, no overflow, no tiny 12px metadata.

### Phase C — Public hero for README (2-3 hrs)
Admissions should never see a login wall first:
1. Add lightweight public hero section on `/login` or `/` when logged out: engraved mark, headline “Your training, with a little more intention.”, 3 bullets, primary “View demo screenshots” anchor + secondary “Run locally”.
2. Keep auth card unchanged underneath — no auth logic change.
3. Capture 1 hero image: 1440px Today mid-session cropped to header + 2 exercise cards + session bar. This becomes README top.
4. Optional 15-sec GIF: log a set → ember check → timer starts. Use Playwright `test:e2e` recording, save as `docs/screenshots/demo.gif` (<5MB).
5. Acceptance: hero screenshot alone explains the app without reading text.

### Phase D — README rewrite to impress (2 hrs)
Rewrite `README.md` in this order:
1. Title lockup: `# Forge — Private Strength Training Tracker` + 1-line hook + badges (Next.js 16, React 19, TypeScript, Tailwind v4, Supabase, Playwright, CI passing).
2. Hero image + `> Private by default. Log set-by-set or in one tap. Track PRs, volume, streaks, muscle balance.`
3. Feature grid with screenshots (2-col table, keep existing 7 but recaption for non-technical reader):
   - Guided logging | Quick entry `60kg x 8` | 162 exercises + typo-tolerant search | Progress dashboard | History calendar | Routines + weekday schedule
4. “Under the hood (for reviewers)” — 6 bullets: App Router + SSR, Supabase Auth + RLS, transactional `save_workout` with revision + mutation-id retry, Epley e1RM, PGlite embedded Postgres tests, GitHub Actions CI.
5. “Run the demo in 2 minutes” — `npm install`, `npm run test:server`, open `127.0.0.1:3000`, no Supabase needed. Link full setup to `docs/SETUP.md`.
6. “What I learned” — 3-4 lines: data modeling, RLS, idempotent saves, chart edge cases. Admissions loves reflection.
7. Keep `Verification` + `Project references` collapsed at bottom.
8. Acceptance: README renders fully on github.com without cloning, all image paths `docs/screenshots/*.png` resolve, no broken links.

### Phase E — Repo hygiene for first impression (1 hr)
1. `docs/screenshots/` keep only final 7 + hero + optional gif. Delete blurry/duplicates. Compress PNGs.
2. Root: ensure `LICENSE`, `.env.example`, `public/icon.svg`, `CI` badge green.
3. Pin `docs/ARCHITECTURE.md` + `docs/DESIGN_BRIEF.md` links from README as “Design system + architecture”.
4. Final `npm run check:format && npm run typecheck && npm run build && npm test` passes before commit.
5. Commit as `Polish showcase: hero, screenshots, README for review`.

## 6. File map

- Edit: `README.md`, `app/globals.css`, `app/login/*`, `components/shell.tsx`, `components/today.tsx`, `components/history.tsx`, `tests/browser/entry.tsx`
- Add: `docs/screenshots/hero.png`, `docs/screenshots/demo.gif` (optional), this plan file
- Do not touch: `supabase/migrations/*`, `lib/validation.ts`, `proxy.ts`, auth callbacks, save functions

## 7. What to do next (when you approve)

1. Run `npm run test:server` and re-capture 7 screenshots at 1440 + 390
2. Apply Phase B CSS tweaks
3. Rewrite README per Phase D template
4. Verify + commit

Say “build it” and I will start with Phase A + B.
