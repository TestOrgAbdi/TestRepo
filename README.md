# TestRepo

Sandbox for verifying that Claude Code cloud routines fire on GitHub events.

## Background

Routines on `Zizou-PM/dentsup` never fired because that account only has contributor
access, so the Claude GitHub App could not be installed — and without the App there is
no webhook delivery. This repo lives in an org where the App *is* installed, to confirm
the trigger works when that prerequisite is met.

## Supported events

Routines subscribe to two GitHub event categories only:

| Event | Fires when |
| --- | --- |
| Pull request | A PR is opened, closed, assigned, labeled, synchronized, or otherwise updated |
| Release | A release is created, published, edited, or deleted |

There is no raw "commits pushed" event. A plain `git push` fires nothing; commits only
reach a routine as `pull_request.synchronize`, when they land on a branch that already
has an open PR.

## Test procedure

1. Initial commit — makes the repo non-empty so it appears in the routine repo picker
2. Install the Claude GitHub App on `TestOrgAbdi`
3. Create a routine with a Pull request trigger set to *all actions*
4. Open a PR — fires `pull_request.opened`
5. Push another commit to the same branch — fires `pull_request.synchronize`

Steps 4 and 5 should each start their own session with no further manual action.

## Results

| Event | Fired | Notes |
| --- | --- | --- |
| `pull_request.opened` | pending | PR #1 opened at 16:50Z |
| `pull_request.synchronize` | pending | second commit pushed to the same branch |

A routine only reacts to the actions its trigger subscribes to. If the trigger is set to
`opened` alone, pushing further commits fires nothing — select *all actions* on the Pull
request category to catch both.
