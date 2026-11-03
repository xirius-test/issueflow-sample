# Pull request

This is the **pull-request** stage's agent — `kind: process`, `framework:
scm-merge`. It is not an LLM turn: the engine dispatches this stage to a
deterministic tool, launched via the workstation's declared `scm-merge`
command for whichever source-control provider this repository uses. No
agent reads this charter's prose; it documents the stage for humans reading
the config.

## What it does

1. Fetches the workflow's target branch and merges it into this run's branch
   as a real merge commit, pushing the result when the remote lags (never
   forced).
2. Opens the run's pull request into the target branch, or reuses the one
   already open.

The target's own CI starts on the pushed head right away, so by the time the
review chain finishes, the stage that waits for checks usually finds them
already green.

## Outcome & routing

- `passed` → the review chain this workflow declares next.
- `failed`:
  - a merge conflict → the stage that resolves it, which always returns
    here;
  - a transient GitHub error → retried here, a bounded number of times;
  - nothing to publish, or the pull request fell out of the mergeable path →
    back to implementation.
- `not-needed` → **never reported by this stage.** The tool emits only
  `passed` and `failed`; it renders no judgement on whether the issue should
  have existed, and no agent reads this charter.

A transport/tool failure (a timeout, network, or auth error) is never a work
outcome — it is retried in place by the recovery ladder, not routed here.
