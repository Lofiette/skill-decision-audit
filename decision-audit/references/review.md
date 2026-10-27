# Review (step 4)

Read this when step 4 applies: before any state change chosen through the procedure, unless it is a preference-only choice between reversible, local candidates.

## Spawn and wait

Spawn one reviewer with `subagent`. Do not use `subagent_fork`: a fork shares your history and model, so it is not independent. `subagent` runs in the background: the call returns an id at once, and the verdict arrives later as the reviewer's message or the runtime's completion notice.

Make the spawn the last call of your message and end your turn. Until the verdict is in:

- take no state-changing action;
- do not poll (`list_agents`, `job_list`, `job_output` with a wait) and do not sleep;
- do not re-probe the questions you put in the brief;
- do not ask the user to choose; a Confirm or Escalate choice is presented only after the verdict, with both raters' scores.

The only work allowed while waiting: draft the decision file per `references/log.md` (read it now), with your own scores and evidence (Outcome: awaiting review).

## The brief

The reviewer starts with an empty conversation. The brief contains:

1. the user's request verbatim, plus every constraint the user stated earlier that bears on the action or the target;
2. each unknown and its kind;
3. every candidate and the facts it rests on: the command and the relevant output lines, or `file:line`;
4. refuted candidates with the refuting fact;
5. the exact planned action and its target;
6. both anchor tables below, with the rule: rate a candidate as if its unconfirmed points hold; whatever is unconfirmed lowers depth, not the rating.

Do not send your ratings or depth estimates.

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

Then append:

> You are an independent reviewer. Work read-only: you may read files and rerun read-only probes; change nothing. For every unrefuted candidate, set your own **evidence depth** (0–10) from the facts given, using the depth anchors, and your own **option rating** (0–10), using the rating anchors; you are not shown the author's scores. If a probe refutes a candidate, quote the fact instead of a depth. For each refuted candidate, say whether the cited fact actually rules it out; if it does not, give it a depth and a rating like the others. Discard every candidate with depth 8 or below. For the survivors, check (a) technical soundness — will it break what currently works; (b) completeness — is a mandatory consequence missing (migration, dependency, restart, rollback); (c) target — does it address what the user actually asked, within their constraints. If an obvious alternative that could rate within 2 points of the best one is missing, name it as an objection and describe it, without a rating or depth. Return: depth and rating per candidate (or the refuting fact), which ones you discarded and why, and concrete objections for the survivors, or "no objections".

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
