# Session selection

## 1. Eligible pool: 182 sessions

Six filters over every Claude Code session of SWE-chat v2 (revision `df393df`), in order:

1. Claude Code session of the normal kind (not imported, review or investigate), with a full transcript: 11,484
2. Transcript available and parseable: 11,330
3. At least 3 user turns: 7,315
4. Model and first user message present, model priced, effort logged (Claude Code 2.1.212 or later): 762
5. Start commit recoverable (the transcript's git block, else the single parent of the earliest linked commit when it
   predates the session): 423
6. Clean start (no uncommitted files at session start): **182 sessions from 49 repositories**

## 2. The 78-session sample

A repository-capped pool of 100 sessions (at most 5 per repository, seed 7), then the 80 with the lowest estimated
replay cost; 2 of the 80 were left out after a replay of each read files outside its workspace in a first run. The
cost cut favours shorter sessions (`limitations.md`).

## 3. Shipped: 43 of 78

`data/sessions.csv` lists all 78 with their `status` and `status_reason`; replays and labels are shipped for the 43
usable ones.

| Reason not shipped | Sessions | Detail |
|---|---|---|
| No image | 9 | The repository could not be built at its start commit in the date-limited environment: Gradle wrapper downloads (5), a cargo git dependency (2), PyTorch's own package index (2) |
| Environment | 24 | The task needs something a replay cannot have: hosted pull requests, CI logs or review remarks (5); secrets, a notes vault or a sibling repository (3); an external tracker or live service (2); Docker, a desktop GUI or a cluster (2); a data remote (3); the branch to work on missing from the image, with the work in a sibling repository (1); a hardware toolchain host or a file on the user's machine (2); the user's planning plugin (2); a toolchain or disk the image lacks: Elixir (1), `protoc` and a full Rust workspace build within 30 GB (3) |
| Start state | 1 | A multi-repository task folder; the change lives in a repository other than the recorded start state |
| Incomplete grid | 1 | One cell's last allowed replay was tainted by registry upstream errors |

The environment cases were found by a manual review of the sessions whose baseline failed or where every cell failed;
environment-limited sessions with a passing cell were not reviewed and are shipped.

