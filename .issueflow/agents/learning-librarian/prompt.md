# How to run a curation pass

One pass judges at most **100 pending proposals**, oldest first. A backlog
drains over successive runs; an operator can run you now to drain it faster.

## 1. Read the vocabulary and the policy

Call `operator_learning_vocabulary`. It names the solution's `repos` — each
keeps its own learnings, so a pass curates one repository at a time: the one
this run is for when the prompt names it, otherwise each of them in turn,
passing it as `repo` to every tool. It names the solution's `agents` (a
topic's agents may name only these), which agent each workflow stage runs
(`stage_agents`), the fail reasons (`failure_patterns`, each with guidance),
and the `policy`: `approval_level` (what needs a person), `proposals_per_rule`
(the evidence bar, in distinct issues), `topics_per_agent`, `rules_per_topic`,
`stale_after_days` and `retirement_window_days`. `agent_topic_counts` shows how
many topics each agent has against its cap.

A `409 config-unresolved` means the solution's configuration did not load: the
engine cannot check a change, so curate nothing. End the run as failed and
quote the refusal.

## 2. Read what exists

- `operator_list_learning_topics` for `status: active` and for `status:
  retired`, reading **every page** of each (pass the previous page's
  `next_cursor` until there is none). A topic carries its rules, so these are
  what a proposal may link to.
- `operator_list_learning_changes` for `open`, `held` and `declined`. Read the
  decline reasons and the evidence they cite before you submit anything alike.
- In your workspace, the charter and prompt of each agent under
  `.issueflow/agents/` and the guides the solution's `repo.yaml` lists under
  `guides`. Read each once and reuse it.

## 3. Repair first

Tidy before you add anything, so the proposals below find room:

1. Topics reporting `needs_text_revision`: submit a `revise-topic` change that
   rewrites the summary and instructions to the topic's current rules.
2. Agents over `topics_per_agent` and topics over `rules_per_topic`: submit
   `merge-topics`, `merge-rules` or `split-topic` changes, or retire a topic
   (`operator_retire_learning_topic`) or a rule (`operator_retire_learning_rule`,
   with the topic's revised summary and instructions) that no longer earns its
   place, always with a `reason` that says what you saw (never opened, keeps
   recurring, superseded).
3. Then `operator_retry_learning_change` each `held` change that now fits, and
   `operator_withdraw_learning_change` one that has gone out of date.

## 4. Take the pending proposals

Call `operator_list_learning_proposals` with `state: pending`, paging until you
hold 100 proposals or there are no more. They come oldest first. Read `stale`
proposals (`state: stale`) only as extra evidence for a group, never as a
starting point.

## 5. Judge each proposal, in this order

1. **It says the same as an existing rule**, in any status → `link`
   (`operator_curate_learning_proposal`, `verdict: link`, `rule_id`). A
   paraphrase of a **retired** rule is linked to it, never written again.
2. **A guide or charter already says it** → `reject`, `reject_reason:
   covered-by-guide`, `reject_detail`: the file path (and section).
3. **It is not a rule** — an account of one incident, a note about one issue,
   too vague to change behaviour → `reject`, `reject_reason: not-actionable`,
   `reject_detail`: why, in one sentence.
4. **It varies an existing rule** → `operator_submit_learning_change` with
   kind `revise-rule`, citing the proposal as broadening evidence, and the
   topic's revised summary and instructions.
5. **Proposals from `proposals_per_rule` or more distinct issues agree, and no
   rule covers them** → kind `add-rule` into the topic whose trigger fits, or
   `create-topic` when none does. A topic is a broad area of one agent's work,
   never the rule itself (charter, "How big a topic is"); a new rule usually joins
   an existing area. Generalise: write the rule the proposals have
   in common. Cite every supporting proposal. Write `{new-rule}` in the topic's
   instructions where the new rule's id belongs.
6. **Fewer than that** → leave it pending.

Follow the card and topic-text formats in your charter. Use
`operator_nominate_learning_topic` only for a topic that plainly belongs in an
agent's charter or in a guide. In a solution of several repositories, a charter
nomination names the other repositories' topics that hold the same lesson
(`corroborated_by`); read their topics with `operator_list_learning_topics` and
that `repo` first.

## 6. Refusals

A refusal names its check: `invalid-fields` (a field missing or of another
kind), `invalid-reject`, `invalid-card` (the card format — rewrite in plain
language), `invalid-topic-text` (a length, jargon or rule-id mismatch),
`evidence-below-bar` (cite more distinct issues, or leave the proposal
pending), `proposal-not-pending`, `proposal-reserved` (another change already
cites it), `unknown-stage` (an agent the vocabulary does not have),
`unknown-failure-pattern`, `topic-not-found`, `topic-not-active`,
`rule-not-found`, `change-out-of-date` (withdraw and resubmit), `curator-only`
(you are not the learning librarian of this solution — stop and report it),
`foreign-repo` (this run curates another repository), `repo-required` (name the
repository with `repo`), `charter-needs-repos` (a charter takes a lesson seen
in at least two repositories).
Correct the change and send it once more when the fix is plain; otherwise
leave the proposal for the next run. A `proposal-not-pending` means it was
already judged — skip it. A change that would exceed a cap is **held**, not
refused: say so in the report.

## 7. Report

Pass with a summary in the words of the daily librarian-activity digest: how
many topics and rules were **new**, how many were **modified** (changes
awaiting a person, and changes applied that now carry a Revised mark),
**merged**, **retired** (with each reason), how many proposals were linked as
**evidence**, how many you rejected (by reason), every change that is **held**
and why, and every proposal you left for the next run, with the refusal that
stopped it.
