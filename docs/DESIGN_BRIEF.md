# Design brief for the Forge redesign

> Status: the "Blackplate" direction in section 3 was implemented in the application on 2026-09-08. The palette, type, layout, and component decisions below are the source of truth for app/globals.css. The alternate directions at the end remain unbuilt.

Everything below the line is the brief. Section 3 is the recommended direction. Two alternate directions are at the end of this file; to use one, replace section 3 with it and leave everything else unchanged.

---

# Redesign Forge, a private strength-training tracker

## 1. Product and user

Forge is a private, single-user strength-training log. One person uses it several times a week: on a phone in the gym to log sets between efforts, and on a desktop afterwards to review progress. There are no social features, no cardio, no body-weight tracking, no goals, no photos. It is a precise instrument, not a fitness-brand marketing surface.

The name matters: **forge** means iron, heat, and repetition. The current design ignores that.

## 2. What exists today and why it is being replaced

Today's build is a dark charcoal dashboard with an emerald accent, Segoe UI, a 224px fixed sidebar, an 80px top bar, three stat cards above the workout, a right-hand aside with promos, and a slogan footer. It is competently built but generic: every surface is a slightly different green-grey bordered box, the emerald is on everything so it signals nothing, and the copy talks like a landing page. The set you are about to log is the smallest text on the page and sits below the fold on a phone.

**Move away from:**
- The green-on-green palette. Emerald tints for surfaces, borders, chips, badges, icons, selected states, done states, and decoration all at once.
- Bordered-box-inside-bordered-box as the only containment. Stat card row as a reflex. Promo panel ("Train with a plan") inside a daily tool.
- Tracked-uppercase eyebrow labels on every page, used for five different jobs.
- Motivational-poster copy: "SHOW UP. BUILD ON IT.", "Make the next rep count.", "One session at a time.", "Your training journal." headlines with trailing periods.
- A system font with no chosen typeface, tiny 12px grey metadata where a lifter reads between sets, numeric inputs at body-text size.
- Chrome before content: top bar with a static crumb, giant page headline, three stats, then the workout.
- A floating rest timer card that covers the log column on mobile.
- Two settings entry points on desktop, a marketing footer on app screens.

**Keep (these are hard constraints, not preferences):**
- Exactly four primary destinations: Today, Exercises, Progress, Routines. Settings reachable on every breakpoint. Sidebar or rail on desktop, bottom navigation on mobile.
- The exercise card anatomy: number, name linking to history, muscle · equipment meta, "Previous: 77.5 kg · 9 days ago" line with Copy previous, a set table with Set / Type / Weight / Reps / Log, Add set, per-exercise rest seconds.
- One tap on the log control both saves the set and starts the rest timer. Add set clones the previous row.
- Text contrast at or above 4.5:1 everywhere, a single visible focus ring, 44px minimum touch targets, 48px nav rows, reduced-motion respected, native dialog elements.
- Terse verb-first action copy: New workout, Finish workout, Copy previous, Add set, Add exercise, Save routine.
- Tabular numerals on every number.
- Private, quiet, no gamification, no streak badges, no confetti.

## 3. Design direction: "Blackplate"

**Mood.** A machined steel instrument. Think engraved plate labels on industrial equipment, a stencil-numbered weight plate, a precision scale readout, a well-set field notebook. Heavy, calm, exact. Not neon, not gamer RGB, not glassmorphism, not gradients.

**Palette (dark, true neutrals, one hot accent).**
- Ground `#0E0F10`. Surface `#161819`. Raised surface `#1E2123`. Inset (inputs) `#0A0B0C`.
- Hairline border `#2A2E31`. Strong border (interactive boundaries only, 3:1) `#4A5056`.
- Ink `#F2F1EC` (bone white). Secondary text `#B5B8B3`. Tertiary text `#7E827E`, never below 13px.
- Ember accent `#FF6A2B` with ink-on-accent `#1A0A03`. Ember has exactly two jobs: the primary action and "done". Nothing else is orange.
- Record highlight `#FFD56A` (white-hot). Used only when a set or session beats an all-time best.
- Danger `#FF5C5C` on `#2A1414`. Warning (unsaved) `#E8B84A`.
- Focus ring: 2px `#F2F1EC` with 2px offset.
- Provide a "Daylight" variant for bright gyms with the same tokens remapped: ground `#F4F2ED`, surface `#FFFFFF`, ink `#111214`, ember stays `#E8531A`. Show it on one artboard (Today, mobile).

**Typography.**
- Display and numerals: **Barlow Condensed** (Google Fonts), weights 500 to 700, `font-variant-numeric: tabular-nums`. Every weight, rep count, timer, and stat is set in this face.
- Body and UI: **Inter** (Google Fonts), weights 400, 500, 600.
- Six-step scale only: 13, 14, 16, 20, 28, 40. Set inputs render at 22px condensed 600. The rest timer countdown at 40px. Page titles at 28px. Nothing decorative is larger than the data.
- One overline style: 12px Inter 600, letter-spacing 0.08em, uppercase, tertiary color. Used for column headers and panel captions only. Never for slogans.
- Headings are nouns without trailing periods: Today, Exercises, Progress, Routines, Settings.

**Layout principles.**
- Desktop: a 64px icon rail on the left (mark, four icons with labels on hover, avatar at the bottom that opens Settings). No top bar. One content column, max 880px, left-aligned. No right aside.
- Mobile: bottom navigation with four tabs. When a workout is open, a docked **session bar** replaces the nav: rest timer countdown, sets logged, Finish. The nav returns behind a small menu affordance.
- The first unlogged set is visible without scrolling on a 390px phone.
- Containment by hairline rules and spacing, not nested boxes. At most three surface levels: ground, one card, one inset.
- Radii: 4px controls, 8px cards, 12px dialogs. No drop shadows except the dialog backdrop.
- Motion: 120ms opacity and transform only. Done state snaps, no bounce.

**Signature elements.**
- **Plate numerals.** Set weight and reps are large condensed digits in inset cells, like a scale readout.
- **Plate steppers.** Each weight cell has ±2.5 kg (±5 lb) buttons on long-press or tap, reps ±1. A tap on the cell opens the keyboard for direct entry.
- **Type toggle in the index.** The set index cell shows `1`, `2`, `3` for working sets and `W` for warm-up. Tap to cycle. No select dropdown in the row.
- **Log button** spans the full row height, 52px, empty square when idle, solid ember with a check when done. The row's numerals turn ember when done.
- **Ember rule.** A 2px ember line under the session title shows progress: it fills as sets are logged, 3/9 becomes one third.
- **Record mark.** A small white-hot flame glyph (lucide `Flame`) beside any value that is an all-time best.
- **Engraved mark.** The Forge logo becomes a heavy slab "F" with a struck horizontal bar, single color, works at 24px.

## 4. Screens to design

Use this sample data everywhere: profile "Alex Morgan", unit kg, timezone Asia/Shanghai, today Monday 7 September 2026. Workout "Upper body · Push" with Barbell bench press 3 × 8 @ 80 kg (previous 77.5 kg on 30 Aug, 9 days ago), Dumbbell bench press 3 × 8 @ 26 kg (previous 25 kg), Overhead press 3 × 8 @ 40 kg (previous 39 kg). 3 of 9 sets logged. This week: 2 sessions, 4 working sets, 1,900 kg volume, +1 vs last week. Recent bests: Barbell bench press 82.5 kg, Dumbbell bench press 26 kg, Overhead press 40 kg.

### 4.1 Today (home, /)
- Compact header: editable session title as the page's real heading (placeholder "Strength session"), date "Mon 7 Sep" with previous and next day chevrons and a calendar icon opening the native date input, and a 7-cell week strip Monday to Sunday with a dot on days that have a completed working set. Selected day and today are visibly different (fill vs ring).
- Status line with a state icon, not a permanent green dot: "Saved · 3/9 sets" / "Unsaved changes" (warning color) / "Saving…" / "Workout completed".
- Session tabs only when more than one workout exists on the date: "Session 1", "Session 2 ✓", plus "New session" while a draft is unsaved.
- Entry mode: a small two-option segmented control, "Guided" (lucide `ListChecks`) and "Quick entry" (lucide `Zap`), not two large cards.
- Weekly numbers as one muted line under the header: "This week · 2 sessions · 4 sets · 1,900 kg · +1 vs last week", linking to Progress. No stat cards on this screen.
- Exercise cards as described in section 2, with the new set row: index/type toggle, weight cell with unit label and stepper, reps cell with stepper, full-height log button, remove behind an overflow or swipe. Column headers appear once per card at 12px overline: SET · WEIGHT KG · REPS. Bodyweight exercises show "BW" in the weight cell; assisted exercises label the column "ASSIST KG" and show an "Assisted" badge.
- Per-exercise controls: move up, move down, remove (in an overflow menu, lucide `EllipsisVertical`), rest seconds input "90 s".
- "Add set" as a text button, "Add exercise" as a full-width dashed 52px button.
- Notes textarea, placeholder "How did this session feel?".
- Footer: secondary "Save changes" (disabled until dirty), danger icon "Delete workout", primary "Finish workout" (or "Reopen workout" when completed). On mobile these live in the docked session bar.
- States to draw: empty date ("No workout on this day" with "Start a workout"), first-ever empty state, unsaved, saving (only the acting control dims, never the whole page), saved, completed, and the inline error with "Retry save" and "Reload saved workout".
- Dialogs: "Add an exercise" picker with search, muscle group filter, and a list of name + "Chest · Barbell · Custom" rows; "Remove this workout?" confirm; the discard-changes confirm.
- Rest timer: docked in the session bar with 40px countdown "1:30", Pause/Start, Reset, +30 s, Skip. Show the collapsed launcher state when no timer is running.
- Quick entry mode: replaces the exercise cards with one monospace textarea (placeholder lines "Back squat 100x5 100x5 100x5", "Barbell bench press 80 x 8, 80 x 6", "Pull-up 8 7 6"), a live preview list where good lines show "Barbell bench press · 2 sets · 80 kg × 8, 80 kg × 6" and bad lines show the error (for example "No exercise matches that name") with a left rule in danger color, a count "2 exercises · 6 sets", "1,200 kg total volume", "1 line needs a fix", and a primary "Save workout". Draw both a valid and an invalid line.

### 4.2 Exercises (/exercises)
- Title "Exercises", primary "Custom exercise".
- Toolbar: search "Search exercises…", muscle group select (All, Chest, Back, Shoulders, Legs, Arms, Core, Full body), equipment select (All, Barbell, Dumbbell, Cable, Machine, Bodyweight, Kettlebell, Band). Results line "83 exercises · 83 standard".
- Catalog as a dense ruled list on desktop (name, muscle · equipment · tracking type, Custom badge, history link) rather than three-column cards; a compact card list on mobile. Custom exercises get edit and archive icon actions.
- Dialog "New exercise" / "Edit exercise": name, muscle group, equipment, tracking type (weighted, bodyweight, assisted, locked after creation with helper "Tracking type can't change after creation so history stays comparable"), description. "Archive Bench press?" confirm with the reassurance that history stays.
- Empty state "No exercises found".

### 4.3 Exercise history (/exercises/[id])
- Back link "Exercises", overline "CHEST · BARBELL", title "Barbell bench press".
- Three stats as a single ruled strip, not cards: Personal record "82.5 kg" with the record mark, "All-time"; Previous session "+2.5 kg" vs last; This week vs last week "+5 kg". For assisted exercises the record reads "Least assistance · 8 reps" and lower is better, so show the improvement in the positive color with a down-arrow, never a raw negative.
- Chart panel: segmented range "4 weeks / 12 weeks / All time", metric select (weighted: Heaviest set / Volume; bodyweight: Most reps; assisted: Least assistance plus a rep-count input), a big latest value "82.5 kg heaviest set", an inline SVG line chart with gaps where no session exists (not interpolated), Monday week boundaries, caption "Completed working sets only".
- Session history: "12 sessions", expandable rows "2026-08-30 · Upper body · Push · 80 kg", expanded set table (Set, Type, Weight, Reps, Logged), "Open workout" link, pagination "1 to 10 of 12" with Previous/Next.
- Empty state "No sets yet" with "Log your first workout".

### 4.4 Progress (/progress)
- Title "Progress", subtitle "Last 12 weeks".
- Stat strip: Training sessions 14, Exercises tracked 9, This week's volume 1,900 kg "Week incomplete".
- Ruled list of trained exercises: name, muscle badge, latest value "82.5 kg" with record mark where applicable, "8 sessions · Last trained 30 Aug", each row opens the history screen. Empty state "Your first working set starts the record".

### 4.5 Routines (/routines)
- Title "Routines", primary "Create routine".
- Routine cards: name "Upper body A", "5 exercises · 15 sets", numbered exercise list with "3 × 8" targets, actions "Start workout" (primary), edit, remove. Two columns on desktop, one on mobile.
- Editor dialog (wide): routine name, ordered exercise rows with Sets / Reps / Rest (s) numeric inputs, move up/down/remove, "Add exercise" opening the picker, Cancel and "Save routine". Draw the inline error state.
- "Remove Upper body A?" confirm. Empty state "No routines yet" with "Create your first routine".

### 4.6 Settings (/settings)
- Title "Settings". One form: Display name, Weight unit (Kilograms kg / Pounds lb) with helper "Existing weights are converted for display. Your original performance stays the same.", Timezone (searchable text input over the IANA list), Workout entry mode (Guided / Quick entry), Auto rest timer toggle. "Save preferences" disabled until dirty. Account section with "Sign out". Draw the saved toast "Preferences saved. Your recorded weights are unchanged."

### 4.7 Auth and system pages
- One centered card on the ground color with the engraved mark, no glow. Login: Email, Password, "Sign in", links "Create an account" and "Forgot password?", ghost "Resend confirmation". Sign-up adds "Your name" and "At least 12 characters" password helper. Reset request, new password, and the "This link has expired" page with "Back to sign in". Draw the inline error (danger) and inline success (neutral with check) banners.
- Setup screen: numbered four-step list with code chips. Error page "Couldn't load this page" with "Try again" and "Return to sign in". 404 "Page not found" with "Back to today".

### 4.8 App shell
- Desktop rail with mark, four icons (lucide `CalendarDays`, `Dumbbell`, `ChartNoAxesCombined`, `Layers`), active state as an ember bar on the rail edge plus ink color, avatar with initial at the bottom. Mobile bottom nav with the same four, and the docked session bar variant. Toast (success neutral with check icon, error danger with `TriangleAlert`), positioned top-center on desktop and above the session bar on mobile. Modal, Confirm, and Empty primitives.

## 5. Component sheet

Buttons: primary ember, secondary outlined strong-border, danger, ghost, small, icon (40px), disabled. Inputs: numeric plate cell with stepper, text, select, textarea, search. Set row in idle, editing, done, and error states. Index/type toggle. Stat strip cell. Badge (neutral, custom, assisted). Segmented control. Tabs. Dialog and confirm. Toast (two kinds). Empty state. Nav rail item and bottom tab in idle and active. Session bar with timer. Table row and expandable row. Pagination. Date control and week strip. Chart line, point, gap, axis, and caption. Record mark. Focus ring on a filled and an unfilled control.

## 6. Deliverables

- Desktop artboards at 1440 wide: Today (guided, mid-session), Today (quick entry), Exercises, Exercise history, Progress, Routines, Routine editor open, Settings, Login.
- Mobile artboards at 390 wide: Today (mid-session with session bar and timer running), Today (empty date), Today (quick entry), Exercise history, Routines, Login. Plus Today mid-session in the Daylight variant.
- Component sheet artboard. One artboard with the token table (colors, type scale, spacing, radii) and the four signature elements explained.
- Dark is the default. Use real data from section 4, never lorem ipsum.

## 7. Technical constraints (so the design maps to code)

Tailwind v4 plus one plain CSS file with custom-property tokens. Icons from lucide-react only, named by their lucide names. Google Fonts only. Native `dialog`, `select`, `input type=date`, and `textarea` elements, styled but not replaced. Charts are inline SVG. No third-party component kits, no animation libraries. Breakpoints 480, 760, 1020, 1180, 1600. `prefers-reduced-motion` disables all motion. WCAG AA text contrast, 3:1 on interactive boundaries, 44px targets, visible focus. Inputs never smaller than 16px on mobile.

## 8. Copy voice

Terse, factual, second person only when instructing. Nouns for wayfinding, verbs for actions, numbers for feedback. Personality is allowed once, in empty states, never in chrome. Examples: "SHOW UP. BUILD ON IT. / Today's session." becomes "Today". "Your week, in motion" becomes "+1 vs last week". "A fresh page for your training." becomes "No workout on this day". "Workout complete. Nicely done." stays, it is earned.

---

## Alternate direction A: "Logbook" (light, editorial)

Replace section 3 with this.

**Mood.** A beautifully typeset paper training diary. Warm off-white pages, black ink, one red pencil for marks. Rules and tables instead of cards. Confident, quiet, timeless. Think a good sports almanac or a ledger, not a wellness app.

**Palette (light by default, dark variant defined).** Paper `#F6F3EC`, sheet `#FFFFFF`, inset `#EEEAE1`, hairline `#D9D3C6`, strong border `#8E8778`, ink `#141311`, secondary `#5B574E`, tertiary `#8A8477`. Accent "pencil red" `#C8321E` for the primary action and done; record highlight `#B8860B` gold; danger `#A82A1A`; warning `#9A6B00`. Dark variant: paper `#151412`, sheet `#1C1B18`, ink `#F1EDE3`, accent `#FF5A3C`.

**Typography.** Display: **Fraunces** (Google Fonts) at optical size 72, weights 500 to 700, for page titles and stat values. Numerals in set rows: **IBM Plex Mono** 600, 22px, tabular. Body: **Inter**. Scale 13, 14, 16, 20, 30, 44. One overline style: 12px Inter 600, 0.08em, uppercase, tertiary.

**Layout principles.** Desktop: 200px labeled left column set like a table of contents (wordmark, four entries, account line at the bottom), no top bar, content column max 880px. Mobile: bottom nav, docked session bar during a workout. Exercise cards become ruled ledger blocks: a heavy top rule, exercise name in Fraunces, then a table with hairline rows, no boxes. Radii 2px controls, 6px dialogs. No shadows.

**Signature elements.** Ledger set rows with ruled underlines and mono numerals. A red check "tick" stamp for done sets. Progress as a red pencil line that fills under the session title. Dates set as "Mon 7 Sep" in small caps. Records marked with a gold asterisk and a footnote line "all-time best". The mark: a serif "F" with a long crossbar, like a printer's ornament.

**Mobile strategy.** Large mono numerals, ±2.5 kg and ±1 rep steppers, tap-to-cycle set type in the index, full-height tick button, session bar with the timer. First unlogged set above the fold.

## Alternate direction B: "Quiet Instrument" (warm neutral, minimal)

Replace section 3 with this.

**Mood.** A premium measuring instrument. Almost no chrome, the workout is the interface. Warm dark graphite or warm light stone, one refined accent, generous spacing, everything secondary tucked away. Think a high-end audio interface or a good timer app: nothing shouts, everything is legible at arm's length.

**Palette (dark default, light variant defined).** Ground `#141315`, surface `#1B1A1D`, raised `#232226`, inset `#0F0E10`, hairline `#2C2B30`, strong border `#4E4C55`, ink `#EDEAE4`, secondary `#B0ACA4`, tertiary `#7C7872`. Accent "brass" `#D8A650` for the primary action and done; record highlight `#F2E3B8`; danger `#E06A5A`; warning `#D8A650` at 60% (same hue, so unsaved and done share a family). Light variant: ground `#F3F1EC`, surface `#FBFAF7`, ink `#17161A`, accent `#A67A2C`.

**Typography.** Display and numerals: **Geist** (Google Fonts) 500 to 700, tabular numerals, set inputs 22px. Body: **Geist** 400 and 500. Rest timer and stat values in **Geist Mono** 500. Scale 13, 14, 16, 20, 26, 36. One overline style at 12px, 0.06em, uppercase.

**Layout principles.** Desktop: 56px icon rail, no top bar, content max 840px centered. Mobile: bottom nav, docked session bar. Surfaces are separated by spacing and a single hairline, never by nested boxes. Radii 6px controls, 10px cards, 14px dialogs. One soft shadow only on dialogs.

**Signature elements.** A single large "next set" focus block at the top of the workout on mobile that shows the current exercise, last time's numbers, and the two plate cells at 28px, with the rest of the session listed below in a compact form. Brass log button that fills with a 120ms ease. A thin brass progress ring around the timer countdown. Records marked with a small brass dot and "best" in the overline style. Mark: a lowercase "f" cut from a circle, single color.

**Mobile strategy.** Focus block first, steppers on both cells, tap-to-cycle type in the index, swipe to remove, session bar with the timer, first unlogged set always in view.
