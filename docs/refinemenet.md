# Forge: a workout tracker your brother will keep using

## 1. Goal and direction

Make Forge useful for a regular lifter who records workouts **after training on an Android phone**.

The core experience should be: **Choose a previous workout → change what you did differently → save → see your progress.**

The first release is for you and your brother. Keep the existing visual style and focus on speed, reliable records, and understandable progress.

This plan is based on the current source and your answers. All **129 unit and database tests passed**. Visual browser testing and hosted authentication remain unverified.

## 2. Problems to fix first

| Current issue | Planned correction |
|---|---|
| Calendar’s **Repeat** immediately saves a completed workout; Log’s **Repeat** creates an editable draft. | Every Repeat action opens an unfinished draft. Nothing counts as completed until Save succeeds. |
| Changing the set count or shared fields can overwrite individually edited sets. | Preserve different weights and reps. Adding a set copies only the final set; removing one leaves the others unchanged. |
| Unfinished entries live only in memory, although offline wording implies they are safe. | Add recoverable phone drafts and accurate save-status messages. |
| The compact weight input displays stored kilograms when pounds are selected, and its stepper mixes units. | Convert values consistently for display and entry; keep stored weights in kilograms. |
| Adding an exercise can prefill an increased suggested weight. | Prefill actual previous results. Recording a completed workout must not silently suggest that he lifted more. |
| Progress is outside the main navigation. Some record summaries use only recent history, and assisted-exercise trend wording can reverse the meaning of improvement. | Make Progress a main destination, label comparison periods, and treat lower assistance at matching reps as improvement. |
| Unfinished server workouts can appear as “Saved” summaries. | Distinguish **Draft**, **Saving**, and **Saved workout** throughout the app. |

## 3. The interface to build

### Log

- Show **Resume draft** first when unfinished work exists.
- Otherwise emphasize **Repeat last workout**, with a compact recent-workout chooser for alternating routines. Keep **Use routine** and **Start blank** as secondary actions.
- Show one compact entry per exercise. Automatically expand individual sets when their values differ; provide an explicit **Apply to all sets** action for intentional bulk changes.
- Show previous actual results beside the inputs. Keep exercise history accessible without repeating several statistics and links inside every entry.
- Allow selecting multiple exercises in one picker visit. Retain the existing custom-exercise option.
- Keep **Save workout** reachable in a mobile bottom action bar, clear of navigation and the keyboard.
- After saving, show the recorded workout with **Edit**, **Repeat**, and **Save as routine** actions.

### History

- Replace the Calendar tab with **History**, opening a newest-first workout list. Keep Calendar as an alternate view.
- Show workout date, title, exercise names, and set summaries. Open every workout in the same editor.
- Load complete months on demand, including older workouts; remove the arbitrary 2020 navigation limit and fix partially loaded boundary months.
- Use “No workout logged” for empty days. Do not present the time of retrospective data entry as the workout’s actual duration.

### Progress

- Promote the existing progress functionality into the third main tab.
- Start with recently trained exercises showing their latest result and previous-session comparison.
- Open the existing exercise chart and set history from each exercise.
- Keep lifetime records on exercise detail, using the existing all-time history request.
- Remove the current global “Personal bests” count from this release because its recent-history baseline can misidentify records. Show both weight and reps rather than treating every heavier set as proof of improvement.

### Simplify secondary features

- Reach routines from Log and exercise management from the picker or Settings. Preserve their existing URLs and sharing links.
- Remove the floating rest timer and automatic-timer preference from this after-training experience.
- Leave plate calculators, estimated-max tables, muscle-distribution charts, and training schedules outside the main flow.
- Defer social feeds, leaderboards, AI coaching, nutrition tracking, and a large program library.

## 4. Implementation and release order

1. **Make recording dependable.** Use one shared draft model for blank, repeat, routine, resume, and edit flows. Preserve set types, notes, individual values, and original workout identity. Repeating generates fresh IDs and never silently omits unavailable exercises.
2. **Add phone draft recovery.** Store versioned, account-specific drafts in IndexedDB, including incomplete fields and pending save information. Clear them after confirmed saving, explicit discard, or sign-out. Report storage failures honestly. This provides recovery when the app loads again; full offline launching and background synchronization are deferred.
3. **Repair saving and conflicts.** Reuse the existing workout API and revision protections. Retry an identical pending request with the same mutation ID and payload. Preserve the draft when another device has changed the record; offer reload or save as a separate workout.
4. **Simplify navigation and screens.** Implement Log / History / Progress, the compact editor, and complete month loading. Reuse existing workout-range, exercise-history, routine, and profile endpoints; no new database schema is planned.
5. **Prepare the brother trial.** Offer a short first-use path: choose units, confirm timezone, then log or import an existing routine. Complete Android home-screen icons and manifest support. Verify the hosted migration state, HTTPS access, email confirmation, password recovery, and two-account privacy before sharing the app link. The health endpoint’s hard-coded migration count is not proof that migrations were applied.
6. **Update project documentation.** Replace conflicting older product guidance with this agreed workflow while preserving current uncommitted work.

## 5. Acceptance checks and trial

Before sharing:

- Repeat from any screen creates an editable draft and adds nothing to completed statistics.
- Unequal sets survive editing, changing set count, saving, and reopening.
- Reload restores incomplete entries; signing into another account cannot reveal them.
- Failed saves, lost responses, double taps, and version conflicts neither duplicate workouts nor discard edits.
- Kilograms, pounds, bodyweight, assistance, and existing warmup sets retain correct values.
- Older workouts remain discoverable; loading failures never masquerade as empty history.
- Android layouts work at 360–430px widths with readable numbers, accessible labels, and at least 44px touch targets.
- Run formatting, type checks, production build, unit/database tests, browser workflows, and live authentication checks. Match the test harness’s history window to production so it cannot conceal missing-history bugs.

Then run a **two-week trial**. Initial success targets are a familiar five-exercise workout recorded in about a minute, no lost or duplicated entries, and your brother finding his previous result without help. Ask what slowed him down and what he ignored; use those observations to choose the next feature.
