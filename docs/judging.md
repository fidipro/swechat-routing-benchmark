# Judging: what a "pass" means

## 1. Task analysis (once per session)

A task analyst (Fable 5.1, medium effort) read each original session (transcript, final diff, test and build runs,
the user's completion signals) and wrote `data/sessions/<id>/task.json`:

- **Task statement** and **session class** (`edit+tests`, `edit-notests`, `noedit`).
- **Acceptance criteria** (`data/criteria.csv`), each with
  - a **basis**: `user_request` (the user's own words, quoted in `source`), `repo_rule` (a written rule in the project
    instructions or in a skill or command the user invoked) or `request_consequence` (directly implied by the request:
    the change works, the repository's checks still pass). A detail only the original assistant chose is not a
    criterion; the judge sees the original result as a reference instead;
  - `verifiable_by`: `judge`, `diff`, `tests` or `build`;
  - `remote`: meeting it needs evidence or actions outside the repository (CI logs, trackers, cloud state, web);
  - `introduced_at_turn`: the user turn it comes from; criteria of later turns count once that turn was delivered or
    became moot.
- **Turn script** for the emulated user (`replay-method.md`); entries can **withdraw** earlier criteria.
- **Original completed** (`yes` / `partial`) with evidence, and **environment requirements** found by keyword.

The rules the criteria follow are in `prompts.md`: outcomes rather than implementation; an approval of the assistant's
own plan ("fix everything", "y") is judged against the plan the replay presented; a choice the user delegated does not
encode the original's choice; one requirement per criterion; only what a replay's record can show. The criteria were
written first and then revised under these rules; the turn script the emulated user played was not changed.

## 2. What the judge sees (per replay)

Fable 5.1, medium effort, structured output. The prompt contains:

1. The task statement and the acceptance criteria (marked `remote` where so), which are withdrawn, and which are not
   applicable (introduced by the user's answer to a question the replayed model never asked).
2. The original result **as a reference only**: its final message and final diff.
3. The replay: model and effort, user turns against the original, cap hit, which script entries were delivered,
   skipped (with the reason) or never reached, every emulated-user message, tool counts, the tests it ran with output.
4. For every turn: the user message, the model's messages **before** its final reply (where models mention side
   changes), and the final reply.
5. The replay's final diff (up to 30,000 characters).

It never sees the other cells of the session or the cost.

## 3. Rules

The full text is in `prompts.md`. In short:

- The bar is the acceptance criteria, **not "as good as the original"**; meeting every criterion by another route passes.
- Every non-withdrawn criterion is scored with evidence; **pass only if all are met**; doubt goes into confidence.
- A replay that stopped at a clarifying question where an answer was required fails, unless the emulated user answered
  and the replay then completed. A replay stopped by a cap with criteria unmet fails.
- **Scope**: a change beyond what a criterion names does not by itself fail it when it follows directly from the
  request and the replay disclosed it; unrelated, undisclosed or behaviour-changing extras still fail.
- **Remote actions** (push, open a PR, deploy) are met when every local step is done and the replay pushed, attempted
  to, or said it would. **Remote criteria** are met when the replay used the evidence if reachable, or said it could not
  and its answer or fix is consistent with what the repository shows.
- **Checks the environment cannot run** (tool not installed, failures already present before the change) are met when
  the replay ran the checks it could, said what it could not run, and its diff shows no defect the missing check would
  catch; a check it could have run and did not stays unmet.
- **Skipped script entries**: a criterion of an entry the emulated user skipped as moot is met when nothing in the
  replay called for that turn.
- **Not observable**: a criterion about something the record cannot show (commit message wording) is judged from what
  the replay says, else met with the doubt in confidence.
- **Answers** are judged as the user received them at that turn: every fact the criterion names must be in the answer.

## 4. Votes, confidence, review flag

Each verdict has a confidence in [0, 1]. A **first verdict below 0.7 gets two more**, and the majority of the three
decides (ties: the first). `label.json` keeps every ballot (`votes`). A split vote, or three verdicts all below 0.7,
sets `flag_for_review`. Per criterion, the decisive ballot's `met` and evidence are in `data/criterion_verdicts.csv`.

## 5. Human review and overrides

The sessions where every cell failed were reviewed cell by cell by a person, reading the judge's evidence across all 7
cells, the turn script and the replays' replies. That review set environment-blocked sessions aside
(`selection.md`), shaped the criteria and judge rules above, and checked the resulting labels. Three cells carry a
**human override** of the judge's verdict (`verdict_source = override` in `data/cells.csv`, with `judge_verdict` and
`override_rule`; `label.json` stays the judge's):

| Rule | Cells | Meaning |
|---|---|---|
| `overspecified` | 1, fail → pass | the failing criterion still copied an offer the original assistant made, which this replay never did |
| `user_override` | 1, fail → pass | the reviewer's own verdict against the judge |
| `judge_misread` | 1, pass → fail | the judge's own evidence contradicts its verdict under its rules |

## 6. Measured reliability

- **Noise**: 14 replays judged 10 times each: a single verdict disagreed with the 10-vote majority 2 times in 140
  (1.4 %, 95 % interval 0.4–5.1 %); both disagreeing verdicts had confidence 0.55, and a majority of 3 drawn from the
  repeats never disagreed. Two easy sessions only.
- **Against the human review**: of the 14 cells the review passed by rule, the final judge passes 12; one of the two it
  fails was a reviewer error (a diagnosis presented as fact with no disclosure that the CI logs were unreachable).
- **Not measured**: agreement with human labels on a random sample.

## 7. Who labels what

The analyst and the judge are Fable 5.1, newer than the replayed models; they label, they are never a candidate.
Judge cost is a labelling cost, not part of any routing cost.
