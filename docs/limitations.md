# Limitations

1. **One replay per cell.** A cell's verdict mixes capability with run-to-run variance; the baseline cell fails in
   some sessions whose original completed (`results.md`, "Replay faithfulness"). A majority of 3 replays per cell would
   reduce this.
2. **Downgraded baselines.** No Fable cells and no xhigh / max effort: in the sessions marked `baseline_is_proxy` the
   baseline is below what the user picked, so savings are against the grid's best match, not the user's actual choice.
3. **Selection bias.** The sample keeps the cheapest 80 of a 100-session pool (shorter sessions), and the sessions set
   aside as environment-blocked were found by reviewing failing baselines and all-fail sessions only;
   environment-limited sessions with a passing cell are shipped. The shipped set is easier than SWE-chat's eligible pool.
4. **Replay environment.** No remote, network or user configuration (`replay-method.md`); a scripted user instead of
   the real one. Criteria needing remote actions or evidence are judged on the local work. Known gaps of the images:
   branches other than the default are absent, tools pinned through `mise` (for example golangci-lint) are not
   installed, browser downloads for Playwright are refused, and some repositories' tests fail at the start commit
   inside the image; the judge's rule for checks the environment cannot run covers these, but a replay could not see
   their output.
5. **Judge.** One model family writes the criteria and labels everything (Fable 5.1). Noise was measured on 2 easy
   sessions only; agreement with human labels on a random sample was not measured. The criteria and judge rules were
   shaped by a human review of the sessions where every cell failed, and checked against it; that review saw no
   session with a passing cell. Labels flagged for review (`flag_for_review`) were not all reviewed.
6. **Label cost** prices the first request of each model with cache reads as writes; real costs with a warm cache
   would be lower for every cell alike. Original costs are recomputed from the logged tokens at list price.
7. **Small.** 43 sessions from 26 repositories; the oracle ceiling rests on the sessions whose baseline passed.
8. **Redaction** replaced credential-shaped strings in transcripts and records (`data/redactions.json`, all
   documentation examples or test fixtures); the content they stood for is lost from those files.
