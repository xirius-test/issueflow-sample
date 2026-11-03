# How to run a check pass

## 1. Discover what to check

Call `operator_model_currency_check_targets`. It returns `current_catalog_version`
and, for every curated AGENTIC framework at that version, a `targets` entry:

- `framework` — the name.
- `model_source` — `{"url": ..., "medium": "api" | "page"}`, or `null` when the
  catalog declares none.
- `known_models` — the model names the catalog already associates with this
  framework.

A `503 catalog-unavailable` means this install has no seeded catalog at all —
stop and report the run as failed with that detail; it is not "nothing to
check" (an empty `targets` list, by contrast, legitimately means the catalog
currently curates no agentic framework, and a run that reports nothing is
correct in that case).

## 2. Judge each target, in order

For every entry in `targets`:

**No declared source (`model_source` is `null`).** Record one finding:
`unchecked`, reason `"no declared source"`. Do not guess a URL.

**`medium: "page"`.** Visit the URL and read it the way you would read any
other doc page. Note the models it lists as current, and whether it marks
any `known_models` entry as superseded, retired, or simply absent from the
current list (absent is not automatically `retired` — say so only when the
page itself says the model is gone; otherwise it may just be an older model
the vendor still supports but no longer advertises, which stays `current`).

**`medium: "api"`.** Call the endpoint directly. If the vendor's own docs say
it needs a credential, look for one in `secrets.env` beside your workstation
config, named for this framework (e.g. `claude-code`'s would be
`CLAUDE_CODE_MODEL_SOURCE_API_KEY`) — never a process environment variable.
Absent and required: `unchecked`, reason `"missing credential"`. Present but
rejected (401/403): `unchecked`, reason `"invalid credential"`, naming the
status the vendor returned. The source being unreachable at all (DNS,
timeout, 5xx) is also `unchecked`, reason naming what happened.

**For every model you could judge**, one finding per model:

- it is in the vendor's current list and still in `known_models` → `current`.
- it is in the vendor's current list and NOT yet in `known_models` → `new`.
- the vendor itself marks it superseded → `superseded`, `successor` set to
  whatever the vendor names (omit it when the vendor names none).
- the vendor itself marks it retired (no longer offered at all) →
  `retired`, `successor` the same way.

Every finding's `reason` quotes or paraphrases what the vendor's source
actually said — never a bare label with no evidence.

## 3. Report

Pass with a summary naming, per framework, how many findings of each kind you
produced — and put the full structured list in `evidence.model_currency_check`:

```json
{
  "catalog_version": 7,
  "findings": [
    {"framework": "claude-code", "kind": "current", "model_name": "claude-sonnet-5",
     "reason": "the Anthropic models API still lists it"},
    {"framework": "claude-code", "kind": "new", "model_name": "claude-opus-5-5",
     "reason": "the Anthropic models API lists it; the catalog does not name it yet"},
    {"framework": "claude-code", "kind": "superseded", "model_name": "claude-haiku-4-5",
     "successor": "claude-haiku-5",
     "reason": "the Anthropic models API marks it superseded by claude-haiku-5"},
    {"framework": "gemini-cli", "kind": "unchecked",
     "reason": "source unreachable: HTTP 503 from https://ai.google.dev/gemini-api/docs/models"}
  ]
}
```

Each entry's shape is `packages.core.model_currency_check.FrameworkCurrencyFinding`
(`framework`, `kind`, `model_name`, `successor`, `reason`) — the contract the
registry-write and digest steps read, not free text you invent per run.
Before you report, confirm every framework `operator_model_currency_check_targets`
returned has at least one finding in your list — a framework with none is the
one outcome this check must never produce.
