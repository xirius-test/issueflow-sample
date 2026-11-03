# Merge

This is the **merge** stage's agent — `kind: process`, `framework:
scm-merge`. It is not an LLM turn: the engine dispatches this stage to a
deterministic tool, launched via the workstation's declared `scm-merge`
command for whichever source-control provider this repository uses. No
agent reads this charter's prose; it documents the stage for humans reading
the config.

The tool merges this run's implementation change into the repository's
default branch, after the pull request's required checks are green.

## Outcome & routing

- `passed` → `done`. The change merged cleanly (or was already merged,
  idempotent no-op).
- `failed` → the stage that resolves the problem (a merge conflict, a
  failing required check, an abandoned/closed pull request, or nothing to
  merge) before the next attempt — usually `implementation-rework`, never a
  fresh `implementation` attempt once review has already run.

- `not-needed` → **never reported by this stage.** The tool emits only
  `passed` and `failed`; it renders no judgement on whether the issue should
  have existed, and no agent reads this charter. The `not-needed: escalate`
  edge this stage declares is a fail-safe, not a path: implementation and
  review have run by now, so a "the whole issue should not be implemented"
  verdict arriving HERE would mean the change is **missing**, not unnecessary
  — a fault to look at rather than a conclusion to act on.

A transport/tool failure (a timeout, network, or auth error) is never a work
outcome — it is retried in place by the recovery ladder, not routed here.
