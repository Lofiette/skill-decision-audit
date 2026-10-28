# Check (step 4)

Read this at step 4: the light check when every Light condition there holds, a review otherwise.

## Light check

It replaces the reviewer, not the evidence: the leader still needs depth 9.

1. **Undo first.** Write the one command that restores the prior state, e.g. `git checkout <commit> -- <path>`, or copying `<backup>` over `<path>`. Not committed yet: copy the files aside now. The undo must not need the changed code to run.
2. **Three answers, each resting on a fact** (command output or `file:line`): does it break what works now (callers, code it shares, states it was not built for)? Is a mandatory consequence missing (reload or restart, migration, dependency, cleanup on unload)? Does it do what the user asked, within their constraints, and nothing more? An answer without a fact is a doubt: review instead.

The check has **passed** when the undo is written and all three answers rest on facts. Nothing is changed yet: go to step 5. Only on the Light rung, in step 6:

- **Act and observe** in the state that exercises the change, and in one it was not built for (another tab, section or theme) if you can reach it. Keys, clicks or a reload in the user's live app need their yes first, each time.
- **Report** what changed, how you observed it, the runner-up with what put it second, and the undo.
- **Failure** or a side effect: run the undo, confirm the prior state is back, report. A retry is a new decision on the review path, never settled (b) or (c).
- **Log** one `settled.md` line (`references/log.md`).

## Review

Do these in order; the spawn comes last.

1. Fill in the brief skeleton below.
2. Check it before sending: no candidate carries a rating, depth or score of yours, in any language or form ("rating 8", "8/10", "Оценка 8, глубина 9"). Rating and depth numbers appear only in the anchor tables.
3. Draft the decision file (`references/log.md`, read it now) with your scores, the evidence and Outcome: awaiting review. Your scores go there, never in the brief.
4. Spawn one reviewer with `subagent`, passing the brief. Not `subagent_fork`: a fork shares your history and model, so it is not independent.
5. When the call returns an id, write one line with no tool call, e.g. "Review requested; I continue when the verdict arrives." That ends your turn.

The verdict arrives as the reviewer's message or the runtime's completion notice and starts your next turn; nothing you call brings it sooner. Until then: no state change; no `list_agents`, `job_list`, `job_output` or sleep; no re-probing what the brief asks; no asking the user to choose (Confirm and Escalate are presented after the verdict, with both raters' scores). Anything else belongs before the spawn or after the verdict.

### Brief skeleton

The reviewer starts with an empty conversation. Copy the block, fill every `<…>`, add nothing outside it; any language.

```text
## Request
<the user's request, verbatim>
Earlier constraints from the user on this action or target: <each, quoted, or "none">

## Unknowns
- <unknown>: <Verifiable | Preference | Settled (a-d)>

## Candidates
### C1. <name>
What it does: <one or two lines>
Rests on: <command and the output lines that matter, or file:line>
Known caveats: <facts only, not how much they matter; or "none found">

### C2. <name>
<the same three lines; one block per unrefuted candidate>

## Refuted
- <candidate>: <the refuting fact and its source>, or "none"

## Planned action
Action: <exact action>
Target: <exact path, identifier, PID or host>
Observation: <how the result will be observed>
Undo: <how it is undone>

## Scales
Rate a candidate as if its unconfirmed points hold; whatever is unconfirmed lowers depth, not the rating.

| Rating | Anchor |
|---|---|
| 0 | does not work, or costs more than the problem is worth |
| 2 | works only with heavy rework or ongoing cost |
| 5 | works, with real caveats or collateral |
| 8 | works cleanly, minor known drawback |
| 10 | fully answers the question as posed, no caveats |

| Depth | Anchor |
|---|---|
| 10 | proven by reproducible facts obtained here; target verified; consequences known |
| 9 | proven; only minor consequences unconfirmed, nothing material |
| 8 or below | something material still rests on inference |

## Your task
You are an independent reviewer. Work read-only: you may read files and rerun read-only probes; change nothing. For every unrefuted candidate, set your own evidence depth (0-10) from the facts given, using the depth anchors, and your own option rating (0-10), using the rating anchors; you are not shown the author's scores. If a probe refutes a candidate, quote the fact instead of a depth. For each refuted candidate, say whether the cited fact actually rules it out; if it does not, give it a depth and a rating like the others. Discard every candidate with depth 8 or below. For the survivors, check (a) technical soundness: will it break what currently works; (b) completeness: is a mandatory consequence missing (migration, dependency, restart, rollback); (c) target: does it address what the user actually asked, within their constraints. If an obvious alternative that could rate within 2 points of the best one is missing, name it as an objection and describe it, without a rating or depth. Return: depth and rating per candidate (or the refuting fact), which ones you discarded and why, and concrete objections for the survivors, or "no objections".
```

## Margin

The leader's rating minus the best other unrefuted candidate's, at the bar or not, computed with your ratings and again with the reviewer's for the same leader; if either is under 2, it is under 2. A margin of 2+ counts only if the runner-up's rating rests on a cited fact. No unrefuted rival: met.

## Handle the answer

- **Ratings** feed the margin. A rating disagreement is not an objection and needs no fact.
- **Refutation rejected** → that candidate is unrefuted again; recompute the margin.
- **Objections**, including a refuting fact the reviewer found → back to step 2. Close each by a fact, not by argument. If the fact confirms the objection, the verdict changes.
- **Missing alternative** → research and score it yourself, then send it to the second review. A number the reviewer gave it is not a review: until reviewed, it cannot lead, be acted on, or be shown to the user with a score.
- **Everything discarded** → back to step 2 for one round of probes aimed at the reviewer's reasons, entered as each candidate's Next probe. If the next review discards everything again, take step 2's exit.
- **No usable answer** → retry once. That covers a call that failed for a reason other than the depth limit, a candidate with neither a depth nor a refuting fact, and a missing rating. If the retry also fails, do not act: go to step 5 as Escalate, marked "unreviewed".
- **Depth-limit rejection** → do not retry: you are a worker (see `references/delegation.md`).
- **No objections** → step 5.

Review again whenever new facts or a changed plan change the leader, the action or the target, or add a candidate. A leader change caused by the review's own discards needs no new review. A decision gets at most two reviews; if a third would be needed, go to step 5 as Escalate.

**Done when** the reviewer has rated every unrefuted candidate, the leader passed, and every objection is closed by a fact.
