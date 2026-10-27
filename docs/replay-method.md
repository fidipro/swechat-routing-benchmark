# Replay method

Each session is replayed once per grid cell, from its recorded start state, by a fresh Claude Code in a locked-down
container.

## Grid: the session's own generation

| Original model | Haiku tier | Sonnet tier | Opus tier | Emulated user |
|---|---|---|---|---|
| Opus 5, Sonnet 5, Fable 5 | Haiku 4.5 | Sonnet 5 | Opus 5 | Sonnet 5 |
| Opus 4.8 | Haiku 4.5 | Sonnet 4.6 | Opus 4.8 | Sonnet 4.6 |
| Opus 4.7 | Haiku 4.5 | Sonnet 4.6 | Opus 4.7 | Sonnet 4.6 |

Cells: Haiku with no effort setting (Haiku 4.5 has none); Sonnet and Opus at low, medium and high: 7 per session. No
model newer than the original is used, so no candidate can have seen the session or its outcome. There are no Fable
cells and no xhigh / max effort, so some baselines are below the hand-picked model or effort (`baseline_is_proxy`).
Inside each replay Claude Code's background model is the generation's Haiku.

## Environment

- **Image per (repository, start commit)**: Ubuntu 24.04 (pinned digest), Claude Code 2.1.304, language toolchains;
  the repository at its start commit plus the default branch as of the session start, with no remotes or tags;
  dependencies installed through the date-limited registry proxy; a build check (clean tree, no future commits, the
  repository's own tests run).
- **Container per replay**: internal network with no route out except two host services; all capabilities dropped,
  no new privileges, 4 CPUs, 8 GB memory, 2,048 processes, 30 GB disk quota; no credentials inside.
- **Model access**: a signing proxy holds the cloud credentials, allows only the session's generation models, meters
  every request (`signer-ledger.jsonl.gz`) and enforces a per-replay cost cap.
- **Packages**: PyPI, npm, Go, crates, Maven and RubyGems through a proxy that hides every version published after the
  session started; every other host (GitHub, model vendors' APIs, cloud services) is refused.
- **Harness**: `claude -p --permission-mode bypassPermissions --model <cell> --effort <effort>`, one call per user turn,
  resumed by session id; telemetry off, no MCP servers, no user-level CLAUDE.md; scheduling tools (cron, wake-ups)
  disallowed, since a one-shot turn cannot wait for them. User plugins are not installed, except superpowers 6.2.0 (the
  last version before the session) for `bb805800`, whose user ran with it.

## Turns: the emulated user

The first user message is sent verbatim. Later turns come from an emulated user (the generation's Sonnet, medium
effort) that follows the session's **turn script** (`task.json`), written by the task analyst from the original
session: each entry has an intent, a trigger (when to send it), a skip condition, the criteria it introduces or
withdraws, and the user's own wording. After each replay turn the emulated user delivers the next entry whose trigger
holds, answers a question only from the user's stated intent (otherwise it says the answer is unknown), acknowledges,
corrects, or ends the session when the script is done or moot. It never sees the original transcript beyond the script.

## Caps and failure classes

| Cap | Value |
|---|---|
| User turns | max(3, 2 × the original's) |
| Turn time | 2,400 s per turn |
| Replay cost | 2 × the estimate from the original's tokens (floor $2); 4 × (floor $4) for cells re-run after hitting it |

Every replay gets a class: `model` (ran normally, judged), `policy` (stopped by a cap, judged: a capped replay fails
its unmet criteria), `infra` (the environment misbehaved; not judged, re-run up to 3 times). Each shipped cell is its
latest attempt.

## How a replay differs from the original

- Claude Code 2.1.304 on Linux and Amazon Bedrock, against the original's own version (2.1.2xx), platform and API.
- No git remote, no GitHub, no external services; packages only as of the session date.
- No user configuration: no user CLAUDE.md, MCP servers or plugins (one exception above).
- A scripted emulated user instead of the real user, who reacted to what the original model actually did.
- One replay per cell: run-to-run variance is not averaged out (`limitations.md`).
