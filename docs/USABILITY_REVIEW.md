# Forge usability review

Reviewed 12 September 2026 against the current local source.

The user records the whole workout **after training**. That is the primary workflow for these recommendations. They are a proposed direction, not implemented changes.

## Main finding

Forge needs a dependable, short path from remembering a workout to finding that same record later. Existing entry modes, routines, history, and progress do not yet form that path. The visual design has a consistent identity, but reducing colors or borders alone will not remove the repeated decisions and data entry.

The product promise should be: **Record what you did, change only what differed, and find it again whenever you need it.**

## What to fix first

| Priority | Current behavior and evidence | Recommended change |
| --- | --- | --- |
| 1 | Quick entry has its own empty rows and saved-session ref. Today loads a selected workout into a separate `current` state, but passes only the date and dirty callback to QuickEntry. Opening an old workout or starting a routine therefore does not hydrate the quick editor. New workout also changes only the guided state. See [today.tsx](../components/today.tsx), [quick.tsx](../components/quick.tsx), and [routines.tsx](../components/routines.tsx). | Give every entry view the same selected workout and draft. New, open, edit, resume, and repeat must have consistent meanings. After saving, keep the saved workout visibly open and editable. |
| 1 | Quick entry filters to `ready` lines, enables saving whenever at least one line is ready, and submits only those lines. It shows an aggregate error count, but still reports a successful save while omitting incomplete entries. Dirty state also ignores incomplete rows. See [quick.tsx](../components/quick.tsx). | Preserve every entered row. Show errors beside the affected fields and block final completion until they are fixed or explicitly removed. Save incomplete drafts without presenting them as completed workouts. |
| 1 | Unsaved workouts live in component state/refs. Navigation warnings and retry behavior do not provide recovery after closing or reloading the app. Only the rest timer has device persistence. See [today.tsx](../components/today.tsx), [quick.tsx](../components/quick.tsx), [provider.tsx](../components/provider.tsx), and [shell.tsx](../components/shell.tsx). | Retain an account-scoped draft on the device and synchronize it safely. Distinguish a retained draft from a completed workout and distinguish device storage from server synchronization. Preserve existing mutation IDs and revision protections. |
| 2 | There is no whole-workout Repeat action. Guided Copy previous works one exercise at a time; routines instantiate with blank weights. Quick entry requires adding exercises individually, always defaults to three sets, and prefills a suggested next load. See [today.tsx](../components/today.tsx), [quick.tsx](../components/quick.tsx), and [progress.ts](../lib/progress.ts). | Make Repeat last workout the main start action, with recent workouts and routines as alternatives. Copy the actual prior exercises, set counts, weights, and reps into an explicitly unconfirmed draft. For retrospective recording, do not silently increase weights. |
| 2 | Each quick-entry exercise row repeats one weight/reps pair across all sets. Different performance such as `80 × 8, 80 × 7, 75 × 8` requires duplicate exercise rows or typed syntax. Switching between row and text entry uses separate state. See [quick.tsx](../components/quick.tsx). | Show a compact summary when sets match, with Edit sets to expand individual weights and reps. Keep one underlying record when switching representations. |
| 2 | The bootstrap loads only the last 84 days of history, and the journal prevents browsing earlier months. Older data can still exist in PostgreSQL and the exercise-specific history endpoint supports all time, so this is an access gap rather than automatic deletion. See [data.ts](../lib/data.ts) and [journal.tsx](../components/journal.tsx). | Load older months or sessions on demand. Make a chronological workout list the default, with a calendar as an alternate view. Include incomplete drafts with an explicit status. |
| 2 | Progress computes recent records and exercise bests from the same 84-day subset. An older, higher lift can disappear from the baseline, making a lower recent lift appear to be a new record. See [history.tsx](../components/history.tsx) and [progress.ts](../lib/progress.ts). | Compute lifetime bests independently of the visible chart range. Label all time-bounded comparisons explicitly. |
| 3 | The home screen combines date navigation, weekly statistics, calendar, mode selection, workout controls, exercise suggestions, and entry fields. There are five main navigation destinations. See [today.tsx](../components/today.tsx) and [shell.tsx](../components/shell.tsx). | Put the workout being recorded first. Make the primary destinations Log, History, and Progress. Reach routines from Repeat/Choose routine and exercise management from the picker or settings. Keep the guided training view available as a secondary option. |

## A smaller experience for recording after training

Opening the app should show a recoverable draft if one exists. Otherwise, show Repeat last workout, a short recent-workout list, and Start blank. First-time users can choose exercises immediately without creating a routine first.

Repeating a workout creates a new draft for the chosen date, with the previous actual values visible. It must not mark those values as completed until the user saves the workout. Selecting an older log edits that log; it must never silently create a replacement or duplicate.

An example of the proposed editor:

```text
Log workout                         Today
Upper body                         Change

Bench press             80 kg × 8 × 3   Edit
Cable row               60 kg × 10 × 3  Edit
Pull-up                 8, 7, 6 reps    Edit

+ Add exercise
Add note

Draft saved on this device
                  [ Save workout ]
```

This is a layout proposal, not an implemented or tested screen. The draft status must describe actual storage behavior.

Use one compact row per exercise, large editable numbers, and details that expand only when needed. Support selecting several exercises in one picker visit and creating a missing exercise there. Keep workout title, date corrections, notes, and optional warmups available without requiring them for every entry. Rest controls, progression suggestions, and per-set completion checks belong in the optional during-training view.

After Save workout, show the saved date and exercises with Edit and Repeat actions. History should show recognizable workout titles and the actual sets, not just volume statistics. A user should be able to open any saved workout in the same editor regardless of their preferred entry style.

## Delivery order

1. Repair shared workout identity, reopening, incomplete-input handling, and draft recovery. These determine whether the log can be trusted.
2. Add whole-workout repeat using actual previous values, plus compact rows with individual-set editing. Make this the default path for this user.
3. Simplify home/navigation and enable complete historical browsing. Correct lifetime record calculations before relying on record feedback.
4. Use it for real workouts for two weeks. Record where logging stalls before expanding the feature list.

Retain the existing transactional save, ownership, retry, and revision checks. They are useful foundations. Review the hosted setup separately: the repository documents outstanding deployment/live-service checks, but their current external status was not verified in this review.

## Proposed success checks

These are initial product targets to validate, not measured results:

- A returning user records a familiar five-exercise workout in about a minute by changing only the differences.
- No exercise search is required to repeat a recent workout.
- Unequal sets, such as `80 × 8, 80 × 7, 75 × 8`, remain one exercise and reopen exactly as saved.
- One valid row plus one incomplete row cannot produce a misleading completed-workout confirmation.
- Closing and reopening restores an unfinished draft, including incomplete fields.
- Opening History while Quick entry is preferred displays the selected saved workout.
- New workout after a quick save creates a distinct workout; the previous session remains unchanged.
- Starting a routine works regardless of the preferred entry style.
- Workouts older than 12 weeks are discoverable and editable, and lifetime bests remain correct when the visible range changes.

## Review scope and validation

This assessment uses current source, existing workflow tests, project documentation, and independent reviews of logging and history. `npm run test` passed all **127 tests** during the review. Application code was not changed.

The current tests do not establish that all the journeys above work. For example, quick-entry browser coverage checks a single invalid row, but not a mixture of valid and invalid rows. The test harness loads all history while production bootstrap restricts it; its navigation also remounts the provider. This can conceal production history-range and cache-continuity problems.

Live browser interactions, real Supabase authentication, hosted migrations, and deployment were not verified. The local test-app startup encountered an occupied port after a sandbox-related initial failure; no existing server was stopped or repurposed.
