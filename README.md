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

## Review routines

Two routines split the work by event, so neither has to work out which case it is in:

| Routine | Event | Scope |
| --- | --- | --- |
| Senior code review | `pull_request.opened` | Whole diff, all three buckets |
| Commits pushed | `pull_request.synchronize` | `baseline..head` only, Block findings only, plus a resolution check of the previous review's items |

Each routine opens its comment with a different first line:

```
Senior code review:  Senior review (rev <short-sha>) - N Block, M Fix-in-PR, K Follow-up
Commits pushed:      Senior review (rev <short-sha>) - re-review of <base>..<head>, N Block
```

What couples them is only the shared prefix. The re-review routine looks for the most
recent comment beginning `Senior review` and reads the SHA out of it, so either format
can serve as its baseline — including its own, which is how consecutive pushes chain.

The suffixes are therefore free to change; `Senior review (rev <short-sha>)` is not. Drop
or reword that prefix and the re-review routine finds no baseline, concludes no prior
review exists, and silently declines — which looks identical to a trigger that never
fired.

Set each routine's trigger to its single action. Selecting *all actions* on either one
makes both fire on the same event and posts two reviews per push.

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
| `pull_request.opened` (PR #2, 18:31Z) | **comment posted in 55 seconds** |

The first three rows were all run against a routine whose GitHub trigger had never
saved — the UI returned *"Routine saved, but the GitHub trigger couldn't be updated."*
Those events had no listener, which is indistinguishable from a broken webhook until you
compare against a manual run.

Once the trigger saved, `pull_request.opened` delivered in under a minute. Webhook
delivery was never the problem; the trigger simply did not exist.

One hypothesis was wrong and is worth recording: routines created through the API were
suspected of being unable to hold a GitHub trigger, since every routine on this account
shares `created_via: "http_api"` and none had ever fired. The routine that finally worked
carries the same field, so the correlation was spurious.
