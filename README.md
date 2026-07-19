# TestRepo

Sandbox for verifying that Claude Code cloud routines fire on GitHub events.

## Purpose

Event-based routines (`commits pushed`, `pull request opened`) require the Claude
GitHub App to be installed on the owning org. Routines on `Zizou-PM/dentsup` never
fired because that account only has contributor access — the app could not be
installed there. This repo exists to confirm the trigger works when the app *is*
installed.

## Test procedure

1. Initial commit — makes the repo non-empty so it appears in the routine repo picker
2. Install the Claude GitHub App on `TestOrgAbdi`
3. Create an event routine (`commits pushed`) pointing at this repo
4. Push a second commit — this is the actual trigger test

Step 4 should fire the routine with no further manual action.
