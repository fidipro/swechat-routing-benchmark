# Data format

Text files are UTF-8; `.gz` files are gzip. Costs are US dollars at Amazon Bedrock list prices.

## Tables

### `sessions.csv`: one row per session of the sample (78)

| Column | Meaning |
|---|---|
| `session_id`, `repo_id` | SWE-chat session id, GitHub repository |
| `status`, `status_reason` | `usable` (shipped) or `not_usable`, and why (`selection.md`) |
| `harness_version`, `started_at`, `start_commit`, `original_user_turns` | The original session |
| `hand_picked_model`, `hand_picked_effort`, `switched_mid_session` | What the user ran (and whether they switched model mid-session) |
| `baseline_cell`, `baseline_is_proxy`, `proxy_reason` | The grid cell standing for the hand-picked choice, and why it differs when it does |
| `original_completed`, `session_class`, `multi_task`, `env_requirements` | From the task analysis |
| `n_criteria`, `n_cells_labelled`, `n_cells_pass`, `passing_cells`, `baseline_verdict` | Grid outcome (usable sessions) |
| `cheapest_passing_cell`, `outcome_category` | Oracle cell; one of five categories (`results.md`) |
| `baseline_cost_usd`, `cheapest_passing_cost_usd`, `original_cost_usd` | Label costs of the two cells; the original session's cost recomputed from its logged tokens |

### `cells.csv`: one row per replayed cell of a usable session

`session_id`, `cell`, `tier`, `model`, `effort`; `verdict` (final: `pass` / `fail`), `judge_verdict`, `verdict_source`
(`judge` / `override`), `override_rule`; `judge_confidence` (decisive ballot), `votes` (1 or 3), `flag_for_review`;
`failure_class`, `cap_hit`, `attempt`; `cost_usd` (replay, from its transcript), `cost_usd_label` (oracle cost),
`estimated_cost_usd`, `cost_cap_usd`, `judge_cost_usd`, `emulated_user_cost_usd`; `user_turns`,
`original_user_turns`, `script_entries`, `script_delivered`, `script_never_reached`; `wall_seconds`; token totals
`input_tokens`, `output_tokens`, `cache_read_input_tokens`, `cache_creation_input_tokens`.

### `criteria.csv`: one row per acceptance criterion

`session_id`, `criterion_id`, `verifiable_by` (`judge` / `diff` / `tests` / `build`), `introduced_at_turn`, `basis`
(`user_request` / `repo_rule` / `request_consequence`), `remote`, `text`.

### `criterion_verdicts.csv`: one row per (cell, criterion) the judge scored

`session_id`, `cell`, `criterion_id`, `met`, `evidence` (the judge's one-line pointer into the replay).

### `redactions.json`

Count of credential-shaped strings replaced by `<redacted:kind>` in the shipped files.

## Per session: `sessions/<session_id>/`

| File | Contents |
|---|---|
| `session.json` | Source reference (dataset, revision), repository, start state, hand-picked model and effort, original turn count and cost. Not the original transcript |
| `task.json` | Task analysis: statement, class, acceptance criteria (with `basis`, `source`, `remote`), turn script (with the user's wording), completion and evidence, environment requirements, analyst model |
| `cells/<cell>/label.json` | Judge label: verdict, decisive ballot (criteria with evidence, confidence, reason), all ballots, costs, review flag. A human override is not applied here; see `cells.csv` |
| `cells/<cell>/record.json.gz` | Replay record: every turn (message, final reply, timing, result), emulated-user decisions, caps, failure class, usage per message, cost reconciliation against the signing proxy, tool counts, test and build runs, final diff and git status |
| `cells/<cell>/transcript/<id>.jsonl.gz` | Claude Code's own session log of the replay (main session; `<id>/subagents/*.jsonl.gz` for subagents) |
| `cells/<cell>/final.diff.gz` | The replay's final diff against the start commit |
| `cells/<cell>/signer-ledger.jsonl.gz` | Every model request of the replay as metered by the signing proxy (model, usage, cost, status) |
