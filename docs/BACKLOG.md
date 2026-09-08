# Forge backlog

Durable copy of the ranked feature audit run on 2026-09-08 (workflow wf_027cfd09-8ee: twelve candidate features scored by three judges on value, effort and fit, eight of them specced in detail, plus a 43-finding defect hunt with two findings refuted). The plan and the specs are reproduced here with their decisions intact; only repetition was removed. Where the codebase has moved since the audit ran, a "Since the audit" note says what changed. Consult DEVELOPMENT_PLAN.md for what has shipped and ARCHITECTURE.md for how the subsystems work.

## Scores

Value, effort and fit are 0 to 10 averages across three judges; score is the composite the plan ranks by; builds and skips count judges' explicit verdicts.

| Feature | Value | Effort | Fit | Score | Build / skip | Spec |
| --- | --- | --- | --- | --- | --- | --- |
| repeat-session | 7.3 | 3.3 | 8.0 | 12.0 | 2 / 0 | yes |
| one-rm-estimate | 6.3 | 4.0 | 7.7 | 10.0 | 2 / 0 | yes |
| warmup-generator | 4.7 | 3.7 | 7.7 | 8.7 | 1 / 1 | yes |
| plate-calculator | 5.7 | 5.3 | 6.7 | 7.0 | 2 / 1 | yes, lib shipped |
| rpe-rir | 5.7 | 6.3 | 7.0 | 6.3 | 2 / 1 | yes |
| exercise-substitution | 4.7 | 5.0 | 6.0 | 5.7 | 1 / 1 | yes, killed |
| weekly-recap | 4.3 | 4.7 | 5.7 | 5.3 | 1 / 1 | yes, killed |
| program-library | 5.7 | 6.3 | 5.3 | 4.7 | 1 / 1 | yes, rescoped |
| supersets | 5.3 | 7.3 | 5.7 | 3.7 | 0 / 1 | no, killed |
| data-export | 5.0 | 7.0 | 5.3 | 3.3 | 0 / 0 | no, restore half killed |
| csv-import | 5.3 | 8.0 | 4.7 | 2.0 | 0 / 1 | no, killed |
| rest-notifications | 6.0 | 8.3 | 4.0 | 1.7 | 0 / 1 | no, killed as specced |

## The state you are actually in

The plan opens with this and it still holds: DEVELOPMENT_PLAN.md leaves "configure development Supabase and verify real account confirmation, recovery, and live API integration" unchecked, along with production Supabase, email delivery, hosting and deployment. tests/live/supabase.spec.ts skips itself whenever its environment variables are absent, so it has never run. The product has never stored a real set for a real person. Every score above is a judgment about a gym app nobody has taken to a gym, so the choice is between shipping and continuing to spec, not between features.

## The single most important thing to do next

Fix the workout-save identity model, then put the app on a real Supabase project and log your own training on it for two weeks.

Why it beat every feature:
- Five of the six critical or high defects were one root cause: the client's notion of which server row it is writing to was derived per attempt instead of pinned. Quick entry minted a fresh workout id on every attempt, so a lost response duplicated the session and the mutation-id guard in save_workout could never fire. The saved-workout ref carried no date while the quick entry component was rendered without a key, so saving on the 8th, tapping to the 7th and saving again silently moved the 8th's session to the 7th with a success toast.
- The fix was smaller than any feature: one key prop and moving the id next to the mutation id, minted together and cleared together only on success.
- Every ranked feature reads or writes through this path, so building on it would have handed corrupted inputs to the e1RM chart, repeat-session and RPE.
- Dogfooding is what makes the ranking real, and this bug eats the dogfood.

Since the audit: both halves are done and the Tier 1 items below have largely followed. Commit 334fb39 scoped quick entry's saved workout to its date and gave each date one stable id that every retry reuses. Commit ea3858d fixed the guided-save deadlock by re-reading the session after a failed PUT and reconciling mutation ids. What remains is the second half of the instruction: a real Supabase project (SETUP.md, pending migrations first) and two weeks of your own logging before repeat-session starts. The plan's advice was explicit: do not start repeat-session before the save path is trustworthy and you have felt the thirty-tap problem yourself once.

## Ranked build order

1. repeat-session. Highest score by a wide margin, the highest-frequency action in the product, and genuinely additive to Routines rather than redundant. Build it as "duplicate this session's structure onto today, sets uncompleted, loads prefilled from suggestNext", reusing the instantiate-shaped code path in components/routines.tsx rather than a new one.
2. plate-calculator, before warmup-generator, because the warm-up generator is mostly plate math. Ship lib/plates.ts (done, commit 9d7c930) as a standalone in-gym readout on the weight field with bar weight and available plates in the profile; the generator then becomes a thin layer instead of a second implementation.
3. one-rm-estimate, gated on the chartSeries fix in the defects list. lib/schemes.ts already tells users their sets are "around 80-85% of your max" for a max the app cannot compute, and "Heaviest working set" misleads across rep-scheme changes. But e1RM is a fourth metric on the same unbounded day loop in lib/progress.ts; bound that loop first. Drop strength standards.
4. warmup-generator, only after plates exist and only if your own logging says you want it. Real weekly friction for the barbell lifter, dead weight on most of the catalog (lib/validation.ts forbids weight on bodyweight and band entries). Scope it to weighted tracking with Barbell or Dumbbell equipment and nothing else.
5. rpe-rir: decide after four weeks of real logging, not now. It is the only item that changes the schema and the only one that changes suggestNext from mechanical double progression to autoregulation. Right long-term move, wrong bet on zero usage data. If your log shows grinding and stalling on the 2.5 kg step, build it; if not, you avoided the highest-effort item on the board.
6. program-library, as a content task, not a feature. lib/share.ts decodeRoutine already resolves exercises by stable seed UUID. Hand-encode four to six canonical splits as share payloads and render them in the Routines empty state. An afternoon, no new plumbing, and it kills the worst onboarding moment. Do not build the browsable program-library system that was specced.

## Kill these

- exercise-substitution. components/library.tsx already filters by muscle and equipment in both Library and the picker, and the picker has since become faster with a bigger catalog, which strengthens the existing answer. Serious lifters carry the substitution table in their heads. Dead. The spec is kept below for the record.
- supersets (no judge said build). The only real bug inside it, the rest timer firing between halves of a pair in components/today.tsx, is a defect fix, not a feature. Fix the timer if it bothers you; do not build grouping, ordering and rendering at effort 7.3.
- weekly-recap. Progress already renders sets by muscle this week versus last, and Today's week line already renders sessions, sets and volume with the delta. The only new content was a streak, which is a retention gimmick in a product whose identity is "a precise instrument" and which DESIGN_BRIEF.md forbids outright. Spec kept below.
- csv-import. Its strongest argument is migrating users away from Strong. You have no users. Revisit only when a real person says they will not switch without it.
- rest-notifications as specced. Background push in an installed PWA is the expensive tenth of the value. The valuable nine tenths is navigator.vibrate plus a short WebAudio beep when the countdown in components/shell.tsx hits zero: an hour of work that also fixes the accessibility defect where "Rest complete" is injected into a DOM that already contains that text. Do the hour, kill the feature.
- data-export: kill the restore half permanently. It can overwrite a real log and Supabase already offers point-in-time recovery (SETUP.md). Keep set-level CSV export as a one-day chore whenever wanted; it is not a roadmap item.

## Defects that jump the queue

Tier 0, before anything else ships:
1. Quick-entry save identity (components/quick.tsx; three critical and two high findings that were one defect reported three times). Silent, unprompted, permanent loss of a logged session with a success toast on top. Since the audit: fixed in 334fb39. The plan asked that the fix land with a test that saves, changes date, saves again and asserts one row on the original date; check that tests/browser/quick-entry.spec.ts covers exactly that before calling it closed.
2. chartSeries unbounded day loop in lib/progress.ts plus its enabling hole: the workout date input in components/today.tsx has no min or max and dateSchema in lib/validation.ts is a bare ISO date with no range. One mistyped year persists a row that hangs or exhausts memory on the exercise page on every future visit, and the repair path is the page that crashes. Three-part fix: clamp dateSchema to a sane window, add min and max to the input, and bucket chartSeries by week once the span exceeds about 26 weeks, which also fixes "All time" rendering 1,097 points into a 720px SVG. Since the audit: the loop is bounded by commit 009f396 (window clamped to a sane range, capped at three years of the most recent real data, points beyond a month ahead ignored). dateSchema is still a bare ISO date and the date input still has no min or max, so a far-future or far-past workout row can still be persisted; it just no longer crashes the exercise page.
3. Guided-save deadlock after a lost response in components/today.tsx: save kept the stale revision while a fresh mutation id was minted on the next edit, so every later save returned 409 forever and the only escape discarded everything logged since. Since the audit: fixed in ea3858d.

Tier 1, before the first feature, because dogfooding hits them in week one:
4. Full bootstrap refetch per logged set. save ends with a refresh that re-runs the whole bootstrap (profile, full catalog, 84 days of sessions with every set), so a 32-set session is 32 PUTs plus 32 bootstraps on gym LTE, plus 32 delete-and-reinsert cycles in save_workout. Related waste: the dashboard layout ships 84 days of history into Settings and Routines, and the bests effect on Today fires one all-time history request per exercise and discards most of each response. Since the audit: commit 09d34c8 updates Today's week summary from the saved workout and refetches only on completion, and commit f3c7342 adds points=1 to the history endpoint so the bests effect no longer expands ten sessions per exercise. The layout still ships history into Settings and Routines.
5. Two one-line correctness bugs. The quick-entry field label wraps the Decrease button first, so tapping the word "REPS", "SETS" or "WEIGHT" decrements the value (fix with aria-labelledby on a non-wrapping span, or put the input first in DOM order). Fractional and huge numbers: the quick-entry rounding was two decimal places rather than integer, so 12.5 reps passed the client and the whole session was rejected server-side with a raw zod string, and Sets accepted a million and built that many rows during render. Since the audit: fixed. ea3858d clamps typed reps in guided mode, and 09d34c8 binds each quick-entry label to its own input and clamps typed values to the field range.

Tier 2, while you are in there: tests/database.test.ts hardcodes migrations 0001 to 0004 while tests/harness.ts loads 0001 to 0006, so the database suite tests a save_template that exists in no deployed database. Replace both hardcoded lists with a sorted readdir of supabase/migrations; two lines that close a drift class permanently. Since the audit: fixed in f3c7342; both files replay a sorted readdir of the folder.

## What the judges and hunters collectively missed

1. Nobody noticed the product is not deployed. Forty-three defects and eleven specs for an app that has never authenticated a real user. The suite that would prove auth, RLS and the API route work end to end skips itself, and the harness reimplements the API in about forty lines with no origin check, so "mutations require matching request origin" has never been exercised. The "no test coverage for the origin guard" finding was scored too low for that reason. Since the audit: f3c7342 makes the database test call the real route handler, so the same-origin guard is now exercised without a hosted project; the deployment gap itself remains.
2. The three critical quick-entry defects were one defect reported three times, which inflated the count and made the fix look bigger than it was.
3. "components/quick.tsx has no unit or browser coverage" went stale during the audit (tests/quick-entry.test.ts and tests/browser/quick-entry.spec.ts exist), but the point sharpened: those tests covered steppers, prefill, units and line building and none of them exercised the date change or the retry path. Coverage was added around the bug.
4. Two "features" are already half-built and mis-scoped. program-library is share-payload content, not a system. rest-notifications' whole in-gym value is a vibrate and a beep. Both were scored at feature effort and should be scored at chore effort, which is why both ranked wrong.
5. The accessibility defects share one unnamed root cause: focus is never managed on any async or modal transition. Modals unmount without restoring focus (components/ui.tsx), setBusy disables the focused control on Today and History, and the rest timer swaps its launcher for its panel. That is one restore-focus helper in ui.tsx plus one rule ("do not disable the focused element mid-request"), not seven fixes. Since the audit: ea3858d applied the rule to Today's date and week-strip buttons with aria-disabled, and ada5dad restored focus after modal close, moved focus into and out of the rest timer, and made its live region persistent, with tests/browser/a11y.spec.ts covering it.
6. The save_workout and save_template omitted-array hole is scored low and is the one place where "the database validates independently" is false: jsonb_array_length of NULL is NULL, the guard does not fire, and both RPCs are executable from the browser with the shipped publishable key. Confined to the caller's own rows, so low is defensible on blast radius, but the fix is coalesce(jsonb_array_length(...),0) in two functions. Since the audit: fixed in 202609080008_guards.sql (commit f3c7342), which rejects an absent or non-array exercise list in both functions.

## Notes that apply to every spec below

- Migration numbering. Every spec proposed a file named 202609080006_something.sql. That number is taken by 202609080006_catalog.sql, and 0007 (plates) and 0008 (guards) are committed too. Number any new migration after the highest file present when you start, and note that 0008 restates save_workout and save_template, so a spec that patches either function must slice from 0008, not from schema.sql.
- Migration style. One line, no comments, no trailing newline (scripts/check-format.mjs checks .sql too). Use add column if not exists and drop constraint if exists guards so a re-run is idempotent. A migration that touches an RPC must restate the whole function body, as 202609080005_rep_ranges.sql does for save_template. create or replace preserves the function's grants; a return-type change forces drop and create, which loses the grant and must re-issue the revoke and grant pair or every caller gets a 403.
- Test loader lists. Since f3c7342 both tests/harness.ts and tests/database.test.ts replay a sorted readdir of supabase/migrations, so a new migration is picked up automatically; the specs' instructions to add loader lines are obsolete.
- Numeric columns arrive as strings from PGlite and from PostgREST. Every read of a numeric column coerces with Number, exactly as weight_kg already does throughout lib/progress.ts.
- Catalog size. The specs were written against 83 standard exercises; there are now 150 across two migrations. Any test that parses the seed file must parse both migrations and assert 150.
- Single-line source. Every spec's edits to today.tsx, progress.ts and history.tsx are given as unique anchor substrings for that reason. Run npm run check:format before typecheck; it reports the byte offset of any comment or newline.
- Verification order for every feature: npm run check:format, npm run typecheck, npm test, npm run test:db, then npm run test:e2e.

## repeat-session

Summary. A secondary "Repeat" action on Today opens a small picker of recent sessions drawn entirely from the history already in memory, and starts today's workout from the chosen one: exercises and their logged sets copied as fresh drafts, with each exercise's working sets rewritten to its next double-progression target anchored on the user's most recent performance of that exercise anywhere in history, not on the session being copied. Zero new API surface, zero SQL. One new pure function repeatWorkout in lib/progress.ts composing pointsFor, suggestNext, copySets and applySuggestion; the save is the existing PUT /api/workouts/:id, the same path Routines.start uses. Drafts live in React state until that single PUT.

Decisions:
- Persist immediately, like Routines.start, rather than holding an unsaved draft. The objection that a mis-tap pollutes week counts does not hold: weekSummary, muscleSummary and history_points all count only completed sets, so an unlogged repeated session contributes zero everywhere and the existing Trash button removes it. Persisting buys reload survival and the already-tested retry-safe path.
- A picker, not "repeat last session". The target user is on a split, where the immediately previous session is the wrong day most of the time. The picker reads data.history only (the 84-day bootstrap window), deduped by title plus sorted exercise-id set, capped at eight rows: one useState and a Modal over the existing picker-list CSS. A sessions-list API with paging is the scope risk being ruled out.
- Copy what was actually done: per exercise, the completed sets, falling back to the entry's own sets if nothing was logged. Warm-ups keep their copied loads because applySuggestion already skips non-working sets.
- Archived exercises are filtered client-side, and this is load-bearing: save_workout's archived guard only exempts exercises already attached to that session, so a new session containing an archived exercise raises "Exercise unavailable" and the whole one-tap action fails with a bare 400. validateLoads does not catch it first because archived rows are still selectable. The client can detect it because bootstrap ships archived rows with archived_at set.

Migration: none, verified. No new table, column, RPC, policy, validation or route branch; workoutSchema accepts the payload unchanged and a unit test asserts it. Softening the opaque "Exercise unavailable" error is deliberately out of scope because it would mean replacing all of save_workout.

Files: lib/progress.ts, components/today.tsx, tests/progress.test.ts, tests/database.test.ts, tests/browser/workflows.spec.ts, docs/ARCHITECTURE.md, docs/DEVELOPMENT_PLAN.md.

Steps:
1. lib/progress.ts: add WorkoutExercise to the type import and add repeatWorkout(source, history, exercises, date, unit, id, now) immediately after instantiate. It maps exercises by id; filters history to sessions before the target date (or the same date with an earlier start, mirroring Today's per-card filter, with now passed in to keep the function pure); walks the source's entries in position order; skips archived or unknown exercises into a skipped list; copies completed sets (or all sets when none are completed) with copySets; takes suggestNext over the last point of pointsFor(before, exercise); applies it when non-null, otherwise keeps the copied loads; renumbers positions 0..n-1 after skips; and returns a workout with revision 0, completed_at null, the source's title, empty notes and a fresh id, plus the skipped names.
2. components/today.tsx: import Repeat and Play from lucide, repeatWorkout from progress, Modal from ui. Add repeat state and a separate repeatRef holding key, workout, mutation id and skipped names, keyed on source id plus date (not source id alone, or a failed repeat retried after a date change re-PUTs the old workout_date).
3. Candidate list next to the weekSummary derivations: history sessions on or before the date, not the current one, with at least one completed set, sorted ascending, deduped through a Map keyed on title plus sorted exercise ids (a Map keeps the last value, so ascending plus reverse yields newest-first), sliced to eight. No useMemo; the file uses none.
4. repeatSession(source): bail if busy; confirm if dirty; build through repeatWorkout only when the ref key differs; if every exercise was archived, notify and return without caching; PUT the workout with the stable mutation id; refresh; clear the ref; select the new workout; notify, naming skipped exercises when any. The stable workout id plus stable mutation id across failures is what makes a double tap or a retry idempotent.
5. Page heading: an inline-actions wrapper with a secondary Repeat button (only in guided mode, only when candidates exist) before the primary New workout. Empty state offers both "Repeat a session" and "Start a workout".
6. Picker modal: title "Repeat a session", a muted explanation, picker-item rows showing title, short date, exercise count and working-set count, and a footer note pointing to Routines for a saved plan with its own targets. No new CSS. This copy is what keeps the affordance distinct from Routines.
7. Docs: Progression gains the anchoring rule; UI flows gains the repeat description; DEVELOPMENT_PLAN gains a checked Phase 1 bullet.

Tests:
- progress.test.ts: repeating a 3x80x8 session from 2026-08-17 when the same lift was trained at 100 on 2026-09-01 yields 102.5 on every working set, not 82.5 (the headline regression guard); reps 8, all incomplete, set numbers 1..3, revision 0, the source untouched.
- progress.test.ts: archived exercises are skipped and reported by name, positions renumber contiguously, the payload passes workoutSchema, and the all-archived case returns zero exercises.
- progress.test.ts: only logged sets are copied; a logged warm-up keeps its load and type; the never-logged draft is dropped; an entry with only drafts copies them verbatim; an exercise with no completed history keeps its copied loads.
- database.test.ts: a new session whose only exercise is an owned archived exercise rejects with "Exercise unavailable" and leaves no orphan session row; the same payload with a standard exercise saves at revision 1. Place it after the routine subtest and before the delete subtest.
- workflows.spec.ts: append to the first spec. Repeat the seeded "Upper body · Push" session after logging 82.5x9 on bench in a later session; expect the toast, the title, and bench set 1 at 85 with 9 reps (anchoring on the copied session would give 82.5/8), and a visible Log button proving it landed as a draft. All eight seeded sessions dedupe to one picker row.

Risks:
- Progression anchoring is the silent corruption: pointsFor([source]) reads naturally and hands back a lighter target than the user already lifted. Structural mitigation: history is a required parameter and the first test asserts 102.5.
- Archived exercises: a narrow race between bootstrap and the PUT survives and surfaces through the error toast.
- Duplicate sessions without the stable ids in the ref.
- Non-uniform sessions are flattened: applySuggestion rewrites every unlogged working set to one pair, so a top set plus back-offs becomes the top target throughout. Deliberate and consistent with "Use target"; a per-set delta would need its own branches and test matrix and would roughly double the feature.
- 84-day horizon: older sessions are silently not repeatable. Accepted; extending it is a separate, explicitly scoped piece of work.
- A fourth start path next to Routines, Copy previous, Use target and quick entry; distinctness rests entirely on the copy and hierarchy in step 6.
- Blast radius: today.tsx is the largest single-line file in the repo, covered by one Playwright spec; keep to the named insertion points.
- Queue position: the spec noted it jumped three open Phase 1 items. Since the audit all three are done (quick-entry tests, rep-range targets, and the migration is now a documented pending item), and there is still no "save this workout as a routine", which would be the cheaper bridge to a shareable, compounding artifact.

Size: medium.

## one-rm-estimate

Summary. Add an "Estimated 1RM" chart metric for weighted exercises only, computed identically in SQL (history_points, for the exercise page) and TypeScript (pointsFor, for Progress, Today and the harness). Formula, one and canonical: per completed working set, e1RM = weight_kg when reps is 1, otherwise weight_kg × (30 + reps) / 30, only for weighted tracking and reps 1 to 12; per session take the maximum; round to six decimals; null when no set qualifies. Multiply before divide so the double stays exact longer and agrees with Postgres numeric after round(...,6). Singles are special-cased because raw Epley returns 103% of the load at one rep, which renders a true single as a dip. The 12-rep cap is the honesty bound and is stated in the chart caption. Strength standards are out of scope: they need bodyweight and sex, which the profile does not carry and DEVELOPMENT_PLAN.md lists as out of scope.

Verified in PGlite against the real chain: the migration applies; SQL returns 132.000000 and TypeScript 132 for a session of 100x5, 120x3, a 200x1 draft, 60x20 and a 150x2 warm-up (a cross-maximum of weight 120 with reps 5 would wrongly give 140, so the fixture proves pairing); has_function_privilege is true for authenticated and false for anon after the migration; SQL and TypeScript rounding agree across nine cases including a 225 lb conversion, 2000x12, 140x1 and the half-ulp boundary.

Deliberately untouched, keeping the blast radius to one union member per file: recordFor and the Personal record stat stay heaviest-set, the Progress catalog cards stay heaviest-set, Today's PR detection is unchanged, and the metric stays client-side React state, never a validated query parameter. HistoryPoint literals are built in exactly two places (pointsFor and the spread in exerciseHistory), so a required new field breaks nothing else.

Migration: yes. history_points gains a return column, so create or replace is illegal: drop the function, create it with e1rm appended last in the return table, then re-issue revoke from public and anon and grant to authenticated (the drop destroys the original grant, and a fresh function grants execute to public by default). Only the return clause and the final aggregate differ from the schema.sql original; the valid and assisted CTEs are copied verbatim, so the security-invoker and auth.uid() filters and the completed-working-set filters are preserved by construction. The final aggregate is round(max(case when reps=1 then weight_kg else weight_kg*(30+reps)/30.0 end) filter (where tracking_type='weighted' and reps between 1 and 12),6). PostgREST serialises the scale-6 numeric as a string, hence Number in lib/data.ts. Add it to the hosted-project pending list.

Files: the new migration, lib/types.ts, lib/progress.ts, lib/data.ts, components/history.tsx, tests/harness.ts, tests/database.test.ts, tests/progress.test.ts, tests/browser/workflows.spec.ts, docs/ARCHITECTURE.md, docs/DEVELOPMENT_PLAN.md.

Since the audit: the chartSeries loop this feature was gated on is bounded (009f396), so the gate is cleared.

Steps:
1. Write the migration exactly as described.
2. lib/types.ts: insert e1rm:number|null in HistoryPoint between volume and reps.
3. lib/progress.ts pointsFor: after the loads array, compute estimates for weighted exercises from the valid (completed working) sets with reps 1 to 12, then emit e1rm as Math.round(max×1e6)/1e6 or null, immediately after volume in the point literal. The reps-null test is redundant at runtime but needed to narrow the type.
4. lib/progress.ts chartSeries: widen the metric union with "e1rm". No body change; the default per-day aggregator is already Math.max, the correct e1RM roll-up for two sessions on one day.
5. lib/data.ts exerciseHistory: coerce e1rm with Number like volume, or the chart concatenates strings with no type error.
6. components/history.tsx: widen the Metric type; make the relevant filter exclude points whose e1rm is null (a session of only 13+ rep sets) so the summary and delta do not read as dashes while the chart draws. Do not restructure the filter into a generic value-is-null form: value is declared after relevant and reordering the single-line file is where the task goes wrong. value, display, delta, weekly and the session table need no edit.
7. Copy: the option "Estimated 1RM" inside the existing weighted-only fragment after Training volume (this is the whole gating story); the unit label "estimated 1RM"; the empty-state title "No sets between 1 and 12 repetitions."; and a caption sentence stating the Epley formula and the 1 to 12 limit.
8. TrendChart aria-label: announce "estimated 1RM progression" instead of the raw key; the existing weight, volume, reps and assistance labels are unchanged so the current Playwright matcher keeps passing.
9. tests/harness.ts: add the migration after the rep_ranges line (schema parity and a syntax gate; the harness serves history through pointsFor).
10. tests/database.test.ts: add both 0005 and the new file after 0004. Adding 0005 is safe: the routine subtest omits target_reps_max and the new save_template casts the missing key to null.
11. Run the gates. Docs: ARCHITECTURE Progression gains the formula sentence and "chart metric only, does not feed personal records"; DEVELOPMENT_PLAN gains a checked line and the pending-migration bullet names the new file. Leave the stale "Pass 20 checks" sentence alone unless the whole count is reconciled.

Tests:
- progress.test.ts: the pairing fixture above yields e1rm 132 and weight 120, and explicitly not 140; singles return the weight (140x1 with 60x20 gives 140, not 144.67); a lone 20-rep set gives null e1rm with weight 60; 100x8 gives 126.666667; bodyweight and assisted give null.
- progress.test.ts: extend the same-day aggregation test to prove the per-day roll-up is a maximum, not a sum (100x5 and 110x5 on one day give 128.333333).
- database.test.ts: the existing history subtest (one 100x8 set) additionally asserts Number(e1rm) is 126.666667; do not insert a successful save mid-file because later subtests depend on the session sitting at revision 1.
- database.test.ts: a final subtest saves the pairing fixture, reads history_points, and asserts the SQL value equals pointsFor's value and both are 132; plus has_function_privilege true for authenticated and false for anon. This is the dual-implementation guard the design hinges on and the one-line test that catches the dropped-grant 403.
- workflows.spec.ts: in the chart workflow, select the e1rm metric and expect the img named "estimated 1RM progression".

Risks:
- Drop discards the grant: forgetting the revoke and grant pair either 403s every history chart or exposes the function to anon.
- Dual implementation drift: the number is computed in SQL for the exercise page and in TypeScript elsewhere because the RPC returns points with empty set lists and pointsFor cannot reconstruct it. Mitigated by one pinned expression shape and the SQL-equals-TypeScript assertion; residual risk is a value landing exactly on a sixth-decimal half boundary, unreachable for realistic loads.
- Warm-ups and drafts leaking in: HistoryPoint.sets is unfiltered; compute from the completed-working subset only. The fixture's draft single and warm-up fail loudly if this regresses.
- Formula edges: Brzycki divides by zero at 37 reps, which is why only Epley ships.
- Interpretation noise: with no RPE, a back-off set or an easy day produces a jagged line that reads as strength loss. Mitigated by the per-session maximum and by keeping it strictly a chart option.
- Loader-list drift: a migration added only to the harness is unexercised by the SQL suite.
- Weighted-only reach: bodyweight and assisted users see no dead control because the option lives inside the weighted gate.
- Single-line friction in progress.ts and history.tsx.
- Scope creep back into strength standards: a migration, PII on a private-by-default product, and a normative dataset of unclear licensing for a cosmetic label. Excluded by decision.

Size: medium.

## warmup-generator

Summary. Warm-up generation as an opt-in per-exercise inline action on the guided Today card, mirroring "Use target" and "Copy previous". All persistence already exists (the set_type check constraint, setSchema's enum, save_workout passthrough, completedSets exclusion, the warmup CSS), so the feature is one pure module lib/warmup.ts plus a surgical edit to components/today.tsx: no migration, no API change, nothing in browser storage.

Decisions:
1. Ramp is percent of the working weight: 40% x5, 55% x5, 70% x3, 85% x2. At most four rows, never an empty-bar row.
2. Rounding is anchored at a floor: 20 kg or 45 lb converted when the exercise's equipment is Barbell, otherwise 0. weight = floor + round((pct × top − floor) / loadStep) × loadStep, so every barbell output is bar plus whole 2.5 kg or 5 lb increments and is loadable with 1.25 kg or 2.5 lb pairs. Barbell top 100 kg gives 40, 55, 70, 85; 225 lb gives 90, 125, 160, 190; 30 kg gives a single 25x2 row. No profile migration and no plate-inventory model.
3. Rows at or below the floor, at or above the top, or duplicating an earlier rounded row are dropped, so light lifts degrade to one or two rows or none.
4. Tracking type is gated inside lib/warmup.ts, before the database trigger can fire: bodyweight returns nothing (validate_set_load rejects weight) and assisted returns nothing (assistance decreases toward the work set, so a percentage ramp is inverted). Empty result means no button, mirroring how "Use target" renders only with a suggestion.
5. Source weight: the maximum working-set weight entered so far, falling back to suggestNext's weight, already computed in the card. This solves "ramp to what?" without new state.
6. Idempotent toggle: if the first set is an unlogged warm-up the label becomes "Clear warm-up" and the action strips leading unlogged warm-ups; pressing "Warm-up" twice replaces rather than duplicates; completed warm-ups and every working set are never touched.
7. applyWarmups renumbers the whole array 1..n so the unique set-number refine and constraint hold; save already re-derives set numbers at payload time.
8. Labels: aria-labels and the index numeral switch from the raw set number to setLabels: warm-ups display "W" with label "warm-up N", working sets display and label "set N" counted among working sets only. Invariant: with zero warm-ups the label equals "set " plus set_number, so the existing Playwright specs keep matching exactly and the 22px single-glyph index cell is unchanged.
9. The 100-set cap is respected: applyWarmups takes a limit and keeps only the heaviest rows that fit.

Migration: none, verified: the set_type check, save_workout's verbatim insert, history_points' working-set filter and setSchema's enum all exist. Deliberately not built: profile columns for bar weight, ramp count or per-exercise opt-out; if demanded later it is a separate profiles migration plus a profileSchema field, with no RPC to replace because profile writes go through PostgREST.

Files: lib/warmup.ts (new), components/today.tsx, tests/warmup.test.ts (new), tests/browser/workflows.spec.ts, docs/ARCHITECTURE.md, docs/DEVELOPMENT_PLAN.md.

Steps:
1. lib/warmup.ts exports warmupRamp, barWeight(unit), warmupTarget(sets, suggestion), warmupSets(top, tracking, equipment, unit), applyWarmups(sets, rows, unit, id, limit=100) and setLabels(sets). progress.ts does not import it, so there is no cycle.
2. today.tsx imports: ChevronsUp from lucide; the four warmup functions.
3. Per-card derivations before the article: warmupRows, warmedUp (first set is an unlogged warm-up) and labels.
4. The button, inserted before the "Copy previous" button so the row reads Use target, Warm-up, Copy previous; rendered only when warmedUp or rows exist; aria-label "Add warm-up sets for X" or "Clear warm-up sets for X"; updates the entry through applyWarmups and notifies. No new CSS.
5. Open the set-row map to a block body so it can read labels[si].
6. Replace the nine occurrences of `" set "+s.set_number` inside the row with `" "+label`, and the index text child with display. Do not touch the three other set_number sites (save payload, remove-set renumbering, Add set).
7. tests/warmup.test.ts in the style of schemes.test.ts.
8. Append a fourth Playwright spec, kept last because spec 1 asserts a three-set count that must not see warm-up rows.
9. Docs: UI flows gains the ramp, the toggle and the "excluded from every metric" rule; Progression restates that warm-ups never count; DEVELOPMENT_PLAN gains a checked bullet.
10. Verify; the label invariant is proven by specs 1 to 3 passing unmodified.

Tests (tests/warmup.test.ts):
- kg barbell ramp for 100 gives 40x5, 55x5, 70x3, 85x2.
- lb barbell ramp for 225 lb is 90, 125, 160, 190 and every row is bar plus whole 5 lb increments.
- non-barbell floor is zero: 30 kg dumbbell gives 12.5, 17.5, 20, 25.
- light loads degrade: 30 kg barbell gives one 25x2 row; 22.5 kg barbell gives none; 2.5 kg machine gives none.
- bodyweight and assisted return nothing.
- dedupe and strict bounds: a 5 kg machine gives a single 2.5 row.
- warmupTarget precedence: the heaviest entered working weight beats a suggestion; all-null weights fall back to the suggestion; neither gives null; warm-up rows are ignored.
- idempotency: applying twice equals applying once; set numbers contiguous; working sets keep ids, weights, reps and completion.
- a completed leading warm-up survives; applying with no rows removes only leading unlogged warm-ups.
- 100-set cap: 98 existing plus four rows yields exactly 100 keeping the two heaviest; 100 existing adds none.
- server contract: the output plus a mutation id passes workoutSchema and validateLoads.
- label invariant with and without warm-ups; all labels unique.
- Browser spec: new workout, add bench, the warm-up button is visible before any weight is typed (the suggestNext fallback), fill 100, click, expect warm-up 1..4 at 40/55/70/85 with warm-up 4 reps 2, working set 1 still 100, log set 1, read bootstrap and assert seven sets numbered 1..7 with four warm-ups incomplete and the fifth working and complete; then clear, expect no warm-up labels, save, and assert three working sets again.

Risks:
- Row density on the densest screen: four extra 52px rows roughly double a squat card. Mitigated by opt-in, the four-row cap and one-tap clearing; if the mobile screenshot reads badly, cut the ramp to three rows (45, 65, 85%) rather than adding a preference.
- Step 6 is the real regression surface; safe only because setLabels reproduces the old labels with no warm-ups.
- Personalisation pressure (six-step ramps, singles, 15 kg bars): accept for v1.
- Bar-anchored, not plate-aware: a gym without 1.25 kg plates still sees 22.5. Non-barbell equipment uses loadStep multiples, wrong for fixed-increment stacks. These are hints on draft rows, one stepper tap to correct.
- Gate tracking type in the module, not only in the render condition, or a bodyweight caller fails the whole save.
- Warm-ups now appear in the history detail table's set-type column: correct, but a visible change worth documenting.
- Quick entry stays working-sets-only: deliberate, do not extend the parser.
- Payload size: every save reinserts the whole workout; the limit keeps it legal.

Since the audit: lib/plates.ts exists, and the plan ranks the plate readout ahead of this feature so the generator can lean on it for loadable rounding instead of the bar-anchored heuristic. There are now 34 Barbell exercises among 150.

Size: medium.

## plate-calculator

Status: the lib module shipped in commit 9d7c930 as lib/plates.ts with tests/plates.test.ts; only the UI remains. The shipped module differs from the spec in ways the UI work must respect, listed after the spec.

Summary as specced. A zero-config, unit-native barbell plate readout rendered as a full-width sub-row under each qualifying set row in guided logging, with a one-line hint in quick entry, plus scalar profile columns for bar weight and rack selection.

Key correctness decision: the calculation runs in the display unit, never in canonical kg, and in integer multiples of the rack's smallest plate. That single rule fixes every float hazard the reviewers raised: 225 lb round-trips through kg as 224.99999944884433 lb, per side 89.99999972, which naive greedy renders as 45+35+10 instead of 2x45; an empty 45 lb bar yields a per-side 3.9e-7 that renders as "0.0000004 lb remaining" unless quantised.

Rack versus display unit: bar weight is nullable, null meaning the standard bar for the display unit (20 kg or 45 lb), and plate denominations are fixed per-unit constants selected by a plate_set enum (standard, micro, minimal), so the default path is exact in both units. A user with a bar set in one unit and display in another gets an honest "closest 134.09 lb" line styled as approximate rather than a silent lie; the residual is physically real.

Scope guards: gated on equipment Barbell and tracking weighted (assisted weight is inverted assistance; bodyweight has no weight cell) plus weight greater than zero. Counts per denomination are unlimited, so decomposition stays greedy, heaviest-first, always exact but not always minimal (165 lb gives 45+10+5 rather than 35+25), an accepted and documented trade.

No new API surface: config rides the existing profile PUT; bootstrap selects every profile column so nothing in the data layer changes; the calculator persists nothing, so save_workout, revisions, validateLoads and the workout payload are untouched; the table-level grant and the existing profile policies cover new columns with no new policy.

Biggest non-obvious hazard: profileSchema is strict and the route updates the profile with the whole parsed object, so every site that hand-builds a full profile literal must gain the new fields or an unrelated high-traffic path starts returning 400. There are exactly three: the settings useState literal, changeMode in today.tsx (the entry-mode toggle enumerates every field rather than spreading), and the hardcoded UPDATE column list in tests/harness.ts. The live spec only reads preferred_unit.

Migration as specced: profiles gain plate_helper_enabled boolean not null default true, plate_set text not null default 'standard' with a check on the three values, and bar_weight_kg numeric(6,3) with a drop-then-add check (null or greater than 0 and at most 500). No RPC changes: save_workout, save_template and history_points never touch profiles, handle_new_user inserts with an explicit column list so defaults apply, and touch_updated is column-agnostic. Three scalar columns rather than a numeric[] inventory because every existing preference is a scalar and per-denomination counts turn exact loading into bounded subset-sum. bar_weight_kg arrives as a string; coerce before zod or arithmetic.

Files as specced: the migration, lib/plates.ts, lib/types.ts, lib/validation.ts, components/ui.tsx, components/today.tsx, components/settings.tsx, components/quick.tsx, app/globals.css, tests/plates.test.ts, tests/harness.ts, tests/database.test.ts, tests/browser/workflows.spec.ts, docs.

Steps (UI and plumbing; the lib steps are done):
1. Migration, then lib/types.ts: a PlateSet type next to EntryMode (defined in types.ts so plates.ts can import it without a cycle) and the three Profile fields.
2. lib/validation.ts profileSchema: plate_helper_enabled boolean, plate_set enum, bar_weight_kg finite, greater than 0, at most 500, nullable, with bounds mirroring the SQL check so the database never rejects what zod accepted. Touch no other schema.
3. Move WeightInput from the end of today.tsx into components/ui.tsx, exported, with an optional max prop defaulting to today's 2000 kg or 4409 lb so settings can cap the bar at 500 kg; it reuses the proven raw-string-plus-last-value-ref pattern instead of a second weight input.
4. today.tsx changeMode first: add the three fields with Number coercion on bar_weight_kg. Missing this silently breaks the entry-mode toggle.
5. today.tsx sub-row: a per-card gate (helper enabled, Barbell, weighted); wrap each set row in a Fragment keyed by set id; compute the load when weight is positive; render after the set-row div and still inside set-table a div with class plate-row (plus approx when not exact), role note, aria-label "X set N plates", a "Plates" label span and the summary. Rationale: set-grid is a fixed five-column grid at every breakpoint with no sixth cell, so the readout is a full-width sibling, always visible rather than tap-gated because a calculator that costs a tap is slower than mental arithmetic; do not nest it inside set-row because the done and warmup descendant selectors would recolour it.
6. settings.tsx: extend the useState literal; add a Plate calculator toggle (copy the rest-timer block), a "Plates available" select listing the racks for the unsaved unit with a live denomination preview, and a Bar weight WeightInput keyed on the unsaved unit so it remounts in the new unit, with an empty input meaning the standard bar.
7. quick.tsx: the same gate on each row, converting the row's display weight back to kg for the call, rendered as a plate-hint span at the end of the row footer. The quantisation absorbs the display-kg-display round trip.
8. globals.css: append plate-row, plate-label, plate-list (Barlow Condensed numerals), the approx warning colour, plate-hint, and the 480px override, at end of file so the base rules do not override their own media-query versions.
9. tests/harness.ts: add the migration and extend the hardcoded profile UPDATE column list and parameter array; miss it and every preference-saving browser test fails.
10. tests/database.test.ts: add 0005 and the migration; add the two subtests below.
11. Extend the existing browser specs; do not add a fourth.
12. Docs: ARCHITECTURE Progression describes the display-unit quantisation; UI flows describes the sub-row, gate and settings; DEVELOPMENT_PLAN's pending-migration bullet names the file without marking it applied.
Sequencing: do the migration, types, schema, changeMode and harness column list as one commit-sized unit before any UI work; everything after the sub-row is additive at the read edge.

Tests as specced (lib cases now covered by tests/plates.test.ts against the shipped API; the remaining ones are integration):
- database.test.ts: plate preferences default to true, 'standard' and null after the profile trigger; the check rejects an unknown plate_set, a bar of 0 and of 600; a null bar update succeeds; running the migration twice does not throw.
- workflows.spec.ts spec 1: after filling bench set 1 with 82.5, the note "Barbell bench press set 1 plates" contains "25 · 5 · 1.25 per side · 20 kg bar".
- workflows.spec.ts spec 2: on Settings pick the microplate rack and a 15 kg bar, save, assert the toast and the bootstrap profile values (this is the regression test for the strict-schema fan-out); clear the bar and assert null; switch to lb and assert no plate note appears on a Machine exercise; restore kg and the default rack.
- workflows.spec.ts spec 3: the existing 390px overflow assertion already covers the sub-row.
- Not added: no test asserts minimal plate count, because greedy deliberately does not have that property.

Risks:
- A wrong plate list acted on physically under fatigue is worse than no calculator; the quantisation is the structural mitigation, and because the logic ships as one comment-free line the algorithm must be documented in ARCHITECTURE.md.
- The strict profile schema fan-out across three payload sites.
- Numeric-as-string on bar_weight_kg.
- Rack and display unit can legitimately diverge; the nullable bar and the "closest" reporting keep it honest without eliminating it.
- Coverage is partial: the equipment enum cannot distinguish plate-loaded from selectorised machines, so leg press is unreachable; a plate_loaded flag on exercises is the eventual fix and a much larger change.
- Scope creep into per-exercise bars (EZ, trap, safety squat, 15 kg women's bar): explicitly out; the global nullable bar is the stopping point.
- Vertical space: about 20px under every qualifying set; tap-to-expand was rejected.
- Queue jumping: it adds another unapplied migration; amend the pending list rather than quietly adding to the backlog.
- Greedy is not minimum-count; documented, not tested.

What shipped instead, and what the UI must respect:
- lib/plates.ts models an inventory, not a rack enum: PlateSetup is a bar, a side count and a list of plates each with an optional count (null means unlimited). Defaults come from plateSetup(unit, tracking): 20 kg or 45 lb bar with two sides for weighted movements, no bar and one side for bodyweight and assisted.
- loadPlates(target_kg, unit, setup) scales the display-unit target to integer thousandths and runs a bounded subset-sum over the inventory, so limited counts and fractional custom plates are supported and the result is the fewest plates with heavier plates preferred on ties. Statuses are exact, closest (ties go to the lighter total) and below_bar; a null, non-finite or negative target returns null. platesText renders "2×20 + 10 + 2.5". This supersedes the spec's greedy algorithm, its rack constants, its rackFor, plateFormat and plateSummary helpers, and its exact-by-construction property test.
- Cost grows with the target divided by the smallest plate, so clamp absurd targets at the call site before calling it.
- The profile columns landed in 202609080007_plates.sql (commit 9cd1fe7) as bar_weight_kg (null or 0 to 50) and a plates_kg text column for the inventory, which departs from the spec's plate_set enum and 500 kg bar cap; null means the unit defaults. The same commit made profileSchema partial so a PUT writes only the supplied columns, and Settings sends only changed fields, which removes the strict-schema fan-out hazard and the changeMode edit from the steps above. components/plates.tsx exports profilePlates, profileSetup, plateHintText and a PlateHint component (tests/plates-ui.test.ts).
- Remaining work: mount PlateHint under each qualifying set row on Today (Barbell, weighted, weight above zero, still as a full-width sibling inside the set table rather than a sixth grid cell) and as the row-footer hint in quick entry, add the CSS and the 480px override, clamp absurd targets before calling loadPlates, and extend the browser specs as described. The harness profile UPDATE column list must include the two new columns if it is still hardcoded.
- With inventory-aware loading, the approximate styling should key on status not being "exact" and can show the achieved total from total_kg.

Size: medium (UI remaining: small).

## rpe-rir

Summary. An optional per-set RPE from 1 to 10 in half points, stored as a nullable numeric(3,1) column on exercise_sets, and used in suggestNext as a step multiplier on an increase the existing mechanical rule already decided to make, never as an independent signal. RPE cannot flip a repeat into an increase and cannot deload. One helper effortStep returns 2 for RPE at most 7, 1 for 7 to 9, 0 above 9, and 1 for null. Because factor 1 is today's behaviour and factor 0 produces exactly the existing repeat object, null RPE reproduces today's output through all three tracking branches, and each branch gains two lines rather than tripling. The governing rating is the maximum over the sets at the top load of the previous session; lower-load sets and warm-ups never govern.

Scale decision: store canonical RPE only. No rpe_scale profile column, no RIR column, no settings UI, no percent-of-max table. RIR is taught only in the picker's accessible labels ("RPE 8, two reps in reserve"), mirroring canonical kg with a display unit.

Layout: grid-template-columns is not changed at any breakpoint and there is no sixth column. RPE lives on a full-width second grid line inside the set row, revealed by one per-exercise "Effort" toggle beside Use target and Copy previous, and auto-revealed for any set that already carries a rating. A logged rating also shows as a compact "@8.5" mark inside the reps cell, only when non-null, so the hot path for users who skip RPE is byte-identical to today.

Explainability: Suggestion gains effort (the governing rating) so the card can say why the number moved ("RPE 7 last time, double step" or "holding at RPE 9.5"). This breaks the existing deepEqual assertions in tests/progress.test.ts deliberately; re-deriving the governing set in the view would duplicate the engine.

Verified in PGlite on the real chain: the migration applies; 8.5 and null round-trip through save_workout; the check rejects 11 and 8.3 and the whole save rolls back with the prior row intact; the authenticated execute grant survives create or replace (which is exactly why history_points, whose return type would force drop and recreate, stays out of v1); the numeric arrives as the string '8.5'.

Explicit v1 non-goals: the lib/quick.ts token grammar, the quick-entry tap builder, RPE charting, history_points, and any RIR-scale preference.

Migration: yes. Add the column with a check (null, or between 1 and 10 with rpe×2 integral, exact on numeric), then create or replace save_workout with rpe added to the exercise_sets insert column list and (s->>'rpe')::numeric in the values. Produce it by slicing the function text out of schema.sql and patching two substrings (assert the insert statement occurs exactly once before replacing), never by retyping; nothing else in the body changes. No check tying rpe to completion, because the picker is reachable before a set is logged and completedSets already excludes drafts from the engine. Leave schema.sql alone; its copy goes stale exactly as save_template did. Deployment order on hosted projects: rep_ranges first, then this, in one window.

Files: the migration, lib/types.ts, lib/validation.ts, lib/progress.ts, lib/quick.ts, components/today.tsx, components/history.tsx, app/globals.css, tests/harness.ts, tests/progress.test.ts, tests/database.test.ts, tests/browser/workflows.spec.ts, docs.

Steps:
1. Migration as above.
2. lib/types.ts: SetLog gains rpe:number|null between reps and completed_at, required not optional, so tsc enumerates every construction site (instantiate, quickExercises, the picker seed and Add set on Today, tests). An optional field is what would let a stale rating ride along silently.
3. lib/validation.ts setSchema: rpe as a number from 1 to 10, refined to half points, nullable, default null, so the key stays optional on input (older payloads and the live spec's literal still validate) while the parsed output always carries it. No refine tying it to completed_at; validateLoads unchanged.
4. lib/progress.ts: topEffort(sets) (max of Number-coerced ratings or null) and effortStep(rpe) next to loadStep.
5. Suggestion gains effort:number|null. No third kind; a deload variant needs policy this app has no anchor for.
6. suggestNext, each branch: compute effort from the top-load sets, factor = hit ? effortStep(effort) : 1, kind is increase only when hit and factor greater than 0, and the load or reps move by step × factor. With null effort every branch reduces to today's expression; with factor 0 the object equals the existing miss branch. Report effort even when hit is false, for display only.
7. Draft hygiene, the silent-corruption path: copySets spreads, so it must set rpe:null explicitly; applySuggestion clears the rating on the unlogged working sets it rewrites; instantiate adds rpe:null.
8. lib/quick.ts: rpe:null in the quickExercises literal; do not touch the token grammar.
9. today.tsx save payload: add rpe to the enumerated set fields, or every rating is silently discarded on save while the UI shows it until reload.
10. today.tsx set construction: Add set must not copy the previous set's rating; the picker's three-set seed gets null.
11. Disclosure state in the Today component: a Set of entry ids toggled immutably, per exercise card.
12. The toggle: a Gauge-icon "Effort" button with aria-pressed in the previous-line actions.
13. The row: the "@8.5" mark beside the record flame in the reps cell when non-null, and after the remove button a set-effort div on the second grid line when the card is armed or the set has a rating.
14. The picker: chips for 6, 7, 7.5, 8, 8.5, 9, 9.5, 10 plus a clear chip; aria-pressed; labels "X set N effort RPE v, k reps in reserve" (or "no reps left" at 10); tapping the selected chip clears; changes go through update, mark dirty, and do not auto-save.
15. The reason clause after "Next:" built from suggestion.effort: "holding at RPE x" on a repeat with a load, "RPE x last time, double step" at 7 or below, otherwise "RPE x last time". Copy must never imply the rating caused a repeat that the reps already caused.
16. history.tsx: an Effort column in the per-session detail table, dash when null.
17. globals.css: set-effort on grid-column 1/-1 wrapping chips of at least 44px; rpe-chip with the ember selected treatment; rpe-mark. No change to any set-grid column rule.
18. tests/harness.ts: add the migration; coerce rpe with Number in the workouts mapper; optionally seed a rating on one completed week for the browser assertion.
19. tests/database.test.ts: add 0005 and this migration; confirm the routine subtest still passes; add the subtest below.
20. tests/progress.test.ts: add rpe to the set factory; update the existing deepEqual assertions with effort:null; add the new tests.
21. Gates, then docs: exercise_sets bullet, the Progression rule verbatim, the UI description, a checked plan line, and the pending-migration bullet.

Tests:
- progress.test.ts: on a 3x80x8 baseline, unrated gives increase to 82.5 with effort null (byte-identical to today plus the field); rated 7 gives 85; 8.5 gives 82.5; 9.5 and 10 give repeat at 80; mixed 7 and 9.5 gives repeat (max governs); the 7 versus 7.5 and 9 versus 9.5 boundaries.
- Isolation: missed reps with every set rated 7 still repeat (a good rating cannot manufacture an increase); a warm-up rated 10 is ignored; a lower-load set rated 10 is ignored and effort reflects top-load sets only; a draft rated 10 is ignored.
- Bodyweight 10 and 12 reps: unrated gives 13, rated 7 gives 14, rated 9.5 repeats at 12. Assisted 2x30x8: unrated 27.5, rated 7 gives 25, rated 10 repeats; the floor clamps at 0.
- Drafts never inherit: copySets nulls a 9; applySuggestion nulls the unlogged working set while the completed set keeps 8 and the warm-up keeps 10; instantiate seeds null.
- Validation: accepts 8.5, null and an omitted key (parsed to null); rejects 10.5, 0.5 and 8.3.
- database.test.ts: 8.5 round-trips, null round-trips, 11 and 8.3 reject with the prior row intact, the authenticated execute privilege on save_workout is still true; history_points still returns the same weight and volume.
- workflows.spec.ts: arm Effort, pick the 9.5 chip, save, reload, assert the chip is pressed and the "@9.5" mark is present, and the Next line holds; the phone overflow check passes with the picker open.
- Manual: at 480px and 360px chips wrap rather than shrink, and the closed state is pixel-identical to main.

Risks:
- The migration restates about 3 KB of single-line plpgsql; a one-character slip breaks all workout saving. Generate by slicing, land the round-trip test in the same change.
- The copySets spread path is the one tsc cannot catch; the explicit null and its test are the only defence. Do not weaken the field to optional.
- Sparse subjective input: the multiplier-only design bounds it to 0, 1 or 2 steps the rule already earned, and the reason clause makes deviations legible.
- Sequencing debt: suggestNext derived its rep target from the previous session's first set and ignored target_reps_max, so an 8-12 routine bumped weight immediately and never climbed to 12. Since the audit, commit cbc6c1e surfaces the range as the guided target; verify whether suggestNext now climbs the range before adding a second signal to the engine, and ship rep-range targeting first if it still does not.
- Two unapplied schema changes stack on hosted projects; apply rep_ranges first.
- Numeric-as-string on rpe.
- Scope creep into history_points would drop the grant.
- Per-set friction on the hot path; the closed state must not drift from the canvas-reconciled set row.
- Suggestion's new field breaks existing assertions by design; quick.tsx reads suggestNext by field and is unaffected.
- Deferred by design: an "@8" token in lib/quick.ts (the grammar already overloads "x" and numeric arity), RPE in the tap builder, an RIR preference, and any percent-of-max table.

Size: medium.

## exercise-substitution (killed)

The plan kills this feature; the spec is kept so the reasoning is not lost if it is revisited. Since the audit the catalog grew to 150 and the picker gained the equipment filter this spec asked for, so item 3 below is done and the remaining value is smaller still.

Summary. Swap-in-place plus honest, narrow suggestions with no schema change:
1. A Replace action in the Today exercise-actions row that swaps a movement while preserving the entry id, position, rest, notes and drafted sets.
2. A suggestion function that refuses to guess on the current seven-value taxonomy: candidates must share the muscle group, share the tracking type, and share at least one movement token in the name after equipment and modifier words are stripped. "Barbell curl" suggests Dumbbell, Hammer and Cable curl, never Skull crusher; "Back squat" suggests Goblet, Bulgarian split and Front squat, never a calf raise. When nothing shares a token, no suggestion block renders and the user gets the picker.
3. An equipment dropdown in the picker (done since the audit).
4. The movement_pattern taxonomy migration with a full reseed is deferred to a separate, explicitly approved deliverable.

Integrity rule for logged sets: a swap never reattributes completed sets. With no completed sets the entry is replaced in place. With completed sets the entry splits: the original keeps only its completed sets (so history and records stay correct) and the substitute is inserted immediately after, inheriting notes, rest and the drafted sets. save_workout deletes and reinserts every entry, so both shapes go through the existing PUT.

Safety rule: candidates are hard-filtered to the same tracking type because suggestNext direction, recordFor, bestBefore, isRecord, validateLoads and the validate_set_load trigger all branch on it. If the user browses to a different tracking type anyway, swapEntry nulls carried weights when the target is bodyweight so neither validator trips.

Migration: none. Deferred sketch if the taxonomy is approved: an exercises.movement_pattern column defaulting to 'other' with a check over seventeen patterns, id-driven updates so every seed lands in exactly one, plus exerciseSchema, the Exercise type, an ExerciseForm select, promotion of the pattern above the token score, a database test that no seed remains 'other', and the harness loader line. It changes no RPC.

Files: lib/substitutes.ts (new), components/library.tsx, components/today.tsx, tests/substitutes.test.ts (new), tests/browser/workflows.spec.ts, docs.

Steps:
1. lib/substitutes.ts: a genericWords set (equipment, grip, position and joining words); movementTokens(name); substituteScore (shared tokens × 10, plus 3 when equipment differs, 0 when nothing is shared); substitutesFor(exercises, base, {exclude, limit}) hard-filtering archived, excluded, self, muscle group and tracking type, keeping only positive scores, sorted by score then name, limit 6; blankSets; and swapEntry implementing the in-place and split shapes with contiguous positions. Do not weaken: muscle and tracking are hard filters; score 0 emits nothing; different equipment is only a tiebreak, never a substitute for a shared token; the in-place branch keeps the entry id and only the split mints a new one.
2. ExercisePicker gains title, initialMuscle and suggested props (all defaulted so Routines and Add exercise are unchanged), an equipment select, and a "Closest alternatives" list above the main list when suggestions exist and the search is empty, reusing existing picker CSS.
3. today.tsx: ArrowLeftRight import, swap state holding the entry id, derived swapping entry and its exercise, a Replace icon button between Move down and Remove, and a second picker mount titled "Replace X" pre-filtered to the muscle group with suggestions, excluding every exercise already in the workout. On choose: refuse when the split would exceed 50 exercises; confirm when logged sets exist, explaining they stay on the original; update through swapEntry; notify.
4. Nothing else changes by construction: save re-indexes positions, the bests effect refetches for the new exercise id, previous and suggestion re-derive, and the routine target badge disappears for a movement the routine did not prescribe (intended; document it).

Tests: parse the seed catalog from the migrations and assert the count first; tokens ignore equipment and position words; bench press is answered with presses and never flyes; curls never get triceps work; squats never get calf raises or adduction; suggestions never cross tracking type; a token-isolated movement returns nothing; excluded, archived and self are dropped; different equipment ranks as a tiebreak only; swap without logged sets is in place; swap with logged sets splits without reattribution and with unique set ids; swapping into bodyweight clears drafted weight; the swapped workout passes workoutSchema and validateLoads. Browser: replace bench with dumbbell bench in place, log a set, replace again accepting the confirm dialog (register the dialog handler first or Playwright auto-dismisses it), save, and assert through bootstrap that the logged set stayed on the first exercise and the substitute follows at position 1 with two drafts.

Risks: under-suggestion is the deliberate trade (leg press is not offered for squat); terse custom names degrade to silence, not wrongness; the split adds an entry against the 50 cap; the routine badge is lost on swap; relaxing the tracking filter breaks three things at once; the taxonomy drags in a reseed and permanently couples the seed migration to an advice-quality bar.

Size: medium.

## weekly-recap (killed)

The plan kills this feature because Progress and Today already render the weekly numbers and the only new content was a streak, which the design constraints forbid. The spec is kept for its two reusable pieces: the week_records RPC design and the volumeTrend helper.

Summary. One panel on Progress, not Today, headed "Week of Monday to Sunday" and reporting the last completed week only, carrying exactly what is new: a roll-up of all-time bests set during that week computed server-side by a new week_records(week_start) RPC so it can never label a 12-week-window best as a PR, an eight-week volume trend from a pure volumeTrend in lib/progress.ts, and a one-line sessions, sets and volume summary with a delta against the week before.

Decisions:
- The streak is dropped: DESIGN_BRIEF.md lists "no gamification, no streak badges, no confetti" as a hard constraint, and the 84-day bootstrap window would silently cap it at twelve weeks.
- Sets per muscle is not re-rendered; the existing Progress table remains the single rendering, pinned by a Playwright assertion.
- No dismissal and no Monday gate: the panel always shows, which sidesteps browser storage, a profile column with its strict-schema fan-out, and a date-gated spec that flakes without a frozen clock.
- Home is Progress because the redesign deliberately moved chrome off Today.
- Scope honesty: the roadmap line was "weekly summary email or Monday recap card"; this ships only the card, and the email, which is the half that drives a return visit, stays unchecked.
- Zero mutations: a read-only security-invoker RPC that RLS constrains independently.

Migration: yes, a new function only, nothing replaced. week_records(week_start) returns exercise id, name, tracking type, value, previous and the date; a valid CTE of the caller's completed working sets keyed on weight for weighted and reps for bodyweight (assisted excluded on purpose, matching Today which never flags assisted PRs); this_week takes the per-exercise maximum inside the seven days and the date it was set; before_week takes the maximum strictly before the week; the result keeps rows where there is no earlier best or the week's best exceeds it. Every column reference is table-qualified because OUT-parameter names are in scope in a SQL-language table function. It carries its own revoke and grant. No new index: the scan shape matches history_points.

Files: the migration, lib/types.ts, lib/progress.ts, lib/data.ts, the API route, components/history.tsx, tests/progress.test.ts, tests/database.test.ts, tests/harness.ts, tests/browser/workflows.spec.ts, docs including SETUP.md.

Steps:
1. Migration.
2. Prove the RPC in tests/database.test.ts before any UI (loader line plus the subtest below).
3. Types: RecapRecord and Recap.
4. lib/progress.ts: name the weekSummary return type WeekSummary and add volumeTrend(workouts, exercises, date, weeks=8) returning oldest-first and ending on the week containing the date; callers pass a week back to end on the last completed week; eight weeks sits inside the 84-day window.
5. Unit test volumeTrend.
6. lib/data.ts weeklyRecap: derive the week server-side from the stored profile timezone, page the RPC through allRows, coerce numerics.
7. Route: a GET recap branch between bootstrap and history, no query parameters, so no schema and no origin-check involvement; respond already sets private no-store.
8. history.tsx: a WeekVolume SVG bar chart labelled "Weekly training volume ..." with the last bar in ember, and a WeeklyRecap client component fetching the recap with the cancelled-flag and retry pattern from the bests effect, rendering the week line, the chart, a "Bests set that week" table (trophy, linked exercise name, best, previous or dash, date) or an empty note, and a caption stating completed working sets only, Monday weeks, bests measured against every earlier session, assisted excluded. Rendered between the stats grid and the muscle table with nothing else changed.
9. Harness: the loader line, a deterministic "Deadlift day" seed in the last completed week with no earlier history so the row appears on every weekday, and a recap branch coercing numeric and date values.
10. Browser: extend the second spec before the unit block.
11. Docs: the endpoint line, SETUP's migration list, the plan line rewritten with the streak decision, a new unchecked email line, the pending-migration bullet.
12. Gates, and tick nothing until all five pass.

Tests: volumeTrend covers whole weeks, ends on the requested week, is zero for untrained weeks, and equals weekSummary for its last entry; week_records reports only all-time bests (105 beating an earlier 100, a first-ever session with null previous, a 40 kg week that does not beat a 60 kg history is absent, bodyweight by reps, assisted absent, another user sees zero rows, an empty week returns nothing); the browser spec expects the "Week of" heading, the volume chart, the Deadlift row and the surviving "Sets by muscle group" heading.

Risks: false PRs are the main trap and the reason for the RPC (bootstrap only knows 12 weeks); assisted silence is deliberate and stated in the caption; duplication if the panel grows; a project missing the migration degrades to the panel's own error state while the rest of Progress works; one extra request per Progress mount, deliberately not folded into bootstrap; the card alone does not close the retention story; it sat behind higher-value Phase 1 work.

Size: medium.

## program-library

The plan rescopes this to a content task: hand-encode four to six canonical splits as share payloads and render them in the Routines empty state; do not build the browsable ProgramBrowser system below. The spec is kept because its program data, its resolver refactor and its resumable multi-write pattern are what the content task reuses.

Summary. "Starter programs": a static lib/programs.ts of four honest, weight-agnostic splits (Push Pull Legs six-day, Upper / Lower four-day, Full body 5 × 5 two-day, Beginner linear progression two-day) resolved through the existing share decoder and saved as N routines via the existing idempotent PUT /api/templates/:id. Drop 5/3/1, nSuns and GZCLP entirely: TemplateExercise stores only sets, reps, an optional ceiling and rest, instantiate writes weight_kg null, and suggestNext is one global double-progression engine, so those programs would be famous names on generic rep counts and the app would advise an increase in a week the real program deloads. Also drop "Starting Strength" and "StrongLifts" branding (enforced marks). No migration, no new API branch, no RLS change. The only non-obvious engineering is the multi-write path: two to six sequential PUTs made resumable by holding stable per-day template and mutation ids in a ref, because save_template checks last_mutation_id before the revision check and returns the existing revision, so a retry re-sending revision 0 for an already-saved day is a no-op. Partial failure is reported honestly ("Added 2 of 6 routines.") with a Retry that resumes.

Since the audit: Power clean and Sumo deadlift now exist in the catalog (202609080006_catalog.sql), so the optional seed migration in the spec is moot and its proposed ids 084 and 085 are taken. The seed test must parse both migrations and assert 150.

Migration: none by design. Weights, percent of training max, AMRAP flags, cycle state and a batch save_program RPC are all explicitly out.

Files: lib/programs.ts (new), lib/share.ts, components/routines.tsx, tests/programs.test.ts (new), tests/share.test.ts, tests/database.test.ts, tests/browser/workflows.spec.ts, docs.

Steps:
1. lib/share.ts: export the shared-routine zod schema as sharedRoutine, add a SharedEntry type, and extract the resolution after safeParse into resolveShared(payload, exercises, id) so decodeRoutine becomes decode, parse, resolve. Do not change seed-id-then-name matching, the ceiling-greater-than-reps rule or the missing semantics; share.test.ts must pass untouched.
2. lib/programs.ts: Program and ProgramDay types; every entry carries both the seed UUID and the exact seed name so id match is primary and name match is the fallback. The four programs, with sets × reps or range and rest seconds:
   - Push Pull Legs (Intermediate, six days). Push A: bench 4x5 180, overhead press 3x6-8 150, incline dumbbell press 3x8-12 90, lateral raise 3x12-15 60, triceps pushdown 3x10-12 60. Pull A: deadlift 3x5 210, barbell row 4x6-8 120, lat pulldown 3x8-12 90, face pull 3x12-15 60, barbell curl 3x8-12 60. Legs A: back squat 4x5 180, Romanian deadlift 3x8-10 120, leg press 3x10-12 90, lying leg curl 3x10-12 60, standing calf raise 4x12-15 60. Push B: incline barbell bench 4x8-10 120, dumbbell shoulder press 3x8-12 90, dip 3x8-12 90, cable chest fly 3x12-15 60, overhead triceps extension 3x10-12 60. Pull B: pull-up 4x6-10 120, seated cable row 4x8-12 90, one-arm dumbbell row 3x10-12 75, reverse dumbbell fly 3x12-15 60, hammer curl 3x10-12 60. Legs B: front squat 4x6-8 150, barbell hip thrust 3x8-12 90, Bulgarian split squat 3x10-12 75, seated leg curl 3x12-15 60, hanging knee raise 3x10-15 60.
   - Upper / Lower (Intermediate, four days). Upper A: bench 4x5 180, barbell row 4x6-8 120, overhead press 3x6-8 120, lat pulldown 3x8-12 90, barbell curl 3x8-12 60. Lower A: back squat 4x5 180, Romanian deadlift 3x8-10 120, leg press 3x10-12 90, lying leg curl 3x10-12 60, standing calf raise 4x12-15 60. Upper B: incline barbell bench 4x8-10 120, seated cable row 4x8-12 90, dumbbell shoulder press 3x10-12 75, face pull 3x12-15 60, triceps pushdown 3x10-12 60. Lower B: deadlift 3x5 210, front squat 3x8-10 120, Bulgarian split squat 3x10-12 75, seated leg curl 3x12-15 60, hanging knee raise 3x10-15 60.
   - Full body 5 × 5 (Beginner, three days alternating). A: back squat, bench, barbell row, each 5x5 180. B: back squat 5x5, overhead press 5x5, deadlift 1x5 240. Run A, B, A one week and B, A, B the next.
   - Beginner linear progression (Beginner, three days alternating). A: squat 3x5, bench 3x5, deadlift 1x5 240. B: squat 3x5, overhead press 3x5, barbell row 3x5.
   Set the ceiling only where a range is listed and always above the target. No brand names anywhere.
3. resolveProgram(program, exercises, id): resolves each day through resolveShared with the name "Program · Day", deduplicates missing names, and drops days that resolve to zero exercises because templateSchema and save_template require at least one. The " · Day" naming keeps cards grouped (bootstrap orders by created_at, so save in day order) and keeps sessionTargets working because it matches on template name equals workout title.
4. tests/programs.test.ts before any UI, because a transposed UUID is invisible in review.
5. ProgramBrowser in routines.tsx: picked program, busy, done count, error, and a plan ref of per-day template plus mutation id rebuilt only when the picked program changes; resolve with useMemo so re-renders do not mint new UUIDs.
6. The save loop: sequential PUTs from the done index, a local saved counter for the error message (the state is stale inside the catch), refresh and notify on completion, "Added N of M routines" with Retry on failure.
7. Markup: a list view of programs (sessions, cadence, level) and a preview view with the summary, a missing-exercises alert, a duplicate-name note, the per-day routine cards using existing card CSS, Back, and a primary button with a fixed aria-label "Add these routines" whose text cycles through Add, Adding N of M and Retry.
8. Wiring: a "Browse programs" button before Import in the heading and in the empty state next to "Create your first routine", which is the cold-start moment the feature exists for.
9. tests/share.test.ts: resolveShared and decodeRoutine agree on the fixture.
10. tests/database.test.ts: add 0005 to the chain (a latent gap: the suite exercises the pre-rep-range save_template) and add the idempotency subtest.
11. Browser: extend the second spec with the two-day program and a forced mid-import failure.
12. Docs: the Phase 2 line rewritten with what shipped, a Deferred bullet recording why percentage programs were cut (they need a percent-of-training-max field, a per-exercise training max with its own policies and a per-routine progression policy, a Phase 1 model change to justify on its own merits, plus trademark exposure), and the ARCHITECTURE sentence about programs reusing the share resolver with stable per-session mutation ids.
13. Verify; lib/programs.ts should land near 8 KB on one line.

Tests: the seed parse asserts the catalog count first; every entry's id exists and its name matches (name equality is the point); every day parses under sharedRoutine with any ceiling above its target; a resolved program has no missing names, one template per day, contiguous positions, revision 0, and passes templateSchema with a mutation id; program ids, day names and template names are unique and at most 100 characters; an incomplete library reports the name once and a day with nothing resolvable yields no template; resolveShared equals decodeRoutine; save_template returns the same revision for a repeated mutation id without duplicate rows and rejects a stale revision with a new mutation id; the browser flow adds the two-day program with the second PUT forced to fail once, expects "Added 1 of 2 routines.", retries, expects the success toast, and asserts through bootstrap exactly two templates with the expected day contents and no duplicates.

Risks: naming is the whole credibility risk (ship generic names only); partial import is real and must stay visible, never masked by a success toast, and atomicity would mean a new RPC; re-importing creates duplicate routines because there is no unique name constraint, covered by the preview note and deliberately not blocked; instantiate seeded only the bottom of a range (since the audit, cbc6c1e carries the range through as the target); rep_ranges must be applied to any hosted project before importing, or every ranged day fails at the RPC; this is an activation feature, so the empty-state button matters as much as the header; the two beginner programs are close cousins and the linear one is the cheapest cut; the single-line data blob makes the id-and-name test load-bearing.

Size: medium as specced; small as the content task the plan actually asks for.
