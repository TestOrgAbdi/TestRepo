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

| Path | Outcome |
| --- | --- |
| `pull_request.opened` (PR #1, 16:50Z) | no session, no comment after 20 min |
| `pull_request.synchronize` (16:57Z) | no session, no comment after 14 min |
| Manual run via API (17:09Z) | comment posted in 40 seconds |

Same routine, same prompt, same repo, same tools — only the entry path differed. The
routine, its prompt, and repository access all work; webhook delivery is what fails.

Root cause: the GitHub trigger will not save on the routine. The UI returns *"Routine
saved, but the GitHub trigger couldn't be updated."* So the events had no listener, which
is indistinguishable from a broken webhook until you compare against a manual run.

Two open leads: the org's OAuth authorization for Claude may be missing (separate from
installing the App), and every routine on this account was created through the API, none
of which has ever held a working GitHub trigger.
