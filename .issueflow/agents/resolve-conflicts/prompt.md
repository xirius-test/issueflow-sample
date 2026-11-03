## 1. Bring the branch up to date

`base_branch` is the target branch named in your context's `## Branches`
section — never assume a specific branch name.

```
git fetch origin
git merge origin/<base_branch>
```

If the merge is clean (someone already resolved it, or the base moved
again and the conflict is gone), there is nothing to resolve: run step 3,
then pass.

## 2. Resolve each conflict

For every conflicted file, read **both** sides and the commits that
produced them (`git log origin/<base_branch> -- <file>`, and this branch's
own history). Keep both intents. Typical shapes:

- Both sides added entries to the same list, registry, route table or
  import block → keep both, in the file's own order.
- One side moved or renamed code the other side edited → apply the edit at
  the new location.
- Both sides changed the same logic differently → this is **not
  mechanical**. Fail (see step 4).

Never resolve by taking one side wholesale unless the other side's change
is provably already contained in it.

## 3. Verify what you touched

Run, in the foreground, never backgrounded, this repository's own lint,
type-check and test commands for every file you touched. `git status
--porcelain` must show nothing you did not intend. Commit the merge and
push to this run's branch. Never open, merge or close a pull request — the
stages around you own that.

## 4. Report

**A pass** goes back to the pull-request stage, which pushes your
resolution and opens or keeps the pull request. Report the target commit
you merged, each conflicted file and how you resolved it in one line, and
a `commands:` list with every command you ran and its result counts.

**A failure** goes back to implementation: a conflict that is not
mechanical. Name the files and the contradiction between the two sides,
and say what decision implementation has to make. Use `fail_reason:
work-incomplete`.

**Not needed**: the target branch already contains this change (cite the
commit or PR). The run is disposed of.
