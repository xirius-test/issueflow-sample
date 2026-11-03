# CI verification

This is the **ci-verification** stage's agent — `kind: process`, `framework:
scm-merge`. It is not an LLM turn: the engine dispatches this stage to a
deterministic tool, launched via the workstation's declared `scm-merge`
command for whichever source-control provider this repository uses. No
agent reads this charter's prose; it documents the stage for humans reading
the config.

## What it does

Waits, on its own clock, for the pull request's **required checks** — the
contexts the target branch's protection requires — on the pull request's
current head. A check that is not required is information, not a gate, and
only the newest attempt of a check counts. Because the pull-request stage
already started CI before the review chain ran, this stage usually finds
the checks already done and returns at once.

## Outcome & routing

- `passed` → `merge`, immediately, with no further wait.
- `failed`:
  - a required check failed on the code (tests, lint or types) → the stage
    that fixes it, with the run marked so the fix is re-checked by review
    and CI alone rather than the full chain;
  - the runner itself failed, or a required check never reported → escalate,
    since no commit changes either;
  - a conflict with the target branch → the stage that resolves it;
  - a transient GitHub error, or a check still running at the wait's bound →
    retried here, a bounded number of times, then escalate.
- `not-needed` → **never reported by this stage.** The tool emits only
  `passed` and `failed`.

A red check is final the moment it is seen. A job that died on the runner is
re-run once; if that run goes green the stage passes and says which jobs it
re-ran. The evidence of a failure names the failing checks, the failing
tests and the tail of each failed log.
