# Forge visual redesign: "Forged Ledger" hybrid

Implementation plan for the coding agent. Presentation layer only. Read this whole file before editing anything.

## 0. Background

Forge has two designs:

- **Blackplate** (what is in the code today, see `docs/screenshots/*.png` and `docs/DESIGN_BRIEF.md`): near-black ground, ember orange accent, Barlow Condensed + Inter, every section in a bordered card, sets shown as chips, stat tiles, solid orange bar charts.
- **Ledger** (a mockup only, `docs/design/ledger-redesign.pdf` if present; never implemented): pale paper or graphite, a single blue accent with strict rules, Schibsted Grotesk + Azeret Mono, no boxes, sets in a ruled table, sentence summaries, thin charts.

The verdict from both people who use the app:

| | Blackplate | Ledger |
|---|---|---|
| First impression | Strong: energetic, looks like a training tool, clear blocks, satisfying orange | Weak: feels like a spreadsheet or bank admin page, blue is corporate, mono everywhere looks like a dev tool, too much empty canvas, grey hairline charts look unfinished |
| Over weeks of use | Worse: orange on everything so it means nothing, cards inside cards, chips don't read down a column, toolbar mixes control types, scattered buttons, uppercase eyebrow labels | Better: one primary action per screen, sets read like a timetable, consistent toolbar order, sticky save bar on phone, honest charts, readable copy |

**Goal:** keep Blackplate's character and first-glance energy, and Ledger's structure and rules. One sentence: *Ledger's skeleton, Blackplate's skin.*

## 1. What comes from where

**Keep from Blackplate**
1. Dark-first identity. Dark graphite is the hero theme, and screenshots and reviews use it first. Light still ships, and both follow `prefers-color-scheme`.
2. Ember orange as the brand accent, in place of Ledger's blue.
3. A celebratory record colour (gold) so a PR feels like an event.
4. One level of surface. Each exercise block and each major panel sits on a slightly raised `--surface` with no border. This gives the scannable "blocks" look people liked at first glance. It never nests.
5. Solid, confident bar charts (filled bars, not 6px slivers).
6. Big, heavy set numbers.

**Keep from Ledger**
1. The accent rule: colour only where it carries meaning (section 3).
2. The ledger table for sets (Set · kg · Reps · Done), right-aligned fixed numeric columns, hairlines between rows, edit and read modes share the anatomy.
3. Action order: one primary per screen, rightmost on desktop; secondary actions in one joined group to its left; quiet underlined links beside the thing they change. On phone the primary goes in a sticky bar above the tab bar, with the one progress number ("3/9 sets done").
4. Top bar navigation on desktop (Forge · Log · Exercises · History · Routines … timer · Settings · avatar), and a bottom tab bar with labels on phone. Progress lives under History, as in the mockup.
5. The joined date stepper (‹ · date picker · ›) with a "Today" mark.
6. Sentence summaries instead of stat tiles on History and Progress.
7. Chart rules: direct labels, no legends, at most two labelled reference lines, no grid clutter, and the in-progress week drawn as a hollow bar.
8. Empty states: one sentence plus the one fixing action. Log's empty state keeps the four-step "How a session goes" list.
9. Settings layout: section label column on the left, fields on the right, 2px rule opening each section.
10. Sentence case everywhere. No all-caps eyebrow labels. Small, rectangular status marks (PR, Draft, Saved 19:42, Today, Custom), never pills.
11. Accessibility targets: 44px desktop, 48px phone, 56px set inputs with 22px numerals; 2px `--fg` focus outline with offset; status never by colour alone.

**Changed in both**
- Typography: Schibsted Grotesk for words and for large figures. Azeret Mono only for tabular data (table cells, dates in tables, chart axes, column heads, the timer). This keeps the instrument feel without the dev-tool look.
- Density: Ledger's spacing is too airy for a training tool. Use the scale below, but cap the space above page titles at 48px desktop / 16px phone, and between sections at 40px.
- Desktop emptiness: fill it with useful context, not decoration (see the Log spec: "Previous" line per exercise, and a right-hand rail at ≥1280px).

## 2. Tokens

Implement in `app/globals.css` as CSS variables on `:root`. Use a `@media (prefers-color-scheme: light)` override, matching how the file does it today. Replace the Blackplate token names; update every usage.

### Dark (hero)
| Token | Value | Use |
|---|---|---|
| `--bg` | `#141517` | Page ground |
| `--surface` | `#1C1E21` | Exercise blocks, panels (one level, no border) |
| `--sunk` | `#0F1012` | Column-head band, quiet wells |
| `--field` | `#24272B` | Inputs, menus, popovers |
| `--fg` | `#EEEDE8` | Text, figures, heavy rules, done marks |
| `--fg-2` | `#B5B8B3` | Secondary text, inactive nav |
| `--fg-3` | `#9A9E9A` | Meta, captions, axes (check ≥4.5:1 on `--bg` and `--surface`) |
| `--rule` | `#2E3135` | Hairlines (decorative) |
| `--rule-strong` | `#6E7379` | Field and control borders (≥3:1) |
| `--accent` | `#FF6A2B` | Ember: primary action, where you are |
| `--on-accent` | `#1A0A03` | Text on ember |
| `--accent-wash` | `#3A1B0C` | Current set row, selected day |
| `--record` | `#FFD56A` | Gold: standing PR only |
| `--on-record` | `#1A1203` | Text on gold |
| `--danger` | `#FF7A6B` | Errors, destructive confirm |

### Light
| Token | Value |
|---|---|
| `--bg` | `#F5F4F0` |
| `--surface` | `#FFFFFF` |
| `--sunk` | `#ECEAE5` |
| `--field` | `#FFFFFF` |
| `--fg` | `#16181B` |
| `--fg-2` | `#4A4D52` |
| `--fg-3` | `#5F636A` |
| `--rule` | `#DDDAD3` |
| `--rule-strong` | `#7D8085` |
| `--accent` | `#B83A0C` |
| `--on-accent` | `#FFFFFF` |
| `--accent-wash` | `#F7E0D4` |
| `--record` | `#8A5A00` |
| `--on-record` | `#FFFFFF` |
| `--danger` | `#A32018` |

Verify every text pair with a contrast checker and fix values that miss: text ≥4.5:1 on `--bg`, `--surface` and `--accent-wash`, and control borders ≥3:1. On `--accent-wash`, metadata steps up from `--fg-3` to `--fg-2`.

### Type
Load both fonts with `next/font/google` in `app/layout.tsx`, replacing Barlow Condensed and Inter. Expose them as `--font-sans` and `--font-mono`.

| Token | Size/line · weight (desktop → phone) | Family | Use |
|---|---|---|---|
| display | 44/48 · 700 → 32/36 | Grotesk | Page title, one per screen, letter-spacing -0.02em |
| title | 28/34 · 650 → 22/28 | Grotesk | Workout and section titles |
| heading | 18/24 · 600 | Grotesk | Exercise names |
| body | 15/22 · 400 (500 in controls) | Grotesk | Text, buttons, fields |
| small | 13/18 · 400 | Grotesk | Meta, help text |
| figure-xl | 40/44 · 650 → 32/36 | Grotesk, `tabular-nums` | Key figures (Best, Trend), big summary numbers |
| figure-l | 22/28 · 600 | Grotesk, `tabular-nums` | Set inputs, logged sets |
| data | 14/20 · 400 | Mono | Table cells, dates in tables |
| axis | 12/16 · 400 | Mono | Column heads, chart axes, never caps |

If Schibsted Grotesk's `tabular-nums` does not align columns in practice, fall back to Azeret Mono 500 for figure-l only.

### Space, radius, rules, motion
- Spacing scale (4px base): 4, 8, 12, 16, 24, 32, 40, 48, 64. Nothing off-scale.
- Radius: 0 for layout and tables, 4px for controls, surfaces and marks. Avatar is the only circle.
- Rules: hairline 1px `--rule` between rows; 1px `--rule-strong` under column heads and on fields; 2px `--fg` opens a major section (not inside a surface block).
- Motion: 120ms ease-out, state changes only (set ticked, tab switched, menu opened, timer start). Zero under `prefers-reduced-motion`.

## 3. Colour rules (the core discipline)

**Ember (`--accent`) appears only for:**
1. The one primary action on the screen (filled button).
2. Where you are: active nav item (2px ember rule + `--fg` 600 text), selected day/range (2px ember rule + wash), the current set row (wash + ember set number + "Up next"), a running rest timer.

**Gold (`--record`) appears only for:**
3. A standing personal record: the PR mark (filled gold, `--on-record` text), the record figure, and its point on a chart. When a record is beaten, the old one returns to `--fg`.

**Never coloured:** headings, logo, links (underlined `--fg`), borders, decorative icons, hover states, ordinary chart series, counts, empty states, done checkmarks (done = `--fg` filled square with `--bg` check). Errors use `--danger` only.

**Thumb test:** cover each coloured element. If the screen still means the same thing, remove the colour.

## 4. Components

Build or restyle in `components/ui.tsx` and `components/shell.tsx` first; pages consume them.

- **Button** tiers: primary (ember fill, 600), secondary (`--rule-strong` outline, `--fg` text), quiet (underlined text), icon (outlined square, `aria-label` required), danger (quiet `--danger` text, always confirms). Height 44 / 48 phone.
- **ButtonGroup**: joined secondaries, shared borders, no gaps.
- **Field**: one shape for text, select, search and date. Label above (13/600), help text below (`--fg-3`), error swaps the border and message to `--danger`.
- **SetInput**: right-aligned figure-l, `inputmode="decimal"` for weight and `"numeric"` for reps; 56px tall on phone. The unit goes in the column head, not in each field.
- **Ledger** (set table): edit and read modes, columns Set · kg · Reps · Done, current row wash, done check, PR mark.
- **ExerciseBlock**: `--surface` background, 16/24 padding, no border, no coloured edge. Contains name (link, ↗), meta line, "Previous: 77.5 kg × 8 · 8 days ago" line with a quiet "Copy previous", then the Ledger.
- **Tabs/RangeTabs**: text over a hairline, selected gets the 2px ember rule.
- **DateStepper**: joined ‹ · calendar · › with a "Today" mark.
- **Mark**: small rectangular tags (PR gold, Draft hollow circle + text, Saved filled dot + time, Today outline, Custom outline).
- **Chart primitives**: bars (filled `--fg-3` at 60% of slot width, current-week hollow, record/point of interest in gold or `--fg`), line (1.5px `--fg-3`, 3px dots, record point gold with direct label), one-line mono tooltip on `--field`.
- **Nav**: desktop 64px top bar; phone bottom tab bar (64px, labels always shown) plus a compact header with logo, timer and avatar/Settings.
- **StickyActionBar** (phone): progress figure left, primary right, sits above the tab bar, respects `env(safe-area-inset-bottom)`.

## 5. Screens

Follow the Ledger mockup layouts, with these hybrid changes:

**Log, active draft**
- Header: display date + Today mark; DateStepper and "Calendar" link on the right.
- "Add a workout" row: routine select · [Repeat last workout | New workout] group · "Edit routines" link. On phone this moves below the workout as "Add another workout".
- Workout title, status mark (Draft · not saved), "3/9 sets done", then [Routine | Repeat] + **Save workout** (primary).
- Exercise blocks: 3-column grid at ≥1200px, 2 at ≥768px, 1 on phone; each is an ExerciseBlock on `--surface`.
- At ≥1280px, optional right rail (only if it fits without crowding): rest timer state and "Last time" for the current exercise. Skip it if it complicates the code; do not invent data.

**Log, saved**: read-mode ledgers; [Edit | Routine | Repeat] + **See progress →** primary; a one-line PR banner ("PR · Barbell bench press 80 kg × 8 is your new best, +2.5 kg on Sat 10 Oct") with a gold mark, no box.

**Log, empty day**: "Nothing logged yet", a one-sentence explanation, **Repeat last workout** primary, "or" routine select, New workout secondary, Edit routines link; "How a session goes" 01–04 list on the right (below on phone).

**History**: display title + one sentence summary with figures bold. Month calendar on `--surface` (logged = small `--fg` square, selected = wash + 2px ember rule + ember numerals, today = 1px ring). Day detail beside it (below on phone) as a ruled list with PR marks; "Add a workout on this day" select + Start primary.

**Progress** (under History): summary sentence, then a **key-figures strip**: three big figure-xl numbers (sessions/week, sets this week vs usual, PRs in 6 weeks) separated by vertical hairlines, no boxes. This is where Blackplate's punch comes back. Then "Sets per week" filled bar chart with a "usual week" reference line and a hollow current week, then "Muscle focus" bars with Week/Month tabs.

**Exercise detail**: back link, title, meta, description; key-figures strip (Best with gold PR, Last time, Trend); Progress line chart with range tabs and a metric select; Sessions table (mono data, Heaviest column, Change, PR mark). On phone the strip becomes label-left/figure-right rows.

**Exercises list**: title + "162 standard movements, 0 of your own", **Custom exercise** primary; search + two filter selects in one row; alphabetical ruled table with letter group heads (phone: name + meta line).

**Routines**: title, Import secondary + **Create routine** primary; "Your training week" seven-column strip (phone: rows) with today washed; saved routines as `--surface` blocks with a ruled exercise list, Start workout secondary, icon group (share, edit, delete in `--danger`).

**Settings**: two-column section layout as in the mockup; Save preferences primary; Your account section with Sign out secondary.

**Auth, 404, error, setup**: centred narrow column, 2px rule above the form, one primary. Error page shows a mono reference id.

## 6. Hard constraints

- **Do not change behaviour or data.** No edits to `lib/`, `app/api/`, `supabase/`, or `tests/harness.ts`; another session owns them and has uncommitted work there. If a layout needs data that isn't already passed to a component, stop and ask instead of adding it.
- **Read `AGENTS.md` first.** This Next.js version has breaking changes; read the relevant guide in `node_modules/next/dist/docs/` before touching `app/layout.tsx` or fonts.
- **Source format:** `npm run check:format` requires owned source files to be single-line and comment-free. Match the existing minified style; don't reformat whole files.
- Keep the hard constraints in `docs/DESIGN_BRIEF.md` §"Keep" (four destinations, exercise card anatomy, Settings reachable on every breakpoint).
- Don't revert or overwrite changes you didn't make. Check `git status` before starting and before each commit.
- Commits: short plain subjects, one per phase. No AI or assistant mentions, no co-author trailers.

## 7. Phases

Work in this order. After each phase run the checks in section 8 and take screenshots.

1. **Baseline.** Run `npm run test:server` (harness at http://127.0.0.1:3100). Screenshot every route at 390px and 1440px, dark and light, into `docs/screenshots/before-hybrid/`.
2. **Tokens and fonts.** Rewrite tokens in `app/globals.css`, swap fonts in `app/layout.tsx`, and update `themeColor` in the viewport to the new `--bg` values. Map old token names to new ones throughout.
3. **Shell and primitives.** `components/shell.tsx` (top bar, tab bar, phone header, timer) and `components/ui.tsx` (Button, ButtonGroup, Field, Mark, Tabs, DateStepper, StickyActionBar).
4. **Log.** `components/simple-log.tsx`, `components/quick.tsx`, `components/plates.tsx` styling only: ExerciseBlock, Ledger, the three states (draft, saved, empty), sticky phone bar.
5. **History and Progress.** `components/history.tsx`, the progress page, and chart primitives.
6. **Exercises, Routines, Settings.** `components/library.tsx`, `components/journal.tsx` (if it renders exercise detail), `components/routines.tsx`, `components/settings.tsx`.
7. **Auth, error, not-found, setup.** `components/auth-form.tsx`, `app/error.tsx`, `app/not-found.tsx`, `app/setup`, `app/login`, `app/reset-password`.
8. **Polish and docs.** Run the thumb test on every screen, then save final screenshots to `docs/screenshots/` (replacing the old ones). Update `docs/DESIGN_BRIEF.md` with a status line saying "Forged Ledger" (this file) is now the source of truth.

## 8. Checks after every phase

- `npm run typecheck`, `npm test`, `npm run check:format`, `npm run build`.
- `npm run test:e2e`. If a selector breaks because markup changed, update the selector and keep what the test asserts. Never delete an assertion to make a test pass.
- The a11y spec (`tests/browser/a11y.spec.ts`) must stay green.
- Visual check at 390px and 1440px in both themes: no horizontal scroll, no ember outside the section 3 places, no gold except standing PRs, no nested surfaces, no all-caps labels, one primary per screen.

## 9. Done means

- Every screen in section 5 matches its spec in both themes and both widths.
- All checks in section 8 pass on a clean run.
- Before/after screenshots exist for every route.
- A short summary lists any deviations from this plan and why.
