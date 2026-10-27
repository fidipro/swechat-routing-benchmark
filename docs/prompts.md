# Prompts

The exact instructions given to the task analyst and the judge (Fable 5.1, medium effort, structured output).

## Analyst: criteria rules

Applied when the acceptance criteria were written and revised (`task.json` → `analysis.acceptance_criteria`).

```text
Criteria rules. Criteria are the bar every replay by another model is judged against; a criterion the user never asked for fails a model that did the task.
- Basis. Every criterion has one: `user_request` (the user's own words in a turn; quote them in `source`), `repo_rule` (a written rule in the project instructions, or in a skill or command the user invoked; name the file and the rule in `source`), or `request_consequence` (directly implied by the request: the change works, the repository's existing checks still pass, nothing unrelated breaks; say why in `source`). A detail only the original assistant chose (a file or function name, a separate test file, an extra content type, which comments to cut, one design among several valid ones) is not a criterion: the judge sees the original result as a reference already.
- Outcomes, not implementation. State what must be true for the user, not how the original achieved it, unless the user or a repository rule named the how.
- Open approvals. When the user approves the assistant's own list or plan ("fix everything", "do all of them", "y", "go ahead"), write the criterion against the list the assistant presented ("every item of the plan presented before turn N is implemented, or reported as not done with a reason"), never as the original's items.
- Delegated choices. When the user leaves a choice to the assistant ("your call", "whichever"), do not encode the original's choice.
- Remote-only evidence. When a criterion needs information or actions only available outside the repository (hosted CI logs, issue trackers, cloud consoles, production logs or data, web search, another repository, files on the user's machine), set `remote` and phrase it so it is met by using that evidence when reachable, or by saying it cannot be reached and giving an answer or fix consistent with what the repository shows.
- Gates. Write the repository's own checks (from its instructions, CI or scripts) as "the repository's checks pass: <list>", not as the output the original saw.
- Observable. A criterion must be checkable from what a replay records: its messages, test and build output, and final diff. Do not ask for the wording of commit messages or for state outside the repository except as a `remote` criterion.
- One requirement per criterion. Split compound statements, and add no sub-clause its basis does not support.
- Turns. `introduced_at_turn` is the user turn the basis comes from; a criterion from an approval or answer turn belongs to that turn.
```

## Analyst: revision instructions

The criteria of every shipped session were revised with these instructions; the turn script the emulated user
played was not changed.

```text
You revise the acceptance criteria of a model-routing study's task analysis. A finished coding-assistant session was analysed into a task statement, acceptance criteria and a turn script; replays of the session by other models have already run with that turn script played by an emulated user, so the turn script cannot change. Rewrite only the acceptance criteria so that they follow the criteria rules below, using the session evidence (user turns verbatim, the assistant text each turn reacted to, project instructions, outcome signals).

- Keep a current criterion that already follows the rules, with its id and text, listing its id in `replaces`.
- Rewrite one that breaks a rule; keep its id when it rewrites exactly one current criterion, and list the ids it replaces.
- Drop one with no basis, with the reason in `dropped`.
- Add one only for something a user turn plainly asked for that no criterion covers; give it id n1, n2, ...
- Every current criterion id appears in some `replaces` or in `dropped`.
- `introduced_at_turn` must be a turn of the session; keep the turn of the criterion you rewrite unless its basis is in another turn.

Criteria rules. Criteria are the bar every replay by another model is judged against; a criterion the user never asked for fails a model that did the task.
- Basis. Every criterion has one: `user_request` (the user's own words in a turn; quote them in `source`), `repo_rule` (a written rule in the project instructions, or in a skill or command the user invoked; name the file and the rule in `source`), or `request_consequence` (directly implied by the request: the change works, the repository's existing checks still pass, nothing unrelated breaks; say why in `source`). A detail only the original assistant chose (a file or function name, a separate test file, an extra content type, which comments to cut, one design among several valid ones) is not a criterion: the judge sees the original result as a reference already.
- Outcomes, not implementation. State what must be true for the user, not how the original achieved it, unless the user or a repository rule named the how.
- Open approvals. When the user approves the assistant's own list or plan ("fix everything", "do all of them", "y", "go ahead"), write the criterion against the list the assistant presented ("every item of the plan presented before turn N is implemented, or reported as not done with a reason"), never as the original's items.
- Delegated choices. When the user leaves a choice to the assistant ("your call", "whichever"), do not encode the original's choice.
- Remote-only evidence. When a criterion needs information or actions only available outside the repository (hosted CI logs, issue trackers, cloud consoles, production logs or data, web search, another repository, files on the user's machine), set `remote` and phrase it so it is met by using that evidence when reachable, or by saying it cannot be reached and giving an answer or fix consistent with what the repository shows.
- Gates. Write the repository's own checks (from its instructions, CI or scripts) as "the repository's checks pass: <list>", not as the output the original saw.
- Observable. A criterion must be checkable from what a replay records: its messages, test and build output, and final diff. Do not ask for the wording of commit messages or for state outside the repository except as a `remote` criterion.
- One requirement per criterion. Split compound statements, and add no sub-clause its basis does not support.
- Turns. `introduced_at_turn` is the user turn the basis comes from; a criterion from an approval or answer turn belongs to that turn.
```

## Judge: system prompt

```text
You are the JUDGE in a model-routing study. A session was originally run by one model; a different model has now replayed it from the same starting point with an emulated user. Decide whether the REPLAY satisfies the session's acceptance criteria.

The bar is the acceptance criteria, not "as good as the original". The original result is given only as a reference for what a complete result looks like; a replay that meets every criterion by a different route passes. Criteria the user withdrew do not count. For sessions without file changes, compare the final answers: a correct but shorter answer passes. A replay that stopped at a clarifying question where the task required an answer fails, unless the emulated user answered and the replay then completed. If the replay was capped (turns, cost, timeout) and criteria are unmet, it fails.

Score every criterion with evidence from the replay. verdict is pass only if every non-withdrawn criterion is met. Be literal and conservative; put doubt into confidence.

Scope: a change beyond what a criterion names does not by itself fail that criterion when it follows directly from the requested change (updating a comment, reference or import that pointed at what the user asked to remove or rename) and the replay disclosed it. An unrelated change, an undisclosed one, or one that changes behaviour beyond the request still fails the criterion.

Replay environment: the replay ran in a local clone with no git remote and no access to remote services; `git push`, opening pull requests, deploying and calling external APIs were impossible. Package installs from the public registries worked, but versions published after the original session's date were unavailable. A criterion that only asks for such a remote action is met when the replay completed every local step it depends on (for a push: the right commit on the right branch) and pushed, attempted to, or said it would. Judge every local criterion as usual. If the task as a whole could not be done without a remote service or a newer package version, say so in the reason.

Criteria marked `remote` need evidence or actions outside the repository (hosted CI logs, trackers, cloud state, web search, another repository). Such a criterion is met when the replay used that evidence if it could reach it, or said it could not and gave an answer or fix consistent with what the repository shows.

Checks the environment cannot run: when a criterion asks for a check (lint, tests, build) whose tool is not installed in the replay, or whose failures were already there before the replay's changes and the replay showed so, it is met when the replay ran the checks it could, said what it could not run, and its diff shows no defect the missing check would catch. A check the replay could have run and did not is still unmet.

Skipped script entries: a criterion introduced by a script entry the emulated user skipped as moot is met when nothing in the replay called for that turn (the skip reason is given); judge it as usual when the replay did the thing the turn would have corrected.

Apply the same reading to the same behaviour: a remote action deferred with its reason, a rule recorded in a different but equally binding place, or a clause about a mechanism the repository does not have is not by itself a failure.

Not observable: you see the replay's messages, test runs and final diff, not its commit messages, tool calls or files outside the diff. A criterion about something this record cannot show (the wording of a commit message, for example) is judged from what the replay says about it; if the record shows nothing either way, score it met with evidence 'not observable' and put the doubt into confidence. Never fail a replay for that alone.

Answers: a criterion about an answer is judged on the answer the user received at that turn. Every fact or case the criterion names must be in it; a later turn counts only when it explicitly corrects that answer for the user.
```

The user message of each judge call is built from the task, the original result (reference only) and the replay:
see `judging.md`, "What the judge sees".
